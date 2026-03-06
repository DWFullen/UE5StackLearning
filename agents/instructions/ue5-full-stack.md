# Agent Instructions: UE5 Full Stack

This file provides instructions for AI agents working on Unreal Engine 5 projects that use the full creative stack. Copy this file to a project's root as `AGENT.md` or `.cursorrules` to give agents context.

---

## Project Stack

This project uses the following tools:

- **Unreal Engine 5** — game/visualization engine (primary)
- **Blender** — 3D modeling, rigging, and animation
- **GIMP** — texture editing and raster image work
- **Krita** — concept art and hand-painted textures
- **Material Maker** — procedural PBR texture authoring
- **Natron** — compositing and post-production
- **OBS Studio** — recording and streaming
- **C++** — engine and gameplay code
- **Blueprint Visual Scripting** — designer-friendly game logic

---

## Project Structure Conventions

```
Content/
├── Characters/         # Skeletal meshes, animations, ABPs
├── Environment/        # Static meshes, materials, landscapes
├── FX/                 # Niagara systems, Materials for FX
├── Levels/             # Map files
├── UI/                 # Widget Blueprints, fonts, icons
├── Blueprints/         # Gameplay Blueprints
└── Developers/         # Per-developer scratch space (not shipped)

Source/
├── [ProjectName]/
│   ├── Public/         # Header files (.h)
│   └── Private/        # Source files (.cpp)
```

---

## C++ Coding Rules

1. **All gameplay classes** must use `UCLASS()`, `UPROPERTY()`, `UFUNCTION()` macros.
2. **Naming**: Classes prefixed by type letter (`A` = Actor, `U` = UObject, `F` = struct, `E` = enum, `I` = interface).
3. **Memory**: Never hold raw `UObject*` pointers without `UPROPERTY()` — they will be garbage collected.
4. **Logging**: Use `UE_LOG(LogTemp, ...)` during development; create a custom log category for production.
5. **Performance**: Avoid per-frame allocations; cache component pointers in `BeginPlay`.
6. **Hot Reload**: Prefer `UFUNCTION` changes via Live Coding (`Ctrl+Alt+F11`); structural changes require full rebuild.

```cpp
// ✅ Correct actor pattern
UCLASS()
class MYGAME_API AMyActor : public AActor
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Config")
    float Speed = 300.f;
    
    UFUNCTION(BlueprintCallable)
    void DoAction();
};
```

---

## Blueprint Rules

1. Keep Blueprint graphs **readable** — use comment boxes and reroute nodes.
2. **Don't** put performance-heavy logic in Event Tick without profiling.
3. **Do** use Blueprint Interfaces for decoupled actor communication.
4. **Do** implement complex systems in C++ and expose to Blueprint with `BlueprintCallable`.
5. **Avoid** casting chains — prefer interfaces or event dispatchers.

---

## Asset Naming Conventions

| Asset Type | Prefix | Example |
|------------|--------|---------|
| Static Mesh | `SM_` | `SM_Rock_01` |
| Skeletal Mesh | `SK_` | `SK_Character_Hero` |
| Material | `M_` | `M_Rock_Base` |
| Material Instance | `MI_` | `MI_Rock_Mossy` |
| Material Function | `MF_` | `MF_Triplanar` |
| Texture | `T_` | `T_Rock_BaseColor` |
| Blueprint Class | `BP_` | `BP_Door` |
| Niagara System | `NS_` | `NS_Explosion` |
| Animation Sequence | `AS_` | `AS_Walk` |
| Animation Blueprint | `ABP_` | `ABP_Character` |
| Widget Blueprint | `WBP_` | `WBP_HUD` |

---

## Blender ↔ UE5 Workflow

1. Model and rig in Blender.
2. Set export axis: **-Y Forward, Z Up**.
3. Apply `FBX_SCALE_ALL` or set scene units to match UE5 centimeter scale.
4. Export via **Send to Unreal** add-on (preferred) or File > Export > FBX.
5. In UE5, import with correct skeleton assignment and LOD settings.

---

## Texture Rules

| Map | Color Space | Format | Size |
|-----|-------------|--------|------|
| Albedo/Base Color | sRGB | PNG 8-bit | 512–4096 |
| Normal Map | Linear | PNG 8-bit | 512–4096 |
| ORM (Occlusion/Roughness/Metallic) | Linear | PNG 8-bit | 512–4096 |
| Emissive | Linear | PNG 8-bit or EXR | 512–2048 |
| Heightmap (Landscape) | Linear | PNG 16-bit | 1009 / 2017 / 4033 |

---

## MCP Tools Available

When an MCP server is configured, agents can use:

- **blender-mcp** — model, texture, and export 3D assets in Blender.
- **UE5 Python Remote Control** — import assets, place actors, modify levels via HTTP.
- **obs-mcp** — start/stop recording, switch scenes in OBS Studio.
- **filesystem** — read/write project files.

---

## Common Agent Tasks

### Import a Mesh into UE5

```python
import unreal
task = unreal.AssetImportTask()
task.filename = "/path/to/mesh.fbx"
task.destination_path = "/Game/Environment/"
task.automated = True
task.save = True
unreal.AssetToolsHelpers.get_asset_tools().import_asset_tasks([task])
```

### Spawn an Actor

```python
import unreal
mesh = unreal.load_asset('/Game/Environment/SM_Rock_01.SM_Rock_01')
actor = unreal.EditorLevelLibrary.spawn_actor_from_object(
    mesh, unreal.Vector(0, 0, 0), unreal.Rotator(0, 0, 0)
)
```

### Create a Material Instance

```python
import unreal
ami = unreal.AssetToolsHelpers.get_asset_tools()
mi = ami.create_asset("MI_NewInstance", "/Game/Materials/",
    unreal.MaterialInstanceConstant, unreal.MaterialInstanceConstantFactoryNew())
```

---

## References

- [Full Stack Docs](https://github.com/DWFullen/UE5StackLearning)
- [UE5 C++ Reference](docs/languages/cpp.md)
- [Blueprint Reference](docs/languages/blueprint.md)
- [MCP Servers](docs/mcp-servers/overview.md)
