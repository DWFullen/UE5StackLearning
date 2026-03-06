# Unreal Editor

## Overview

Unreal Editor is the primary development environment for Unreal Engine 5. It combines a real-time 3D viewport, asset management, level design tools, scripting, and build/packaging all in one application.

## Key Concepts

### Project Structure

```
MyProject/
├── Content/          # All assets (meshes, textures, materials, blueprints)
├── Source/           # C++ source files
├── Config/           # Project configuration (.ini files)
├── Saved/            # Auto-saves, logs, intermediate artifacts
├── Plugins/          # Project-scoped plugins
└── MyProject.uproject
```

### Editor Layout

- **Viewport** — Real-time 3D scene view; supports Perspective, Top, Front, and Side projections.
- **Content Browser** — Asset explorer and importer. Supports drag-and-drop into the viewport.
- **Details Panel** — Property inspector for selected actors and assets.
- **Outliner** — Scene hierarchy listing all actors in the level.
- **Toolbar** — Play-in-Editor (PIE), build, source control, and platform launchers.

### Modes

| Mode | Shortcut | Purpose |
|------|----------|---------|
| Selection | `W/E/R` | Translate, Rotate, Scale actors |
| Landscape | `Shift+2` | Terrain sculpting and painting |
| Foliage | `Shift+3` | Scatter foliage instances |
| Modeling | `Shift+4` | In-editor mesh editing |
| Fracture | `Shift+5` | Geometry Collection (chaos destruction) |

## Common Workflows

### Importing Assets

1. Drag asset files (FBX, OBJ, PNG, etc.) into the Content Browser or use **Import** button.
2. Configure the import options dialog (scale, skeleton assignment, material creation).
3. Accept to create UAssets in the target folder.

### Placing Actors

- Drag from Content Browser into Viewport.
- Use **Place Actors** panel (`Shift+1`) for primitives, lights, volumes, etc.

### Level Streaming / World Partition

- UE5 uses **World Partition** by default for open-world levels.
- Cells are loaded/unloaded based on the player's HLOD distance.
- Use **World Partition Editor** (Window > World Partition) to visualize and configure cells.

### Source Control Integration

- Supports Perforce (P4V), Git (via plugin), and Plastic SCM.
- Enable under **Edit > Project Settings > Source Control**.
- Unreal Game Sync (UGS) is the recommended client for P4 team workflows.

### Packaging / Building

1. **File > Package Project > [Platform]**
2. Configure in **Project Settings > Packaging**.
3. For iterative builds, use **File > Cook Content for [Platform]** to pre-cook assets.

## Performance & Profiling

- **Stat commands** — Type in viewport console (e.g., `stat fps`, `stat unit`, `stat gpu`).
- **GPU Profiler** — `ProfileGPU` command or **Tools > GPU Visualizer**.
- **Unreal Insights** — Standalone profiling tool (`UnrealInsights.exe`), records traces for CPU, GPU, memory, and network.
- **Nanite** — Enable per-mesh for virtualized geometry; view coverage with `NaniteVisualize` viewport mode.
- **Lumen** — Global illumination and reflections system; configure quality in Project Settings > Rendering.

## Useful Console Commands

| Command | Effect |
|---------|--------|
| `r.SetRes 1920x1080` | Set viewport resolution |
| `t.MaxFPS 60` | Cap frame rate |
| `ShowFlag.PostProcessing 0` | Toggle post-processing |
| `stat SceneRendering` | Rendering statistics |
| `dumpticks` | List all ticking objects |
| `obj list class=StaticMeshActor` | List actors of a type |

## Editor Scripting

### Python (Editor Utility Scripts)

```python
import unreal

# Get all static mesh actors in the current level
actors = unreal.EditorLevelLibrary.get_all_level_actors()
for actor in actors:
    if isinstance(actor, unreal.StaticMeshActor):
        print(actor.get_name())
```

### Editor Utility Widgets (Blueprint)

Create **Editor Utility Widget** blueprints to build custom in-editor tools with a UMG UI.

## AI / MCP Integration

See [docs/mcp-servers/overview.md](../mcp-servers/overview.md) for MCP servers that can drive the Unreal Editor programmatically from an AI agent.

## References

- [Unreal Engine Documentation](https://docs.unrealengine.com)
- [UE5 Quick Start Guide](https://docs.unrealengine.com/5.0/en-US/unreal-engine-5-0-quick-start/)
- [Unreal Online Learning](https://dev.epicgames.com/community/learning)
