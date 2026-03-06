# Sequencer

## Overview

Sequencer is Unreal Engine 5's non-linear cinematic and animation editor. It allows you to create cutscenes, gameplay cinematics, and complex animated sequences by keyframing actors, cameras, animations, audio, and events on a timeline.

## Key Concepts

### Sequence Types

| Type | Use Case |
|------|----------|
| **Level Sequence** | Cinematic cutscenes, in-level animations |
| **Master Sequence** | Container for multiple child sequences (shots) |
| **Shot** | A single camera cut within a Master Sequence |
| **Subsequence** | Reusable sequence nested inside another |

### Tracks

| Track | Purpose |
|-------|---------|
| Camera Cut | Switches between cameras; required for rendering |
| Cine Camera Actor | Camera with filmback/lens settings |
| Skeletal Mesh Animation | Plays AnimSequences on characters |
| Transform | Keyframe position/rotation/scale |
| Visibility | Show/hide actors |
| Audio | Play sound waves/cues |
| Event | Fire Blueprint or C++ events at specific frames |
| Material Parameter | Animate material scalar/vector parameters |
| Fade | Screen fade in/out |
| Level Visibility | Control level streaming |

## Opening Sequencer

1. **Window > Cinematics > Sequencer** to open the panel.
2. Create a new Level Sequence: **Add New > Animation > Level Sequence** in the Content Browser.
3. Double-click a Level Sequence asset to open it in Sequencer.

## Common Workflows

### Creating a Cinematic

1. Open or create a Level Sequence.
2. Add a **Camera Cut** track and a **Cine Camera Actor**.
3. Set the filmback (sensor size) and focal length on the camera.
4. Keyframe the camera transform at different points in time.
5. Add character actors with **Skeletal Mesh Animation** tracks.
6. Use **Render Movie** to export.

### Keyframing

- Press `S` to key all transforms on the selected actor.
- Press `K` with a property focused to key that specific property.
- Right-click a property in the Details Panel and choose **Add Key to Sequencer**.
- Tangent types: Auto, User, Break, Linear, Constant — right-click a key to change.

### Camera Rigs

- **Camera Rig Crane** — Simulates a camera crane/dolly.
- **Camera Rig Rail** — Moves the camera along a spline path.
- Attach a Cine Camera Actor to these rigs via the **Attach** track.

### Rendering with Movie Render Queue (MRQ)

1. Open **Movie Render Queue**: Window > Cinematics > Movie Render Queue.
2. Add your Level Sequence.
3. Configure output settings:
   - **Format**: EXR (for compositing), PNG, JPEG, or video (requires plugin).
   - **Anti-Aliasing**: Increase samples for high-quality renders.
   - **Temporal Sample Count**: Use with `r.MotionBlurSampleCount` for cinematic motion blur.
4. Click **Render (Local)** or **Render (Remote)** for farm rendering.

### Working with Shots (Master Sequence)

1. Create a **Master Sequence** asset.
2. Inside, add a **Shot** track.
3. Right-click the Shot track and **Add New Shot** — this auto-creates a sub-sequence.
4. Each shot can have its own camera, characters, and animation.
5. Use **Take Recorder** to record live performances into shots.

## Take Recorder

- **Window > Cinematics > Take Recorder**.
- Add sources: Player, Actor, Audio, Level Sequence.
- Press **Record** to capture gameplay or real-time performance.
- Generates a Level Sequence with all recorded tracks.

## Sequencer + Blueprint Events

- Add an **Event Track** to a sequence.
- Create **Event Endpoints** — these fire Blueprint events on bound actors.
- Useful for triggering gameplay logic at precise cinematic moments.

## Python Scripting

```python
import unreal

# Load a level sequence asset
seq = unreal.load_asset('/Game/Cinematics/MySequence.MySequence')
editor = unreal.LevelSequenceEditorBlueprintLibrary

# Get all tracks
tracks = seq.get_master_tracks()
for track in tracks:
    print(track.get_display_name())
```

## Performance Tips

- Disable **World Partition** streaming during cinematic playback.
- Use **Pre-roll** frames to allow physics/cloth simulation to settle before recording.
- Reduce playback resolution during editing; set **Render** quality for final output.
- Enable **Evaluate sub-sequences in isolation** when working on individual shots.

## AI / MCP Integration

Sequencer can be driven via Python scripting. An AI agent can automate keyframe placement, shot management, and rendering pipelines. See [docs/mcp-servers/overview.md](../mcp-servers/overview.md).

## References

- [Sequencer Overview](https://docs.unrealengine.com/5.0/en-US/sequencer-cinematic-editor-overview/)
- [Movie Render Queue](https://docs.unrealengine.com/5.0/en-US/movie-render-pipeline-overview-in-unreal-engine/)
- [Take Recorder](https://docs.unrealengine.com/5.0/en-US/take-recorder-in-unreal-engine/)
