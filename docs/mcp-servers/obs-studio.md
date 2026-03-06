# OBS Studio MCP Server

## Overview

OBS Studio can be controlled remotely via its built-in **WebSocket server** (v5 protocol), which functions as a de-facto MCP-compatible interface. AI agents can use this to automate streaming, recording, scene management, and source control.

## WebSocket Server Setup

### Enable in OBS

1. Open OBS Studio.
2. Go to **Tools > WebSocket Server Settings**.
3. Enable **WebSocket server**.
4. Set a **port** (default: `4455`) and optionally a **password**.
5. Click OK — the server starts immediately.

### Connection Details

```
Protocol: obs-websocket v5
Host:     localhost (or remote IP)
Port:     4455 (default)
Password: (optional, recommended)
```

## Using obs-websocket-py (Python Client)

```bash
pip install obs-websocket-py
```

```python
from obswebsocket import obsws, requests as obsreq

# Connect
ws = obsws("localhost", 4455, "your-password")
ws.connect()

# Get scene list
response = ws.call(obsreq.GetSceneList())
print(response.getScenes())

# Switch scene
ws.call(obsreq.SetCurrentProgramScene(sceneName="Gameplay"))

# Start recording
ws.call(obsreq.StartRecord())

# Stop recording and get output path
response = ws.call(obsreq.StopRecord())
print(f"Saved to: {response.getOutputPath()}")

# Take screenshot
ws.call(obsreq.SaveSourceScreenshot(
    sourceName="Game Capture",
    imageFormat="png",
    imageFilePath="/tmp/screenshot.png"
))

ws.disconnect()
```

## Building an OBS MCP Server

Wrap the OBS WebSocket API in an MCP server for AI agent access:

```python
# obs_mcp_server.py
import asyncio
from obswebsocket import obsws, requests as obsreq
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

OBS_HOST = "localhost"
OBS_PORT = 4455
OBS_PASSWORD = "your-password"

app = Server("obs-server")
obs_client = None

def get_obs():
    global obs_client
    if obs_client is None:
        obs_client = obsws(OBS_HOST, OBS_PORT, OBS_PASSWORD)
        obs_client.connect()
    return obs_client

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="obs_start_recording",
            description="Start OBS recording",
            inputSchema={"type": "object", "properties": {}}
        ),
        Tool(
            name="obs_stop_recording",
            description="Stop OBS recording and return output path",
            inputSchema={"type": "object", "properties": {}}
        ),
        Tool(
            name="obs_switch_scene",
            description="Switch to a named OBS scene",
            inputSchema={
                "type": "object",
                "properties": {
                    "scene_name": {"type": "string", "description": "Name of the scene to switch to"}
                },
                "required": ["scene_name"]
            }
        ),
        Tool(
            name="obs_list_scenes",
            description="List all available OBS scenes",
            inputSchema={"type": "object", "properties": {}}
        ),
        Tool(
            name="obs_screenshot",
            description="Take a screenshot of a source",
            inputSchema={
                "type": "object",
                "properties": {
                    "source_name": {"type": "string"},
                    "output_path": {"type": "string"}
                },
                "required": ["source_name", "output_path"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    obs = get_obs()
    
    if name == "obs_start_recording":
        obs.call(obsreq.StartRecord())
        return [TextContent(type="text", text="Recording started")]
    
    elif name == "obs_stop_recording":
        response = obs.call(obsreq.StopRecord())
        path = response.getOutputPath()
        return [TextContent(type="text", text=f"Recording saved to: {path}")]
    
    elif name == "obs_switch_scene":
        obs.call(obsreq.SetCurrentProgramScene(sceneName=arguments["scene_name"]))
        return [TextContent(type="text", text=f"Switched to scene: {arguments['scene_name']}")]
    
    elif name == "obs_list_scenes":
        response = obs.call(obsreq.GetSceneList())
        scenes = [s["sceneName"] for s in response.getScenes()]
        return [TextContent(type="text", text="\n".join(scenes))]
    
    elif name == "obs_screenshot":
        obs.call(obsreq.SaveSourceScreenshot(
            sourceName=arguments["source_name"],
            imageFormat="png",
            imageFilePath=arguments["output_path"]
        ))
        return [TextContent(type="text", text=f"Screenshot saved to: {arguments['output_path']}")]

async def main():
    async with stdio_server() as streams:
        await app.run(*streams, app.create_initialization_options())

asyncio.run(main())
```

**Install dependencies:**
```bash
pip install mcp obs-websocket-py
```

**Claude Desktop config:**
```json
{
  "mcpServers": {
    "obs": {
      "command": "python",
      "args": ["/path/to/obs_mcp_server.py"]
    }
  }
}
```

## OBS WebSocket v5 Key Requests

| Request | Description |
|---------|-------------|
| `GetSceneList` | List all scenes |
| `SetCurrentProgramScene` | Switch to a scene |
| `StartRecord` | Begin recording |
| `StopRecord` | Stop recording |
| `StartStream` | Begin streaming |
| `StopStream` | Stop streaming |
| `GetRecordStatus` | Check if recording |
| `GetStreamStatus` | Check if streaming |
| `GetSourceScreenshot` | Capture a source to base64 |
| `SaveSourceScreenshot` | Capture a source to file |
| `SetInputVolume` | Change audio input volume |
| `SetInputMute` | Mute/unmute an input |
| `GetInputList` | List all inputs/sources |
| `CreateInput` | Add a new source |
| `RemoveInput` | Delete a source |
| `SetSceneItemEnabled` | Show/hide a source in a scene |

## UE5 Pipeline Integration

AI-agent automation use cases for OBS in the UE5 pipeline:

1. **Auto-record playtests** — agent starts OBS recording before launching PIE in UE5.
2. **Development streams** — agent switches OBS scenes as workflow changes (coding → viewport → review).
3. **Capture reference footage** — agent records in-engine references for animation or VFX.
4. **Screenshot comparisons** — agent captures before/after screenshots when testing visual changes.

## References

- [obs-websocket Protocol v5](https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md)
- [obs-websocket-py](https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md)
- [OBS WebSocket GitHub](https://github.com/obsproject/obs-websocket)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
