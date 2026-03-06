# Natron

## Overview

Natron is a free, open-source node-based compositing application comparable to Nuke. In the UE5 pipeline, it is used for post-production compositing of rendered sequences from Movie Render Queue, VFX compositing, color grading, and creating final deliverables.

## Key Features

- **Node-based compositing** — non-destructive pipeline using a graph of processing nodes.
- **OpenFX plugin support** — compatible with a wide range of industry VFX plugins.
- **OpenEXR / multi-layer EXR** — native support for multi-pass EXR renders from UE5.
- **Color management** — OCIO (OpenColorIO) support for accurate color pipelines.
- **Rotoscoping & tracking** — built-in roto shapes and planar tracker.
- **GPU acceleration** — GLSL-accelerated compositing for many operations.
- **Free and open source** — [natrongithub.github.io](https://natrongithub.github.io)

## Interface Overview

- **Node Graph** — main compositing graph workspace.
- **Properties Panel** — selected node parameters.
- **Viewer** — interactive output preview (supports A/B comparison).
- **Curve Editor** — animate node parameters over time.
- **Dope Sheet** — timeline overview of all animated parameters.
- **Project Settings** — frame rate, resolution, color management.

## Core Node Categories

| Category | Key Nodes |
|----------|-----------|
| **Image** | Read (import), Write (export), Constant, Checker |
| **Transform** | Transform, CornerPin, Roto, Warp |
| **Channel** | Shuffle, Copy, Premult, Unpremult |
| **Color** | Grade, ColorCorrect, Saturation, Hue, Gamma |
| **Merge** | Merge (Over, Plus, Screen, etc.), Keymix |
| **Filter** | Blur, Sharpen, Glow, Soften, Defocus |
| **Keying** | Primatte, Keyer, IBKColour, HueKeyer |
| **3D** | Camera, ScanlineRender, Card3D, Transform3D |
| **Time** | FrameHold, Retime, FrameRange |
| **Draw** | Roto, RotoPaint, Text |

## UE5 Compositing Workflow

### Step 1: Export Multi-Pass EXR from UE5

In Movie Render Queue (UE5):
1. Add output format: **EXR Sequence (Multi-layer)**.
2. Enable render passes:
   - **Beauty** (final color)
   - **Base Color** (albedo)
   - **Diffuse** (lighting)
   - **Specular**
   - **Shadow** / **AO**
   - **Depth** (for depth-of-field compositing)
   - **Motion Vectors** (for motion blur in post)
3. Render the sequence.

### Step 2: Import into Natron

1. Add a **Read** node.
2. Navigate to the EXR sequence folder.
3. Select the first frame — Natron detects the sequence automatically.
4. Set the frame range in Project Settings to match.

### Step 3: Extract EXR Layers

Multi-layer EXRs contain multiple passes in one file:

1. Add a **Shuffle** node after the Read node.
2. In Shuffle's properties, select the layer (e.g., `Diffuse.R`, `Specular.G`).
3. Route each layer to the appropriate compositing branch.

### Step 4: Basic Beauty Comp

```
[Read EXR] → [Shuffle: Beauty] → [Grade] → [Write]
                                   ↑
                              [Glow] ← [Shuffle: Specular]
```

1. Extract **Beauty** pass → apply color grade.
2. Extract **Specular** pass → apply **Glow** effect → **Merge (Screen)** back over beauty.
3. Add **Depth** pass → **ZDefocus** node for depth-of-field.
4. Output via **Write** node as PNG sequence or ProRes video.

### Step 5: Export Final Comp

1. Add a **Write** node.
2. Set output path and format:
   - **EXR** for intermediate deliverables
   - **PNG sequence** for web-friendly output
   - **MP4/MOV** via FFmpeg integration (if installed)
3. Render: **Render > Render All Writers**.

## Color Management with OCIO

1. **Edit > Project Settings > Color Management**.
2. Set **OCIO Config** path to your production config (e.g., ACES or sRGB).
3. Set Viewer transform to match your display.
4. This ensures colors from UE5's linear EXR renders are viewed and output correctly.

## Rotoscoping

1. Add a **Roto** node.
2. Draw Bezier or B-Spline shapes over the footage in the Viewer.
3. Animate shape keyframes to track movement.
4. Use the output matte as a mask on color correction or effect nodes.

## Tracking

1. Add a **Tracker** node.
2. Select tracking patterns in the Viewer.
3. Click **Track Forward** — Natron tracks the pattern through frames.
4. Export tracking data to a **Transform** or **CornerPin** node for stabilization/matchmove.

## Python Scripting

Natron has a built-in Python 2 scripting console:

```python
import NatronEngine

app = NatronEngine.natron.getActiveInstance()

# Get a node
read_node = app.getNode("Read1")
print(read_node.getParam("filename").getValue())

# Set a parameter
grade_node = app.getNode("Grade1")
grade_node.getParam("gain").setValue(1.2, 0)  # 0 = R channel
```

Access via **Script > Script Editor** in the Natron menu.

## AI / MCP Integration

Natron does not currently have an official MCP server. However, its Python scripting API and CLI rendering mode enable pipeline automation:

```bash
# Headless render via CLI
Natron --background --writer Write1 myproject.ntp
```

An AI agent can:
- Generate Natron project files (.ntp) programmatically
- Trigger CLI renders
- Parse and validate render outputs

See [docs/mcp-servers/overview.md](../mcp-servers/overview.md) for current MCP status.

## References

- [Natron Documentation](https://natron.readthedocs.io)
- [Natron GitHub](https://github.com/NatronGitHub/Natron)
- [OpenColorIO](https://opencolorio.org)
- [UE5 Movie Render Queue](https://docs.unrealengine.com/5.0/en-US/movie-render-pipeline-overview-in-unreal-engine/)
