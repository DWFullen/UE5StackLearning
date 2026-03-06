# Blender MCP Server

## Overview

The **blender-mcp** server allows AI agents to control Blender 3D directly through the Model Context Protocol. Using this server, an agent can create objects, modify materials, run Python scripts, render scenes, and export assets without manual Blender interaction.

## Repository

- **GitHub**: [ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp)
- **PyPI**: `blender-mcp`

## How It Works

```
AI Agent (Claude/Cursor)
        │
        │ MCP Protocol
        ▼
blender-mcp Python Server (stdio)
        │
        │ TCP Socket (localhost:9876)
        ▼
Blender Add-on (addon_server.py)
        │
        │ bpy Python API
        ▼
Blender Application
```

The blender-mcp server consists of two parts:
1. **Blender Add-on** — runs inside Blender, listens on a local TCP socket.
2. **MCP Server** — Python process that translates MCP calls to socket messages.

## Installation

### Step 1: Install the Blender Add-on

1. Download the latest release from [blender-mcp releases](https://github.com/ahujasid/blender-mcp/releases).
2. In Blender: **Edit > Preferences > Add-ons > Install**.
3. Select the downloaded `addon.zip` file.
4. Enable the add-on: search for "Blender MCP" and check the box.

### Step 2: Install the MCP Server

```bash
# Using pip
pip install blender-mcp

# Using uv (recommended)
pip install uv
```

### Step 3: Start the Blender Add-on Server

1. In Blender, open the **N Panel** (press `N` in the 3D viewport).
2. Navigate to the **BlenderMCP** tab.
3. Click **Start MCP Server** — the status should show "Running on port 9876".

### Step 4: Configure Your AI Client

**Claude Desktop** (`claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "blender": {
      "command": "uvx",
      "args": ["blender-mcp"]
    }
  }
}
```

**Cursor** (`.cursor/mcp.json`):
```json
{
  "mcpServers": {
    "blender": {
      "command": "uvx",
      "args": ["blender-mcp"]
    }
  }
}
```

## Available Tools

| Tool | Description |
|------|-------------|
| `get_scene_info` | Get overview of current scene objects and settings |
| `get_object_info` | Detailed info about a specific object |
| `create_object` | Create primitive mesh objects |
| `modify_object` | Modify object transform, visibility, and properties |
| `delete_object` | Delete an object from the scene |
| `set_material` | Create and assign materials with PBR properties |
| `render_scene` | Render the current scene and return the image |
| `execute_blender_code` | Execute arbitrary Python code in Blender |
| `get_viewport_screenshot` | Take a screenshot of the current viewport |
| `set_render_settings` | Configure render engine, resolution, samples |
| `export_scene` | Export scene to FBX, OBJ, or glTF |

## Example Agent Interactions

### Create and Position Objects

```
User: Create a low-poly tree made of a cylinder trunk and a sphere foliage ball
Agent → blender-mcp:
  1. create_object(type="CYLINDER", location=[0,0,0.5], scale=[0.2,0.2,0.5])
  2. create_object(type="SPHERE", location=[0,0,1.5], scale=[0.8,0.8,0.8])
  3. set_material(object="Cylinder", color=[0.4,0.2,0.1])
  4. set_material(object="Sphere", color=[0.1,0.6,0.1])
```

### Export for UE5

```
User: Export the scene as FBX for Unreal Engine 5
Agent → blender-mcp:
  1. execute_blender_code(code="""
       import bpy
       bpy.ops.export_scene.fbx(
           filepath='/tmp/scene.fbx',
           apply_scale_options='FBX_SCALE_ALL',
           axis_forward='-Y',
           axis_up='Z'
       )
     """)
```

### Render a Preview

```
User: Render the current scene and show me what it looks like
Agent → blender-mcp:
  1. set_render_settings(engine="CYCLES", samples=32, resolution_x=1280, resolution_y=720)
  2. render_scene(output_path="/tmp/render.png")
```

## UE5 Integration Workflow

Using blender-mcp in an AI-driven UE5 asset creation pipeline:

```
1. Agent receives mesh requirements (poly count, UV requirements, scale)
2. Agent uses blender-mcp to model/import/modify the mesh in Blender
3. Agent exports FBX via blender-mcp execute_blender_code
4. Agent uses UE5 Python MCP to import FBX into UE5 Content Browser
5. Agent configures materials and LODs in UE5
```

## Troubleshooting

| Issue | Solution |
|-------|---------|
| "Connection refused" | Ensure the Blender add-on server is running (N Panel > BlenderMCP > Start) |
| "Add-on not found" | Reinstall the add-on zip; ensure the correct Blender version |
| Blender crashes | Run `execute_blender_code` in try/except; check Blender system console |
| Slow rendering | Reduce samples; use EEVEE instead of Cycles for preview |

## References

- [blender-mcp GitHub](https://github.com/ahujasid/blender-mcp)
- [MCP Specification](https://modelcontextprotocol.io)
- [Blender Python API](https://docs.blender.org/api/current/)
