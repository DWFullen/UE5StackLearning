# Modeling Mode

## Overview

Modeling Mode is Unreal Engine 5's built-in mesh editing environment. It allows artists and developers to create, modify, and UV-unwrap static meshes directly inside the Unreal Editor without needing to round-trip through an external DCC application like Blender or Maya.

## Accessing Modeling Mode

- Press **Shift+4** or click the **Modeling** icon in the mode toolbar.
- Alternatively: **Window > Modes > Modeling**.
- Requires the **Modeling Tools Editor Mode** plugin (enabled by default in UE5).

## Tool Categories

### Create

| Tool | Description |
|------|-------------|
| Box | Generate a box primitive |
| Sphere | Generate a UV sphere |
| Cylinder | Generate a cylinder |
| Cone | Generate a cone |
| Torus | Generate a torus |
| Arrow, Rectangle, Disc | Additional primitives |
| PolyModel | Free-form polygon modeling (push/pull faces) |
| CubeGrid | Block-out modeling on a voxel grid |
| Path Extrude | Extrude a shape along a spline path |
| Revolve | Revolve a profile curve |

### PolyModel (Main Mesh Editing)

| Tool | Description |
|------|-------------|
| PolyEd | Select and edit polygons, edges, and vertices |
| TriSel | Triangle-level selection and editing |
| Extrude | Push/pull selected polygons |
| Inset | Inset selected polygons |
| Cut Face | Draw cut lines across faces |
| Loop Cut | Insert edge loops |
| Weld | Merge nearby vertices |
| Simplify | Reduce polygon count |
| Boolean | CSG boolean operations (Union, Subtract, Intersect) |

### Transform

| Tool | Description |
|------|-------------|
| Transform | Precise move/rotate/scale |
| Align | Align meshes to surfaces or each other |
| Pivot | Adjust the mesh pivot point |

### Deform

| Tool | Description |
|------|-------------|
| Sculpt | Brush-based sculpting (raise, smooth, pinch, etc.) |
| Lattice | Free-form deformation cage |
| Bend | Bend a mesh along an axis |
| Twist | Twist a mesh along an axis |
| Displace | Heightmap-driven vertex displacement |

### UVs

| Tool | Description |
|------|-------------|
| AutoUV | Automatic UV atlas generation |
| UV Editor | Full UV layout editor (Window > UV Editor) |
| Project | Project UVs from a plane, cylinder, or box |
| Unwrap | Unfold seam-based UV unwrapping |
| Layout | Pack UV islands within 0-1 space |
| Seams | Add/remove seams for unwrapping |

### Attributes & LODs

| Tool | Description |
|------|-------------|
| Normals | Recalculate, smooth, or harden normals |
| Tangents | Recompute mesh tangents |
| Bake | Bake textures (normals, AO, curvature) to a lower-poly mesh |
| LOD Generation | Auto-generate LOD levels |
| LOD Manager | Manage and preview LOD levels |
| Collision | Generate collision hulls (Simple, Complex, Convex, UCX) |

## Common Workflows

### Block-out / Greybox

1. Switch to Modeling Mode.
2. Use **CubeGrid** to rough-block level geometry quickly on a grid.
3. Use **PolyEd** to refine individual shapes.
4. Move finished pieces to the Content Browser as Static Mesh assets.

### Mesh Boolean Operations

1. Place two overlapping meshes in the level.
2. Select both, choose **Boolean** tool.
3. Pick operation: Union / Intersect / Subtract.
4. Result is a new Static Mesh saved to the Content Browser.

### UV Unwrapping

1. Select a mesh, activate **UV Editor** (Window > UV Editor).
2. In the UV Editor, use **Seams** tool to mark seam edges.
3. Apply **Unwrap** to unfold the UVs along seams.
4. Use **Layout** to pack islands and maximize texel density.

### Baking Textures

1. Create a high-poly and low-poly version of the same mesh.
2. Select both, open **Bake** tool.
3. Set target mesh (low-poly), source mesh (high-poly).
4. Choose output maps: Normal, AO, Curvature, etc.
5. Click **Bake** — outputs texture assets to the Content Browser.

## Performance Tips

- Modeling Mode edits create **new Static Mesh assets** — original source assets are not modified.
- For complex sculpting workflows, prefer Blender or ZBrush then import.
- Use **LOD Generation** to automatically create performance LODs after modeling.
- Keep poly counts reasonable; use Nanite for high-fidelity meshes if UE5.

## AI / MCP Integration

Modeling Mode operations can be scripted via **Unreal Python** and the **GeometryScripting** plugin, which exposes mesh manipulation functions (booleans, UVs, normals) to Python and Blueprint. An AI agent can use these APIs to procedurally generate or modify geometry.

```python
import unreal

# Example: Generate a box mesh via Geometry Script
# (Requires GeometryScript plugin)
mesh = unreal.DynamicMesh()
options = unreal.GeometryScriptPrimitiveOptions()
unreal.GeometryScriptLibrary_MeshPrimitiveFunctions.append_box(
    mesh, options, unreal.Transform(), 100.0, 100.0, 100.0, 2, 2, 2,
    unreal.GeometryScriptPrimitiveOriginMode.BASE
)
```

## References

- [Modeling Mode Overview](https://docs.unrealengine.com/5.0/en-US/modeling-mode-in-unreal-engine/)
- [UV Editor](https://docs.unrealengine.com/5.0/en-US/uv-editor-in-unreal-engine/)
- [Geometry Scripting](https://docs.unrealengine.com/5.0/en-US/geometry-scripting-overview-in-unreal-engine/)
