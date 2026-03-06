# OBS Studio

## Overview

OBS Studio (Open Broadcaster Software) is a free, open-source tool for live streaming and screen/gameplay recording. In the UE5 pipeline, it is used for capturing development sessions, recording in-editor walkthroughs, streaming game test builds, creating development video logs, and capturing reference footage for animation.

## Key Features

- **Multi-source capture** — game capture, window capture, screen capture, webcam, audio.
- **Scene composition** — multiple named scenes with transitions between them.
- **Virtual Camera** — output OBS as a virtual webcam device.
- **Plugin ecosystem** — extensive plugin library for advanced features.
- **Recording formats** — MKV, MP4, FLV, or lossless formats.
- **Streaming support** — Twitch, YouTube, custom RTMP, and more.
- **Free and open source** — [obsproject.com](https://obsproject.com)

## Key Concepts

### Scenes

A **Scene** is a named composition of sources. Switch between scenes instantly via hotkeys or the scene list.

### Sources

| Source Type | Use Case |
|-------------|---------|
| **Game Capture** | Capture a specific game/application with low overhead |
| **Window Capture** | Capture any application window |
| **Display Capture** | Capture an entire monitor |
| **Video Capture Device** | Webcam, capture card, virtual camera |
| **Audio Input Capture** | Microphone or line-in |
| **Audio Output Capture** | System audio / game audio |
| **Image** | Static overlay image |
| **Text (GDI+)** | Overlay text labels |
| **Browser Source** | Embed a web page (for alerts, chat overlays) |
| **Media Source** | Play video/audio files on stream |

### Filters

Filters can be applied per-source or per-scene:
- **Chroma Key** — green screen removal
- **Color Correction** — brightness, contrast, saturation, gamma
- **Crop/Pad** — crop a source
- **Noise Gate / Noise Suppression** — clean up audio
- **Gain** — amplify audio signal

## Common UE5 Pipeline Uses

### Recording UE5 Editor Sessions

1. Add a **Window Capture** source and select "Unreal Editor".
2. Or add a **Display Capture** if using full-screen editor.
3. Set recording format to **MKV** (most stable for long recordings).
4. Click **Start Recording**.

> **Tip**: For in-editor captures, use UE5's built-in **viewport screenshot** (`Ctrl+F9`) or **Movie Render Queue** for final quality output instead of OBS.

### Game Capture for Playtesting

1. Add a **Game Capture** source.
2. Select the UE5 project executable or "Capture any fullscreen application".
3. Configure hotkeys for start/stop recording.
4. Record playtests for review and bug documentation.

### Streaming UE5 Development

1. Configure streaming service in **Settings > Stream**.
2. Add scenes: "Coding" (IDE focus), "Viewport" (Unreal Editor), "Webcam Only" (breaks).
3. Use **Studio Mode** to preview the next scene before switching live.

### Virtual Camera Output

OBS can output as a virtual webcam:
1. **Tools > VirtualCam > Start**.
2. Use in video calls, or pipe into UE5 via Live Link Face or a video capture device node.

## Settings for High-Quality Recording

### Video Settings

```
Settings > Video:
- Base (Canvas) Resolution: 1920x1080 or 2560x1440
- Output (Scaled) Resolution: 1920x1080
- Downscale Filter: Lanczos
- FPS: 60
```

### Output Settings (Recording)

```
Settings > Output > Recording:
- Recording Format: MKV (remux to MP4 after if needed)
- Encoder: NVIDIA NVENC H.264 (GPU) or x264 (CPU)
- Rate Control: CQP (quality-based)
- CQ Level: 18 (lower = higher quality; 0 = lossless)
- Keyframe Interval: 2
- Preset: Quality
```

For **lossless recording** (best for video editing in Natron):
```
- Encoder: FFMPEG (Custom Output - Advanced) 
- Container: MKV
- Video Codec: utvideo or FFV1
```

### Audio Settings

```
Settings > Audio:
- Sample Rate: 48 kHz
- Channels: Stereo
- Desktop Audio: Default
- Mic/Auxiliary Audio: Your microphone
```

## OBS WebSocket (Remote Control)

OBS Studio includes a **WebSocket server** for remote control:

1. **Tools > WebSocket Server Settings** — enable and set a password.
2. Default port: `4455`.
3. Use the [obs-websocket](https://github.com/obsproject/obs-websocket) protocol.

Example with Python (`obs-websocket-py`):

```python
from obswebsocket import obsws, requests

client = obsws("localhost", 4455, "your-password")
client.connect()

# Switch scene
client.call(requests.SetCurrentScene("Gameplay"))

# Start recording
client.call(requests.StartRecording())

# Stop recording
client.call(requests.StopRecording())

client.disconnect()
```

## Useful Plugins

| Plugin | Purpose |
|--------|---------|
| **Move Transition** | Animated transitions between scenes |
| **Source Clone** | Clone a source without duplicating settings |
| **Advanced Scene Switcher** | Auto-switch scenes based on conditions (window focus, time, etc.) |
| **OBS-NDI** | NDI protocol support for low-latency LAN streaming |
| **obs-ndi** | Network Device Interface source/output |
| **obs-shaderfilter** | Apply custom GLSL shaders as filters |
| **Aitum Vertical** | Vertical/portrait canvas for mobile streams |

## AI / MCP Integration

OBS Studio has a WebSocket API (v5) that acts as a functional equivalent of an MCP server for streaming/recording control:

- **obs-websocket** — remote control OBS via WebSocket protocol.
- An AI agent can start/stop recording, switch scenes, control sources, and respond to stream events automatically.

See [docs/mcp-servers/obs-studio.md](../mcp-servers/obs-studio.md) for the MCP server setup.

## References

- [OBS Studio Documentation](https://obsproject.com/wiki/)
- [obs-websocket Protocol](https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md)
- [OBS Forum](https://obsproject.com/forum/)
