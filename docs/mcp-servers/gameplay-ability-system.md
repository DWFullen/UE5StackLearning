# MCP Server: Gameplay Ability System (GAS)

## Overview

There is no standalone MCP server specifically for the Gameplay Ability System. Instead, GAS is controlled through the **UE5 Remote Control API** and **UE5 Python scripting** infrastructure described in [docs/mcp-servers/unreal-engine.md](unreal-engine.md). This guide covers the GAS-specific tools and patterns to add to your UE5 MCP server to enable AI agents to inspect and drive GAS at runtime.

## Architecture

```
AI Agent (Claude / Copilot / Cursor)
    │
    ▼ MCP stdio transport
Python MCP Server (ue5_gas_mcp_server.py)
    │
    ├─► UE5 Remote Control HTTP API (localhost:30010)
    │       └── Blueprint function calls on ASC, abilities, effects
    └─► UE5 Python Scripting (via ExecutePythonCommand)
            └── Editor-time GAS asset inspection and batch operations
```

## Required UE5 Plugins

Enable in **Edit > Plugins**:

| Plugin | Purpose |
|--------|---------|
| `Gameplay Abilities` | Core GAS framework |
| `Python Editor Script Plugin` | Editor-time Python automation |
| `Remote Control API` | HTTP REST + WebSocket control surface |
| `Remote Control Web Interface` | Optional browser-based API explorer |

## GAS-Specific MCP Server

Add these tools to your UE5 MCP server (or run as a standalone server) to expose GAS functionality to AI agents.

### Full Server Example

```python
# ue5_gas_mcp_server.py
import asyncio
import re
import httpx
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

UE5_REST_URL = "http://localhost:30010"

# Allowlist patterns for inputs that are interpolated into executed Python code.
# UE5 object paths: /Game/Maps/Level.Level:PersistentLevel.BP_Actor_0
_ACTOR_PATH_RE = re.compile(r'^[/A-Za-z0-9_.:\-]+$')
# Gameplay tags: dot-separated alphanumeric segments, e.g. Ability.Fireball
_TAG_RE = re.compile(r'^[A-Za-z][A-Za-z0-9_.]*$')
# UE5 content paths: /Game/Folder/Asset.Asset_C
_ASSET_PATH_RE = re.compile(r'^[/A-Za-z0-9_.:\-]+$')
# Attribute names: simple identifier, e.g. Health, MaxMana
_ATTR_NAME_RE = re.compile(r'^[A-Za-z][A-Za-z0-9_]*$')


def _validate_actor_path(value: str) -> str:
    """Validate and return an actor/object path safe for code interpolation."""
    if not isinstance(value, str) or not _ACTOR_PATH_RE.match(value):
        raise ValueError(f"Invalid actor path: {value!r}")
    return value


def _validate_tag(value: str) -> str:
    """Validate and return a gameplay tag safe for code interpolation."""
    if not isinstance(value, str) or not _TAG_RE.match(value):
        raise ValueError(f"Invalid gameplay tag: {value!r}")
    return value


def _validate_asset_path(value: str) -> str:
    """Validate and return an asset content path safe for code interpolation."""
    if not isinstance(value, str) or not _ASSET_PATH_RE.match(value):
        raise ValueError(f"Invalid asset path: {value!r}")
    return value


def _validate_attr_name(value: str) -> str:
    """Validate and return an attribute name safe for code interpolation."""
    if not isinstance(value, str) or not _ATTR_NAME_RE.match(value):
        raise ValueError(f"Invalid attribute name: {value!r}")
    return value


def _validate_number(value, default=0) -> float:
    """Validate and return a numeric value safe for code interpolation."""
    try:
        return float(value if value is not None else default)
    except (TypeError, ValueError):
        raise ValueError(f"Expected a number, got: {value!r}")


app = Server("ue5-gas-server")


async def ue5_call(client: httpx.AsyncClient, object_path: str,
                   function_name: str, parameters: dict = None) -> str:
    """Helper: call a function on a UObject via Remote Control API."""
    response = await client.put(
        f"{UE5_REST_URL}/remote/object/call",
        json={
            "objectPath": object_path,
            "functionName": function_name,
            "parameters": parameters or {},
        },
        timeout=10.0,
    )
    response.raise_for_status()
    return response.text


async def ue5_python(client: httpx.AsyncClient, code: str) -> str:
    """Helper: execute Python in the Unreal Editor."""
    response = await client.put(
        f"{UE5_REST_URL}/remote/object/call",
        json={
            "objectPath": "/Script/PythonScriptPlugin.Default__PythonScriptLibrary",
            "functionName": "ExecutePythonCommand",
            "parameters": {"PythonCommand": code},
        },
        timeout=30.0,
    )
    response.raise_for_status()
    return response.text


@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="gas_get_attributes",
            description=(
                "Get current attribute values (Health, Mana, etc.) from an "
                "actor's Ability System Component."
            ),
            inputSchema={
                "type": "object",
                "properties": {
                    "actorPath": {
                        "type": "string",
                        "description": "Full object path of the actor, e.g. "
                                       "'/Game/Maps/TestLevel.TestLevel:"
                                       "PersistentLevel.BP_Hero_0'",
                    }
                },
                "required": ["actorPath"],
            },
        ),
        Tool(
            name="gas_get_active_effects",
            description="List all active Gameplay Effects on an actor's ASC.",
            inputSchema={
                "type": "object",
                "properties": {
                    "actorPath": {"type": "string"}
                },
                "required": ["actorPath"],
            },
        ),
        Tool(
            name="gas_get_active_tags",
            description="Get all Gameplay Tags currently applied to an actor's ASC.",
            inputSchema={
                "type": "object",
                "properties": {
                    "actorPath": {"type": "string"}
                },
                "required": ["actorPath"],
            },
        ),
        Tool(
            name="gas_activate_ability",
            description="Activate a Gameplay Ability by tag on the target actor.",
            inputSchema={
                "type": "object",
                "properties": {
                    "actorPath": {"type": "string"},
                    "abilityTag": {
                        "type": "string",
                        "description": "Gameplay Tag that identifies the ability, "
                                       "e.g. 'Ability.Fireball'",
                    },
                },
                "required": ["actorPath", "abilityTag"],
            },
        ),
        Tool(
            name="gas_apply_effect",
            description=(
                "Apply a Gameplay Effect Blueprint class to an actor. "
                "Useful for testing damage, healing, or status effects."
            ),
            inputSchema={
                "type": "object",
                "properties": {
                    "actorPath": {"type": "string"},
                    "effectClassPath": {
                        "type": "string",
                        "description": "Content path to the GE Blueprint, "
                                       "e.g. '/Game/Abilities/GE_FireDamage.GE_FireDamage_C'",
                    },
                    "level": {
                        "type": "number",
                        "description": "Effect level (default 1)",
                        "default": 1,
                    },
                },
                "required": ["actorPath", "effectClassPath"],
            },
        ),
        Tool(
            name="gas_set_attribute",
            description=(
                "Directly set an attribute value on an actor for testing purposes. "
                "Uses an Instant Gameplay Effect under the hood."
            ),
            inputSchema={
                "type": "object",
                "properties": {
                    "actorPath": {"type": "string"},
                    "attributeName": {
                        "type": "string",
                        "description": "Attribute name as in the AttributeSet, "
                                       "e.g. 'Health', 'Mana'",
                    },
                    "value": {"type": "number"},
                },
                "required": ["actorPath", "attributeName", "value"],
            },
        ),
        Tool(
            name="gas_send_gameplay_event",
            description="Send a Gameplay Event tag to an actor (can trigger abilities).",
            inputSchema={
                "type": "object",
                "properties": {
                    "actorPath": {"type": "string"},
                    "eventTag": {
                        "type": "string",
                        "description": "Event tag, e.g. 'Event.Damage.Physical'",
                    },
                    "magnitude": {
                        "type": "number",
                        "description": "Event magnitude payload (optional)",
                        "default": 0,
                    },
                },
                "required": ["actorPath", "eventTag"],
            },
        ),
        Tool(
            name="gas_list_granted_abilities",
            description="List all abilities currently granted to an actor's ASC.",
            inputSchema={
                "type": "object",
                "properties": {
                    "actorPath": {"type": "string"}
                },
                "required": ["actorPath"],
            },
        ),
        Tool(
            name="gas_inspect_effect_asset",
            description=(
                "Inspect a Gameplay Effect Blueprint asset at editor time: "
                "list its modifiers, duration policy, and granted tags."
            ),
            inputSchema={
                "type": "object",
                "properties": {
                    "effectAssetPath": {
                        "type": "string",
                        "description": "Content path, e.g. '/Game/Abilities/GE_FireDamage'",
                    }
                },
                "required": ["effectAssetPath"],
            },
        ),
    ]


@app.call_tool()
async def call_tool(name: str, arguments: dict):
    async with httpx.AsyncClient() as client:

        if name == "gas_get_attributes":
            actor_path = _validate_actor_path(arguments["actorPath"])
            code = f"""
import unreal, json
actor = unreal.find_object(None, '{actor_path}')
asc = actor.get_component_by_class(unreal.AbilitySystemComponent)
attr_set = asc.get_attribute_set(unreal.AttributeSet)
attrs = {{}}
for attr in dir(attr_set):
    if not attr.startswith('_'):
        val = getattr(attr_set, attr, None)
        if isinstance(val, float):
            attrs[attr] = val
print(json.dumps(attrs))
"""
            result = await ue5_python(client, code)
            return [TextContent(type="text", text=result)]

        if name == "gas_get_active_effects":
            actor_path = _validate_actor_path(arguments["actorPath"])
            code = f"""
import unreal, json
actor = unreal.find_object(None, '{actor_path}')
asc = actor.get_component_by_class(unreal.AbilitySystemComponent)
effects = asc.get_active_effects_with_all_tags(unreal.GameplayTagContainer())
result = [str(e) for e in effects]
print(json.dumps(result))
"""
            result = await ue5_python(client, code)
            return [TextContent(type="text", text=result)]

        if name == "gas_get_active_tags":
            actor_path = _validate_actor_path(arguments["actorPath"])
            code = f"""
import unreal, json
actor = unreal.find_object(None, '{actor_path}')
asc = actor.get_component_by_class(unreal.AbilitySystemComponent)
tags = asc.get_owned_gameplay_tags()
tag_list = [str(t) for t in tags]
print(json.dumps(tag_list))
"""
            result = await ue5_python(client, code)
            return [TextContent(type="text", text=result)]

        if name == "gas_activate_ability":
            actor_path = _validate_actor_path(arguments["actorPath"])
            ability_tag = _validate_tag(arguments["abilityTag"])
            code = f"""
import unreal
actor = unreal.find_object(None, '{actor_path}')
asc = actor.get_component_by_class(unreal.AbilitySystemComponent)
tag = unreal.GameplayTag.request_gameplay_tag('{ability_tag}')
container = unreal.GameplayTagContainer()
container.add_tag(tag)
success = asc.try_activate_abilities_by_tag(container)
print(f'Activated: {{success}}')
"""
            result = await ue5_python(client, code)
            return [TextContent(type="text", text=result)]

        if name == "gas_apply_effect":
            actor_path = _validate_actor_path(arguments["actorPath"])
            effect_path = _validate_asset_path(arguments["effectClassPath"])
            level = _validate_number(arguments.get("level", 1))
            code = f"""
import unreal
actor = unreal.find_object(None, '{actor_path}')
asc = actor.get_component_by_class(unreal.AbilitySystemComponent)
effect_class = unreal.load_class(None, '{effect_path}')
ctx = asc.make_outgoing_spec(effect_class, {level}, asc.make_effect_context())
asc.apply_gameplay_effect_spec_to_self(ctx)
print('Effect applied')
"""
            result = await ue5_python(client, code)
            return [TextContent(type="text", text=result)]

        if name == "gas_set_attribute":
            actor_path = _validate_actor_path(arguments["actorPath"])
            attr_name = _validate_attr_name(arguments["attributeName"])
            value = _validate_number(arguments["value"])
            result = await ue5_call(
                client,
                actor_path,
                "SetAttributeValue",
                {
                    "AttributeName": attr_name,
                    "Value": value,
                },
            )
            return [TextContent(type="text", text=result)]

        if name == "gas_send_gameplay_event":
            actor_path = _validate_actor_path(arguments["actorPath"])
            event_tag = _validate_tag(arguments["eventTag"])
            magnitude = _validate_number(arguments.get("magnitude", 0))
            code = f"""
import unreal
actor = unreal.find_object(None, '{actor_path}')
tag = unreal.GameplayTag.request_gameplay_tag('{event_tag}')
payload = unreal.GameplayEventData()
payload.event_magnitude = {magnitude}
unreal.AbilitySystemBlueprintLibrary.send_gameplay_event_to_actor(actor, tag, payload)
print('Event sent')
"""
            result = await ue5_python(client, code)
            return [TextContent(type="text", text=result)]

        if name == "gas_list_granted_abilities":
            actor_path = _validate_actor_path(arguments["actorPath"])
            code = f"""
import unreal, json
actor = unreal.find_object(None, '{actor_path}')
asc = actor.get_component_by_class(unreal.AbilitySystemComponent)
specs = asc.get_activatable_abilities()
result = [str(s.ability.get_class().get_name()) for s in specs]
print(json.dumps(result))
"""
            result = await ue5_python(client, code)
            return [TextContent(type="text", text=result)]

        if name == "gas_inspect_effect_asset":
            asset_path = _validate_asset_path(arguments["effectAssetPath"])
            code = f"""
import unreal, json
ge = unreal.load_asset('{asset_path}')
info = {{
    'class': ge.get_class().get_name(),
    'duration_policy': str(ge.duration_policy),
    'period': ge.period,
    'modifiers': len(ge.modifiers),
}}
print(json.dumps(info))
"""
            result = await ue5_python(client, code)
            return [TextContent(type="text", text=result)]

        return [TextContent(type="text", text=f"Unknown tool: {name}")]


async def main():
    async with stdio_server() as streams:
        await app.run(*streams, app.create_initialization_options())


asyncio.run(main())
```

### Install Dependencies

```bash
pip install mcp httpx
```

### Claude Desktop Configuration

```json
{
  "mcpServers": {
    "ue5-gas": {
      "command": "python",
      "args": ["/path/to/ue5_gas_mcp_server.py"]
    }
  }
}
```

### VS Code / GitHub Copilot Configuration

Add to `.vscode/mcp.json` in your project root (VS Code 1.99+):

```json
{
  "servers": {
    "ue5-gas": {
      "type": "stdio",
      "command": "python",
      "args": ["${workspaceFolder}/tools/ue5_gas_mcp_server.py"]
    }
  }
}
```

### Cursor IDE Configuration

In Cursor, go to **Settings > Features > MCP** and add:

```json
{
  "ue5-gas": {
    "command": "python",
    "args": ["/path/to/ue5_gas_mcp_server.py"]
  }
}
```

## Example AI Agent Workflow

With the GAS MCP server running, an AI agent can perform a complete ability debug session:

```
Agent: "The hero is taking too much fire damage. List their active effects and current health."

→ gas_get_active_effects(actorPath="/Game/Maps/L_Test.L_Test:PersistentLevel.BP_Hero_0")
→ gas_get_attributes(actorPath="...")

Agent: "Apply the fire resistance effect and confirm the health attribute is stable."

→ gas_apply_effect(actorPath="...", effectClassPath="/Game/Abilities/GE_FireResistance.GE_FireResistance_C")
→ gas_get_attributes(actorPath="...")

Agent: "Now trigger the Fireball ability and check tags after activation."

→ gas_activate_ability(actorPath="...", abilityTag="Ability.Fireball")
→ gas_get_active_tags(actorPath="...")
```

## Editor-Time GAS Automation via Python

For batch operations on GAS assets in the editor (without the MCP server running), use the UE5 Python API directly:

### List All Gameplay Effect Assets

```python
import unreal

ar = unreal.AssetRegistry.get()
filter = unreal.ARFilter(
    class_names=["GameplayEffect"],
    recursive_classes=True
)
assets = ar.get_assets(filter)
for asset in assets:
    print(f"{asset.asset_name}: {asset.object_path}")
```

### Bulk-Update GE Magnitudes

```python
import unreal

ge_path = "/Game/Abilities/GE_FireDamage.GE_FireDamage"
ge = unreal.load_asset(ge_path)
for modifier in ge.modifiers:
    if str(modifier.modifier_op) == "GameplayModOp::Additive":
        modifier.modifier_magnitude.scalable_float_magnitude = 50.0
unreal.EditorAssetLibrary.save_asset(ge_path)
print("GE magnitude updated")
```

### Generate a GAS Attribute Report

```python
import unreal, csv, os

actors = unreal.EditorLevelLibrary.get_all_level_actors()
rows = []
for actor in actors:
    asc = actor.get_component_by_class(unreal.AbilitySystemComponent)
    if asc:
        rows.append({
            "Actor": actor.get_name(),
            "Health": asc.get_numeric_attribute(
                unreal.AttributeSet.get_health_attribute()),
            "Mana": asc.get_numeric_attribute(
                unreal.AttributeSet.get_mana_attribute()),
        })

report_path = os.path.expanduser("~/gas_attribute_report.csv")
with open(report_path, "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["Actor", "Health", "Mana"])
    writer.writeheader()
    writer.writerows(rows)
print(f"Report written to {report_path}")
```

## Available UE5 Python APIs for GAS

| API | Purpose |
|-----|---------|
| `unreal.AbilitySystemComponent` | Core ASC operations |
| `unreal.AbilitySystemBlueprintLibrary` | Static helper functions (send events, apply effects) |
| `unreal.GameplayStatics` | World-level GAS queries |
| `unreal.AttributeSet` | Attribute value access |
| `unreal.GameplayTagsManager` | Tag registration and lookup |
| `unreal.AssetRegistry` | Find GAS assets by class |
| `unreal.EditorAssetLibrary` | Save/load GAS assets |

## Combining with the Base UE5 MCP Server

For full coverage, run both the base UE5 MCP server and the GAS-specific server simultaneously:

```json
{
  "mcpServers": {
    "unreal": {
      "command": "python",
      "args": ["/path/to/ue5_mcp_server.py"]
    },
    "ue5-gas": {
      "command": "python",
      "args": ["/path/to/ue5_gas_mcp_server.py"]
    }
  }
}
```

## References

- [GAS Tool Documentation](../tools/gameplay-ability-system.md)
- [UE5 MCP Server Base](unreal-engine.md)
- [MCP Overview](overview.md)
- [GASDocumentation (Tranek)](https://github.com/tranek/GASDocumentation)
- [Lyra Starter Game](https://docs.unrealengine.com/5.0/en-US/lyra-sample-game-in-unreal-engine/)
- [UE5 Remote Control API](https://docs.unrealengine.com/5.0/en-US/remote-control-api-for-unreal-engine/)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
