# Unreal Engine 5 MCP Server

## Overview

There is no single official MCP server for Unreal Engine 5 from Epic Games, but UE5's built-in **Python scripting** system provides a powerful foundation for building one. This document covers the available approaches to give AI agents control over the Unreal Editor and runtime gameplay.

## Approaches

### 1. UE5 Python Remote Control (Built-in)

Unreal Engine 5 includes a **Remote Control API** plugin and a **Python Editor Script Plugin** that expose HTTP and WebSocket endpoints, enabling agents to control the editor without writing a native MCP server.

**Plugins required:**
- `Python Editor Script Plugin` — enables Python scripting in the editor.
- `Remote Control API` — HTTP REST + WebSocket endpoint.
- `Remote Control Web Interface` — Optional web UI for testing.

**Enable in Project Settings:**
```
Edit > Plugins > search "Remote Control API" → enable
Edit > Plugins > search "Python Editor Script Plugin" → enable
```

**REST API endpoint (default):**
```
http://localhost:30010/remote/object/call
http://localhost:30010/remote/object/property
```

**Example: Call a Blueprint function via REST:**
```bash
curl -X PUT http://localhost:30010/remote/object/call \
  -H "Content-Type: application/json" \
  -d '{
    "objectPath": "/Game/Maps/TestLevel.TestLevel:PersistentLevel.BP_MyActor_0",
    "functionName": "MyFunction",
    "parameters": {
      "Speed": 500.0
    }
  }'
```

### 2. UE5 Python MCP Server (Custom)

Build a custom MCP server that wraps UE5's Python API:

**Architecture:**
```
AI Agent → MCP stdio → Python MCP Server → UE5 Python API (via subprocess/socket)
                                          └→ UE5 Remote Control HTTP API
```

**Example server skeleton:**
```python
# ue5_mcp_server.py
import asyncio
import httpx
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

UE5_REST_URL = "http://localhost:30010"

app = Server("ue5-server")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="ue5_call_function",
            description="Call a function on a UE5 actor",
            inputSchema={
                "type": "object",
                "properties": {
                    "objectPath": {"type": "string"},
                    "functionName": {"type": "string"},
                    "parameters": {"type": "object"}
                },
                "required": ["objectPath", "functionName"]
            }
        ),
        Tool(
            name="ue5_run_python",
            description="Execute Python code in the Unreal Editor",
            inputSchema={
                "type": "object",
                "properties": {
                    "code": {"type": "string", "description": "Python code to execute"}
                },
                "required": ["code"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    async with httpx.AsyncClient() as client:
        if name == "ue5_call_function":
            response = await client.put(
                f"{UE5_REST_URL}/remote/object/call",
                json={
                    "objectPath": arguments["objectPath"],
                    "functionName": arguments["functionName"],
                    "parameters": arguments.get("parameters", {})
                }
            )
            return [TextContent(type="text", text=response.text)]

        if name == "ue5_run_python":
            response = await client.put(
                f"{UE5_REST_URL}/remote/object/call",
                json={
                    "objectPath": "/Script/PythonScriptPlugin.Default__PythonScriptLibrary",
                    "functionName": "ExecutePythonCommand",
                    "parameters": {"PythonCommand": arguments["code"]}
                }
            )
            return [TextContent(type="text", text=response.text)]

async def main():
    async with stdio_server() as streams:
        await app.run(*streams, app.create_initialization_options())

asyncio.run(main())
```

**Install dependencies:**
```bash
pip install mcp httpx
```

**Claude Desktop config:**
```json
{
  "mcpServers": {
    "unreal": {
      "command": "python",
      "args": ["/path/to/ue5_mcp_server.py"]
    }
  }
}
```

### 3. Editor Utility Widgets (Blueprint-based Agent Tools)

For designers working with AI-assisted workflows, **Editor Utility Widgets** provide a Blueprint-native way to expose AI-callable tools directly inside the Unreal Editor UI.

1. Create an **Editor Utility Widget Blueprint**.
2. Add buttons or text fields for each AI action.
3. Implement actions that call Python or C++ functions.
4. Open via **Tools > Run Editor Utility Widget**.

## Available UE5 Python APIs for Agent Automation

| API | Purpose |
|-----|---------|
| `unreal.EditorLevelLibrary` | Spawn, delete, move actors |
| `unreal.EditorAssetLibrary` | Import, duplicate, delete assets |
| `unreal.EditorUtilityLibrary` | Get selected assets/actors |
| `unreal.LevelSequenceEditorBlueprintLibrary` | Control Sequencer |
| `unreal.NiagaraFunctionLibrary` | Spawn Niagara systems |
| `unreal.MaterialEditingLibrary` | Create and edit materials |
| `unreal.StaticMeshEditorSubsystem` | LOD generation, UV calculation |
| `unreal.AutomationLibrary` | Run automation tests |
| `unreal.PythonScriptLibrary` | Execute Python commands |

## Example Agent Tasks via UE5 Python

### Import an FBX Mesh

```python
import unreal

task = unreal.AssetImportTask()
task.filename = "C:/Assets/MyMesh.fbx"
task.destination_path = "/Game/Meshes/"
task.automated = True
task.save = True

options = unreal.FbxImportUI()
options.import_mesh = True
options.import_as_skeletal = False
task.options = options

unreal.AssetToolsHelpers.get_asset_tools().import_asset_tasks([task])
```

### Place an Actor in the Level

```python
import unreal

# Load an asset
mesh = unreal.load_asset('/Game/Meshes/MyMesh.MyMesh')

# Spawn a Static Mesh Actor
location = unreal.Vector(0.0, 0.0, 100.0)
rotation = unreal.Rotator(0.0, 0.0, 0.0)
actor = unreal.EditorLevelLibrary.spawn_actor_from_object(mesh, location, rotation)
actor.set_actor_label("AI_PlacedMesh")
```

### Batch Rename Assets

```python
import unreal

assets = unreal.EditorUtilityLibrary.get_selected_assets()
for i, asset in enumerate(assets):
    old_name = asset.get_name()
    new_name = f"SM_BatchRenamed_{i:03d}"
    unreal.EditorAssetLibrary.rename_asset(asset.get_path_name(), f"/Game/Meshes/{new_name}")
```

## Remote Control API Reference

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/remote/object/call` | PUT | Call a function on a UObject |
| `/remote/object/property` | GET | Get a property value |
| `/remote/object/property` | PUT | Set a property value |
| `/remote/object/describe` | GET | Get object metadata |
| `/remote/preset/` | GET | List Remote Control presets |

Enable the **Remote Control Web Interface** plugin for an interactive browser-based API explorer at `http://localhost:30010`.

## Community MCP Projects

| Project | URL | Status |
|---------|-----|--------|
| ue5-mcp (community) | Search GitHub | Various |
| Unreal Python Remote | Built-in | ✅ Stable |

## References

- [Unreal Python API](https://docs.unrealengine.com/5.0/en-US/scripting-the-unreal-editor-using-python/)
- [Remote Control API](https://docs.unrealengine.com/5.0/en-US/remote-control-api-for-unreal-engine/)
- [Remote Control Web Interface](https://docs.unrealengine.com/5.0/en-US/remote-control-web-interface-for-unreal-engine/)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
