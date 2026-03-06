# Blender

## Overview

Blender is a free, open-source 3D creation suite used extensively in UE5 pipelines for modeling, rigging, animation, simulation, and rendering. It is the primary external DCC (Digital Content Creation) tool for most independent UE5 developers.

## Key Workspaces

| Workspace | Purpose |
|-----------|---------|
| **Layout** | General object placement and viewport navigation |
| **Modeling** | Mesh editing (Edit Mode) |
| **Sculpting** | Brush-based high-poly sculpting |
| **UV Editing** | UV unwrap and layout |
| **Texture Paint** | Paint directly on mesh surfaces |
| **Shading** | Node-based material editor |
| **Animation** | Keyframe animation, NLA, graph editor |
| **Rendering** | Cycles/EEVEE render output |
| **Compositing** | Node-based post-process compositing |
| **Geometry Nodes** | Procedural geometry / VFX |

## UE5 Export Workflow

### FBX Export (Recommended for UE5)

1. Select the mesh(es) to export.
2. **File > Export > FBX (.fbx)**.
3. Settings:
   - **Scale**: 1.0 (UE5 expects centimeter-scale; set manually if needed).
   - **Apply Scalings**: FBX Units Scale.
   - **Forward**: -Y Forward, Z Up (matches UE5 coordinate system).
   - **Include**: Selected Objects, Apply Modifiers.
   - For **Skeletal Meshes**: Enable **Armature** and **Only Deform Bones**.
   - For **Animations**: Enable **Bake Animation**, set frame range.

### Scale Consideration

Blender uses meters; UE5 uses centimeters.
- Option 1: In Blender, set scene units to **None** and treat 1 unit = 1 cm (scale model to UE5 scale).
- Option 2: Export with `100x` scale factor (set in FBX export dialog under **Scale**).

### glTF / glb Export (Alternative)

UE5 supports glTF import (via plugin). Use for web-ready assets or when FBX has issues.

```
File > Export > glTF 2.0 (.glb/.gltf)
```

## Rigging for UE5

- Blender rigs export well to UE5 when using **Rigify** or custom armatures.
- Export with **Only Deform Bones** to exclude control bones.
- UE5's humanoid skeleton: Use **UE5 Mannequin Skeleton** retarget if needed.
- Import in UE5 with **Import Mesh + Skeleton**.

### Naming Convention for Bones

UE5 recognizes common bone names for retargeting. Use Epic's standard naming:
```
root, pelvis, spine_01...spine_05
clavicle_l, upperarm_l, lowerarm_l, hand_l
thigh_l, calf_l, foot_l, ball_l
```

## Animation Workflow

1. Animate in Blender NLA Editor or Action Editor.
2. Export FBX with **Bake Animation** enabled.
3. Import into UE5:
   - **Import > FBX** — select **Import as Skeletal Mesh + Animation**.
   - Or import animations separately targeting an existing skeleton.

## Geometry Nodes → UE5

- Geometry Nodes procedural assets can be baked to mesh before export.
- Use **Object > Apply > Make Instances Real** to convert instances.
- Export baked mesh via FBX.

## Blender + UE5 Direct Bridge

### Send to Unreal (Epic's Add-on)

- **Send to Unreal** is a free Blender add-on by Epic Games.
- Automates export, scale correction, and import into UE5.
- [Download](https://github.com/EpicGames/BlenderTools)

```
# Installation
1. Download the add-on from GitHub.
2. Blender: Edit > Preferences > Add-ons > Install.
3. Enable "Pipeline: Send to Unreal".
4. Use the N-panel > "Unreal" tab to send assets directly.
```

### UE to Rigify (Reverse: UE5 → Blender)

- Import UE5 Mannequin FBX into Blender.
- Use **UE to Rigify** add-on to convert to a Rigify control rig.

## Python Scripting in Blender

```python
import bpy

# Create a cube and export to FBX
bpy.ops.mesh.primitive_cube_add(size=2, location=(0, 0, 0))
obj = bpy.context.active_object

# Rename and set scale
obj.name = "MyCube"
obj.scale = (0.01, 0.01, 0.01)  # Convert meters to cm-equivalent

# Export
bpy.ops.export_scene.fbx(
    filepath="/tmp/output/mycube.fbx",
    use_selection=True,
    apply_scale_options='FBX_SCALE_ALL',
    axis_forward='-Y',
    axis_up='Z'
)
```

## MCP Server for Blender

Blender has a community MCP server — **blender-mcp** — that enables AI agents to control Blender via natural language.

- Repository: [ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp)
- See [docs/mcp-servers/blender.md](../mcp-servers/blender.md) for setup.

## Performance Tips for UE5 Assets

- **Poly count**: Target under 100K tris for hero characters; under 10K for environment props.
- **UV channels**: UE5 uses UV0 for materials; UV1 for lightmap UVs (can be auto-generated).
- **Materials**: Merge material slots where possible to reduce draw calls.
- **LODs**: Create LOD0, LOD1, LOD2 in Blender or use UE5's auto-LOD.
- **Normals**: Hard/soft edges baked into a normal map look better than high geo counts.

## References

- [Blender Documentation](https://docs.blender.org)
- [Send to Unreal Add-on](https://github.com/EpicGames/BlenderTools)
- [blender-mcp MCP Server](https://github.com/ahujasid/blender-mcp)
- [UE5 FBX Import](https://docs.unrealengine.com/5.0/en-US/fbx-import-pipeline-in-unreal-engine/)
