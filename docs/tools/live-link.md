# Live Link

## Overview

Live Link is Unreal Engine 5's real-time data streaming framework. It allows external data sources — motion capture systems, animation rigs, cameras, lighting consoles, face trackers, and more — to stream data into the Unreal Editor or a running game in real time without rebaking or reimporting assets.

## Key Concepts

### Architecture

```
External Source (Mocap, Maya, Shogun, etc.)
        │
        ▼
Live Link Source Plugin  ──►  Live Link Hub (optional broker)
        │
        ▼
Live Link Subject (named data stream)
        │
        ▼
Live Link Preset applied to Actor in UE5
```

### Core Types

| Type | Description |
|------|-------------|
| **Live Link Source** | Plugin that connects to a specific data provider (e.g., Vicon, OptiTrack, Maya, ARKit) |
| **Live Link Subject** | A named stream of data (e.g., "Spine", "Camera01") |
| **Live Link Role** | Defines the data schema (Animation, Camera, Light, Transform, etc.) |
| **Live Link Preset** | Asset that saves and restores a set of sources/subjects |

### Built-in Roles

| Role | Data Type |
|------|-----------|
| Animation | Full skeletal pose (bones + blend shapes) |
| Transform | World or local transform |
| Camera | Lens, aperture, focus, field of view |
| Light | Color, intensity, temperature |
| Basic | Generic timestamped float/int data |

## Opening Live Link

1. **Window > Live Link** to open the Live Link panel.
2. Click **+ Source** to add a data source.
3. Subjects appear in the list once data is streaming.

## Common Sources (Plugins)

| Source | Plugin |
|--------|--------|
| Maya | Live Link plugin included with Maya 2022+ |
| MotionBuilder | Live Link plugin for MotionBuilder |
| Vicon | Vicon Shogun Live — streams to Live Link |
| OptiTrack | Motive software → Live Link plugin |
| ARKit (iOS Face) | ARKit Face plugin; uses iPhone via Unreal Remote 2 |
| OSC | Live Link OSC plugin — receive OSC data |
| VRPN | VRPN plugin for lab/research hardware |
| ZED (Stereolabs) | AI body tracking → Live Link |
| Live Link Hub | Standalone broker app for routing multiple sources |

## Setting Up a Maya → UE5 Live Link Connection

1. In Maya: **Windows > General Editors > Live Link**.
2. Add a connection with your UE5 machine IP and default port `54321`.
3. Add subjects (joints/cameras/lights) to the Maya Live Link window.
4. In UE5: **Window > Live Link**, add source **Message Bus Source** — subjects from Maya appear.
5. On a Skeletal Mesh Actor in UE5, add the **Live Link Controller** component.
6. In the component, set the **Subject Name** to match the Maya subject.

## Using Live Link in Animation Blueprint

1. In an **Animation Blueprint**, add a **Live Link Pose** node.
2. Set **Subject Name** and the **Role** (Animation).
3. Connect to the **Output Pose** node.
4. The character mirrors the incoming Live Link skeleton in real time.

```
Live Link Pose ──► Component To Local ──► Output Pose
```

## Using Live Link with Camera

1. Place a **Cine Camera Actor** in the level.
2. Add a **Live Link Controller** component.
3. Set Subject Name and Role to **Camera**.
4. The camera now tracks the incoming camera data (position, rotation, lens).
5. Use with **Virtual Production** workflows and ICVFX walls.

## Live Link XR (VR Controllers)

- Enable **Live Link XR** plugin.
- Streams HMD, left controller, and right controller transforms.
- Subjects appear automatically: `LiveLinkXR_HMD`, `LiveLinkXR_LeftHand`, etc.

## Blueprinting with Live Link

```cpp
// C++ — get current subject frame data
#include "LiveLinkClientReference.h"
#include "Roles/LiveLinkAnimationRole.h"

FLiveLinkClientReference ClientRef;
ILiveLinkClient* Client = ClientRef.GetClient();
FLiveLinkSubjectKey SubjectKey(FLiveLinkSource(), FName("MySubject"));

FLiveLinkSubjectFrameData FrameData;
if (Client->EvaluateFrame_AnyThread(SubjectKey, ULiveLinkAnimationRole::StaticClass(), FrameData))
{
    FLiveLinkAnimationFrameData* AnimData = FrameData.FrameData.Cast<FLiveLinkAnimationFrameData>();
    // AnimData->Transforms contains bone transforms
}
```

## Live Link Preset

- Save your current source/subject configuration as a **Live Link Preset** asset.
- Load it on startup to auto-connect: **Project Settings > Live Link > Default Live Link Preset**.
- Useful for production environments where the same rig is used daily.

## Virtual Production & ICVFX

Live Link is central to Unreal Engine's **Virtual Production** pipeline:

- **Live Link VCAM** (iOS app) — stream iPhone camera data to UE5.
- **nDisplay** — Live Link data drives the in-camera VFX LED volume rendering.
- **Live Link Face** (iOS app) — stream 52-blendshape ARKit face data for digital doubles.

## Performance Considerations

- Use **Interpolation Mode** per subject:
  - **None** — latest frame only (lowest latency).
  - **Interpolated** — smooth playback between frames (adds latency).
  - **Extrapolated** — predict ahead to reduce perceived latency.
- Set **Buffer Size** (source frames buffered) based on network latency.
- Live Link Hub can act as a broker to reduce direct connections.

## AI / MCP Integration

An AI agent can drive Live Link subjects programmatically, enabling AI-controlled character animation and camera movements in real time. See [docs/mcp-servers/overview.md](../mcp-servers/overview.md).

## References

- [Live Link Overview](https://docs.unrealengine.com/5.0/en-US/live-link-in-unreal-engine/)
- [Live Link Plugin Development](https://docs.unrealengine.com/5.0/en-US/creating-a-live-link-source-in-unreal-engine/)
- [Virtual Production](https://docs.unrealengine.com/5.0/en-US/virtual-production-overview-in-unreal-engine/)
