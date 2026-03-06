# MCP Servers Overview

## What is MCP?

Model Context Protocol (MCP) is an open standard that allows AI agents (Claude, Copilot, Cursor, etc.) to connect to and interact with external tools, data sources, and applications through a unified interface. MCP servers expose tools and resources that AI agents can call to take actions in external applications.

## MCP Servers for the UE5 Stack

| Tool | MCP Server | Status | Notes |
|------|------------|--------|-------|
| Blender | blender-mcp | ✅ Available | Full 3D control |
| Unreal Engine 5 | ue5-mcp / custom | 🔧 Community/Custom | Via Python scripting |
| OBS Studio | obs-mcp (WebSocket) | ✅ Available | Via obs-websocket |
| GIMP | None official | ❌ Not available | Use Script-Fu batch mode |
| Krita | None official | ❌ Not available | Use Python batch scripts |
| Material Maker | None official | ❌ Not available | Use CLI headless mode |
| Natron | None official | ❌ Not available | Use Python + CLI |
| Sequencer (UE5) | Via UE5 Python | 🔧 Indirect | UE Python scripting |
| Niagara (UE5) | Via UE5 Python | 🔧 Indirect | UE Python scripting |

## Quick Setup Guides

### Blender MCP

See [docs/mcp-servers/blender.md](blender.md) for full setup.

**Quick start:**
```bash
# 1. Clone blender-mcp
git clone https://github.com/ahujasid/blender-mcp

# 2. Install the Blender add-on from blender-mcp/blender_addon/
# Blender: Edit > Preferences > Add-ons > Install > blender_mcp.zip

# 3. Install Python client
pip install blender-mcp

# 4. Enable addon in Blender (sidebar > BlenderMCP > Connect)

# 5. Configure in your MCP client (e.g., Claude Desktop):
# Add to claude_desktop_config.json:
{
  "mcpServers": {
    "blender": {
      "command": "uvx",
      "args": ["blender-mcp"]
    }
  }
}
```

### Unreal Engine 5 (via Python)

See [docs/mcp-servers/unreal-engine.md](unreal-engine.md) for full setup.

**Quick start:**
```json
// Claude Desktop config — UE5 Python server
{
  "mcpServers": {
    "unreal": {
      "command": "python",
      "args": ["/path/to/ue5-mcp-server/server.py"],
      "env": {
        "UE_PROJECT_PATH": "C:/MyProject/MyProject.uproject"
      }
    }
  }
}
```

### OBS Studio (via WebSocket)

See [docs/mcp-servers/obs-studio.md](obs-studio.md) for full setup.

```bash
# 1. Enable OBS WebSocket Server in OBS:
#    Tools > WebSocket Server Settings

# 2. Install obs-mcp or use obs-websocket directly
pip install obs-websocket-py

# 3. Configure MCP server
```

## MCP Client Configuration

### Claude Desktop

Edit `~/.config/claude_desktop_config.json` (Mac/Linux) or `%APPDATA%/Claude/claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "blender": {
      "command": "uvx",
      "args": ["blender-mcp"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/project"]
    }
  }
}
```

### Cursor IDE

In Cursor settings, navigate to **Features > MCP** and add server configurations.

### VS Code (GitHub Copilot)

Add to `.vscode/settings.json`:
```json
{
  "github.copilot.advanced": {
    "mcpServers": {
      "blender": {
        "command": "uvx",
        "args": ["blender-mcp"]
      }
    }
  }
}
```

## Building a Custom MCP Server for UE5 Tools

For tools without existing MCP servers, you can build a custom one:

### Python MCP Server Template

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent
import asyncio

app = Server("my-tool-server")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="do_something",
            description="Perform an action in My Tool",
            inputSchema={
                "type": "object",
                "properties": {
                    "parameter": {"type": "string", "description": "What to do"}
                },
                "required": ["parameter"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "do_something":
        result = perform_action(arguments["parameter"])
        return [TextContent(type="text", text=str(result))]

async def main():
    async with stdio_server() as streams:
        await app.run(*streams, app.create_initialization_options())

asyncio.run(main())
```

### Installing the MCP Python SDK

```bash
pip install mcp
# or with uvx
pip install uv
uvx mcp
```

## Recommended MCP Ecosystem Tools

| MCP Server | Package | Purpose |
|------------|---------|---------|
| Filesystem | `@modelcontextprotocol/server-filesystem` | Read/write project files |
| Git | `@modelcontextprotocol/server-git` | Git operations |
| Fetch | `@modelcontextprotocol/server-fetch` | HTTP requests for API access |
| SQLite | `@modelcontextprotocol/server-sqlite` | Query asset databases |
| Blender | `blender-mcp` | 3D modeling in Blender |

## Resources

- [Model Context Protocol Specification](https://modelcontextprotocol.io)
- [MCP GitHub Organization](https://github.com/modelcontextprotocol)
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [Blender MCP](https://github.com/ahujasid/blender-mcp)
