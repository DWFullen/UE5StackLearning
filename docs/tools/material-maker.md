# Material Maker

## Overview

Material Maker is a free, open-source procedural material authoring tool built on the Godot Engine. It uses a node-based graph to create complex PBR (Physically Based Rendering) materials and export texture maps for use in game engines like Unreal Engine 5.

## Key Features

- **Node-based graph** — connect procedural generators, filters, and blending nodes.
- **Real-time preview** — instant 3D preview on a sphere, plane, cube, or custom mesh.
- **PBR output** — generates Albedo, Normal, Roughness, Metallic, Emissive, and Height maps.
- **Tileable textures** — all generators produce seamlessly tiling outputs by default.
- **Export presets** — built-in presets for UE4/UE5, Unity, Godot, and more.
- **Python-like scripting** — custom nodes can be scripted in GDScript (Godot's Python-like language).
- **Free and open source** — [GitHub: RodZill4/material-maker](https://github.com/RodZill4/material-maker)

## Interface Overview

- **Graph Editor** — main canvas for connecting material nodes.
- **Node Library** — searchable palette of all available nodes.
- **2D Preview** — flat preview of the selected output map.
- **3D Preview** — PBR preview on a 3D mesh with lighting.
- **Parameters Panel** — exposes adjustable parameters from selected nodes.
- **Export Dialog** — configure and export texture maps.

## Core Node Types

| Category | Examples |
|----------|---------|
| **Generators** | Noise (Perlin, Voronoi, FBM), Brick, Checker, Dots, Gradient |
| **Filters** | Blur, Sharpen, Emboss, Warp, Level, Colorize |
| **Blending** | Mix, Blend (various modes), Mask, Layer |
| **Transform** | Tile, Rotate, Flip, Offset, Scale |
| **Math** | Add, Multiply, Power, Remap, Clamp |
| **Normal** | Normal Map, Curvature, Slope |
| **Shape** | Circle, Polygon, Star, Text, Bevel |
| **UV** | UV Transform, UV Warp, Tri-planar |

## Workflow: Creating a Material for UE5

### Basic PBR Material

1. Open Material Maker and create a new project.
2. Add a **PBR** output node (the default project includes this).
3. Connect a **Noise** generator to the Albedo input.
4. Add a **Normal Map** node between a height noise and the Normal input.
5. Connect **Roughness** and **Metallic** values (can be constants or gradients).
6. Preview in the 3D viewport.

### Exporting for UE5

1. **File > Export Textures**.
2. Select export preset: **Unreal Engine 4/5**.
3. Set resolution: 1024, 2048, or 4096.
4. Click **Export** — generates:
   - `*_BaseColor.png` — Albedo map
   - `*_Normal.png` — Normal map (DirectX style for UE5)
   - `*_ORM.png` — Packed Occlusion/Roughness/Metallic
   - `*_Height.png` — Displacement/Height map (optional)

### Importing into UE5

1. Drag all exported PNG files into the UE5 Content Browser.
2. Create a new **Material**.
3. Connect:
   - BaseColor PNG → **Base Color** input
   - Normal PNG → **Normal** input (set texture to Normal Map type)
   - ORM PNG:
     - R channel → **Ambient Occlusion** input
     - G channel → **Roughness** input
     - B channel → **Metallic** input
4. Optionally connect Height PNG to a **World Position Offset** node via Tessellation (UE5.x) or use Displacement in the material.

## Advanced Features

### Custom Node Scripts (GDScript)

```gdscript
# Custom Material Maker node — generates a checkerboard pattern
tool
extends "res://addons/material_maker/engine/gen_shader.gd"

func _get_shader_code(uv, output_index, context):
    var checker = "(mod(floor(%s.x*10.0)+floor(%s.y*10.0), 2.0))" % [uv, uv]
    return { "rgb": "vec3(%s)" % checker, "a": "1.0" }
```

### Subgraphs / Templates

- Select a group of nodes → **Right-click > Group**.
- Groups become reusable subgraph templates.
- Save to the library for reuse across projects.

### Material Variations

- Use **Randomize** nodes to generate infinite variations from the same graph.
- Export multiple variation sets by changing a seed parameter.

## Integration with Blender

- Export from Material Maker as PNG maps.
- Import into Blender's **Shader Editor** using **Image Texture** nodes.
- Or apply directly in UE5 — no Blender step required.

## AI / MCP Integration

Material Maker does not currently have an official MCP server. However, its export pipeline is scriptable, and procedural material generation can be driven by AI agents through:
- GDScript automation scripts
- CLI batch export (Godot headless mode)

Example headless export:
```bash
material-maker --headless --export my_material.mmb --output /path/to/textures/
```

See [docs/mcp-servers/overview.md](../mcp-servers/overview.md) for current MCP status.

## References

- [Material Maker Documentation](https://www.materialmaker.org/documentation)
- [Material Maker GitHub](https://github.com/RodZill4/material-maker)
- [Material Maker Tutorials (YouTube)](https://www.youtube.com/results?search_query=material+maker+tutorial)
