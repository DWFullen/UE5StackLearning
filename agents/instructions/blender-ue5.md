# Agent Instructions: Blender for UE5

Copy this file into a Blender-centric project as `AGENT.md` or add its contents to `.cursorrules` to give AI agents context about Blender workflows targeting Unreal Engine 5.

---

## Tool: Blender

**Version**: 4.x (recommended for UE5 workflows)
**Export Target**: Unreal Engine 5
**Coordinate System**: -Y Forward, Z Up (matches UE5)

---

## MCP Server

Use **blender-mcp** for direct Blender control:

```json
{
  "mcpServers": {
    "blender": {
      "command": "uvx",
      "args": ["blender-mcp"]
    }
  }
}
```

Ensure Blender is open with the BlenderMCP add-on server running (N Panel > BlenderMCP > Start).

---

## Modeling Rules

1. **Scale**: Work in Blender units = centimeters (1 unit = 1 cm to match UE5). If using meters, apply a 100x scale on FBX export.
2. **Apply transforms** before export: `Object > Apply > All Transforms`.
3. **Triangulate** faces for game meshes; avoid n-gons.
4. **Normals**: Hard edges for mechanical/angular shapes; smooth shading for organic shapes. Bake normals to a texture for high-detail work.
5. **Pivot** should be at the logical origin of the mesh (bottom center for upright objects, center for symmetric objects).

---

## UV Rules

1. UV islands must be within the **0–1 UV space** (no overlapping unless intentional for tiling).
2. **UV0** = material UVs; **UV1** = lightmap UVs (can be auto-generated in UE5).
3. Maximize texel density; use UV pack to fill UV space efficiently.
4. For tiling textures, UVs can extend beyond 0–1 space.

---

## FBX Export Settings

```python
bpy.ops.export_scene.fbx(
    filepath="output.fbx",
    use_selection=True,
    apply_scale_options='FBX_SCALE_ALL',
    axis_forward='-Y',
    axis_up='Z',
    bake_space_transform=True,
    mesh_smooth_type='FACE',
    use_mesh_modifiers=True,
    add_leaf_bones=False,          # For skeletal meshes
    primary_bone_axis='Y',
    secondary_bone_axis='X'
)
```

---

## Skeletal Mesh Rules

1. Bone rolls must be consistent (use **Recalculate Bone Rolls** in Armature edit mode).
2. Export **only deform bones** (`Only Deform Bones` export option).
3. Vertex groups must match bone names exactly.
4. Root bone at world origin, named `root`.
5. No IK/control bones in the export — deform chain only.

---

## Animation Export

1. Each animation = one **Action** in the NLA Editor.
2. Export with `Bake Animation = True`, set frame range to the action's range.
3. For multiple animations: export each action separately or use NLA strips.
4. Target frame rate: 30fps (match UE5 project FPS).

---

## Python Scripting (bpy)

```python
import bpy

# Create a mesh
bpy.ops.mesh.primitive_cube_add(size=1)
obj = bpy.context.active_object
obj.name = "SM_MyMesh"

# Apply scale
bpy.ops.object.transform_apply(scale=True)

# Export FBX
bpy.ops.export_scene.fbx(
    filepath="/tmp/SM_MyMesh.fbx",
    use_selection=True,
    apply_scale_options='FBX_SCALE_ALL',
    axis_forward='-Y',
    axis_up='Z'
)
```

---

## Common Agent Tasks via blender-mcp

| Task | Tool Call |
|------|-----------|
| Get scene info | `get_scene_info()` |
| Create a cube | `create_object(type="CUBE")` |
| Move an object | `modify_object(name="Cube", location=[0,0,1])` |
| Set material color | `set_material(object="Cube", color=[1,0,0])` |
| Render the scene | `render_scene(output_path="/tmp/render.png")` |
| Export to FBX | `execute_blender_code(code="bpy.ops.export_scene.fbx(...)")` |

---

## References

- [Blender Documentation](https://docs.blender.org)
- [blender-mcp](https://github.com/ahujasid/blender-mcp)
- [Send to Unreal Add-on](https://github.com/EpicGames/BlenderTools)
- [UE5 FBX Import Guide](https://docs.unrealengine.com/5.0/en-US/fbx-import-pipeline-in-unreal-engine/)
