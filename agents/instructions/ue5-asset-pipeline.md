# Agent Instructions: UE5 Asset Pipeline

Copy this file into any project requiring asset creation and import into UE5 as `AGENT.md` or `.cursorrules`.

---

## Asset Pipeline Overview

```
Concept Art (Krita)
        │
        ▼
3D Modeling / Rigging (Blender)
        │
Texture Authoring (GIMP / Krita / Material Maker)
        │
        ▼
FBX / PNG Export
        │
        ▼
UE5 Import (Content Browser)
        │
        ▼
Material Setup → Level Placement → Testing
        │
        ▼
Rendering (Movie Render Queue → Natron compositing)
```

---

## Blender Export Checklist

Before exporting from Blender:

- [ ] Apply all modifiers (`Object > Apply > All Modifiers`)
- [ ] Apply transforms (`Object > Apply > All Transforms`)
- [ ] Triangulate mesh (Modifier > Triangulate, or export with `Triangulate Faces` option)
- [ ] UV unwrapped; UV0 for material, UV1 for lightmaps (or let UE5 auto-generate UV1)
- [ ] Normals correct (no inverted faces; check with `Viewport Overlay > Face Orientation`)
- [ ] Pivot at logical origin (bottom-center for upright objects)
- [ ] Bone deform only (no control/IK bones in skeletal mesh export)
- [ ] Scale applied (or using `FBX_SCALE_ALL` export option)
- [ ] Axis: -Y Forward, Z Up

---

## Texture Preparation Checklist

Before importing textures into UE5:

- [ ] Power-of-two dimensions (512, 1024, 2048, 4096)
- [ ] Color space correct (see table below)
- [ ] File format: PNG for most maps; EXR for HDR/32-bit
- [ ] Normal maps: DirectX-style (Y-axis NOT flipped; +Y = up in DirectX space)

| Map | Color Space | Bit Depth |
|-----|-------------|-----------|
| Base Color / Albedo | sRGB | 8-bit |
| Normal Map | Linear | 8-bit |
| Roughness | Linear | 8-bit |
| Metallic | Linear | 8-bit |
| Occlusion | Linear | 8-bit |
| ORM (Packed) | Linear | 8-bit |
| Emissive | Linear | 8-bit or EXR |
| Height / Displacement | Linear | 16-bit |
| HDR Sky / Environment | Linear | 32-bit EXR |

---

## UE5 Import Settings

### Static Mesh FBX Import

```
Import Mesh: ✅
Import as Skeletal: ❌
Generate Lightmap UVs: ✅ (unless pre-baked UV1 in Blender)
Auto Generate Collision: ✅ (for simple geometry) or ❌ (add custom UCX)
Combine Meshes: depends on use case
Transform:
  Import Translation: 0, 0, 0
  Import Rotation: 0, 0, 0
  Import Uniform Scale: 1.0
```

### Texture Import Settings

After import, verify in the Texture Asset:
- **Compression Settings**: BC7 (default for color), NormalMap (for normals), Masks (for greyscale data)
- **sRGB**: ✅ for Albedo; ❌ for all data maps
- **LOD Bias**: 0 for production
- **Power of Two Mode**: None (if already POT)

---

## UE5 Material Setup

### PBR Master Material Inputs

```
Base Color     ← T_*_BaseColor (sRGB texture)
Normal         ← T_*_Normal (normal map, flipped G channel if needed)
Roughness      ← ORM.G channel (Mask R component)
Metallic       ← ORM.B channel (Mask B component)
Ambient Occ.   ← ORM.R channel (Mask G component)
Emissive       ← T_*_Emissive × EmissiveStrength (scalar parameter)
```

### Material Instance Parameters

Expose these parameters in the master material for per-instance customization:
- `BaseColor` (color parameter)
- `EmissiveStrength` (scalar, 0–10)
- `RoughnessScale` (scalar, 0–1)
- `NormalStrength` (scalar, 0–2)
- `TilingScale` (scalar, default 1)

---

## Python Import Automation

```python
import unreal

def import_asset(source_path: str, destination_folder: str, as_skeletal: bool = False):
    """Import an FBX or PNG asset into UE5."""
    task = unreal.AssetImportTask()
    task.filename = source_path
    task.destination_path = destination_folder
    task.automated = True
    task.save = True
    task.replace_existing = True
    
    if source_path.endswith('.fbx'):
        options = unreal.FbxImportUI()
        options.import_mesh = True
        options.import_as_skeletal = as_skeletal
        options.import_animations = as_skeletal
        options.auto_generate_collision = True
        task.options = options
    
    unreal.AssetToolsHelpers.get_asset_tools().import_asset_tasks([task])
    print(f"Imported: {source_path} → {destination_folder}")

# Example usage
import_asset("C:/Assets/SM_Rock.fbx", "/Game/Environment/")
import_asset("C:/Textures/T_Rock_BaseColor.png", "/Game/Environment/Textures/")
```

---

## Naming Conventions

| Asset | Prefix | Example |
|-------|--------|---------|
| Static Mesh | `SM_` | `SM_Rock_01` |
| Skeletal Mesh | `SK_` | `SK_Character` |
| Texture | `T_` | `T_Rock_BaseColor` |
| Material | `M_` | `M_Rock_PBR` |
| Material Instance | `MI_` | `MI_Rock_Mossy` |
| Niagara System | `NS_` | `NS_Dust` |
| Blueprint | `BP_` | `BP_Door` |

---

## MCP Tools for Asset Pipeline

| Step | MCP Tool | Action |
|------|----------|--------|
| Modeling | blender-mcp | Create/modify mesh |
| Texturing | blender-mcp | Apply and render material previews |
| Export | blender-mcp | `execute_blender_code` → FBX export |
| UE5 Import | UE5 Python Remote | `import_asset_tasks` |
| UE5 Material | UE5 Python Remote | `MaterialEditingLibrary` |
| UE5 Placement | UE5 Python Remote | `spawn_actor_from_object` |

---

## References

- [UE5 FBX Import](https://docs.unrealengine.com/5.0/en-US/fbx-import-pipeline-in-unreal-engine/)
- [UE5 Texture Import](https://docs.unrealengine.com/5.0/en-US/textures-in-unreal-engine/)
- [UE5 Material Editor](https://docs.unrealengine.com/5.0/en-US/unreal-engine-material-editor-user-guide/)
- [Blender Export Guide](../../docs/tools/blender.md)
