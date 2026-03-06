# GIMP

## Overview

GIMP (GNU Image Manipulation Program) is a free, open-source raster image editor. In the UE5 pipeline, it is used for texture creation and editing, photo manipulation for texture work, and preparing 2D assets such as UI elements, decals, and splat maps.

## Key Uses in UE5 Pipeline

| Task | GIMP Workflow |
|------|--------------|
| Texture authoring | Create/edit diffuse, roughness, metallic maps |
| Normal map editing | Adjust or combine normal maps |
| Alpha mask creation | Create opacity masks for foliage, decals |
| Heightmap creation | Paint grayscale heightmaps for landscapes |
| Texture atlasing | Arrange sprites/tiles into texture atlases |
| UI asset creation | Design HUD elements and menu graphics |
| Color grading LUTs | Create lookup tables for post-processing |

## Interface Overview

- **Toolbox** — brush, paint, selection, transform tools.
- **Tool Options** — per-tool settings docked below the toolbox.
- **Layers Dialog** — layer stack management.
- **Channels Dialog** — RGB + alpha channel control.
- **Paths Dialog** — vector paths for precise masking.
- **Script-Fu Console** — GIMP's Scheme-based scripting console.

## Common Workflows

### Creating a Texture for UE5

1. **File > New** — set size to power-of-two (512, 1024, 2048, 4096).
2. Set **Color Space** to sRGB for diffuse/albedo, or linear for data maps (roughness, metallic, normal).
3. Paint or import photo reference.
4. Use **Filters > Enhance > Unsharp Mask** for sharpening.
5. **File > Export As** — choose PNG (lossless) or TGA for import into UE5.

### Extracting a Channel (for ORM Maps)

UE5 uses packed ORM (Occlusion/Roughness/Metallic) textures — R=AO, G=Roughness, B=Metallic:

1. Open each map (AO, roughness, metallic) as separate images.
2. In the AO image: **Colors > Components > Decompose** to get R channel.
3. Repeat for roughness (G) and metallic (B).
4. Use **Colors > Components > Compose** in a new image to combine R, G, B.
5. Export as PNG.

### Normal Map Workflow

1. Paint or edit normals in GIMP.
2. **Filters > Map > Normal Map** plugin (requires GIMP Normal Map plugin or GIMP 2.10+).
3. Set strength and filter type.
4. Export as PNG — import in UE5 with **Texture Type: Normal Map**.

### Creating Landscape Heightmaps

1. Create a new 16-bit grayscale image (must match landscape dimensions — e.g., 1009×1009 for 1km landscape).
2. Paint terrain heights (white = high, black = low).
3. Export as **16-bit PNG** (File > Export As, change bit depth in PNG options).
4. Import in UE5: **Landscape tool > Import from file**.

### Batch Processing with Script-Fu

```scheme
; GIMP Script-Fu — resize and export all PNG files in a folder
(let* ((filelist (cadr (file-glob "/path/to/textures/*.png" 1))))
  (for-each
    (lambda (filename)
      (let* ((image (car (gimp-file-load RUN-NONINTERACTIVE filename filename)))
             (drawable (car (gimp-image-get-active-drawable image))))
        (gimp-image-scale-full image 2048 2048 INTERPOLATION-LINEAR)
        (file-png-save RUN-NONINTERACTIVE image drawable filename filename 0 9 1 1 1 1 1)))
    filelist))
```

## Useful Plugins for UE5 Work

| Plugin | Purpose |
|--------|---------|
| Normal Map plugin | Normal map generation/editing |
| G'MIC | Advanced filters (tiling, texture synthesis) |
| BIMP | Batch image processing |
| GIMP-DDS | Import/export DirectX DDS textures (useful for legacy workflows) |
| Resynthesizer | Content-aware fill |

## Texture Export Settings for UE5

| Map Type | Format | Color Space | Bit Depth |
|----------|--------|-------------|-----------|
| Albedo / Diffuse | PNG / TGA | sRGB | 8-bit |
| Normal Map | PNG / TGA | Linear | 8-bit |
| ORM (packed) | PNG | Linear | 8-bit |
| Roughness / Metallic | PNG | Linear | 8-bit |
| Heightmap | PNG | Linear | 16-bit |
| Emissive | PNG / EXR | Linear | 8-16-bit |
| HDR Panorama | EXR | Linear | 32-bit float |

## AI / MCP Integration

GIMP supports scripting via **Script-Fu** (Scheme) and **Python-Fu** (Python). While there is no official GIMP MCP server, GIMP can be driven via its batch mode for AI-pipeline texture processing:

```bash
gimp -i -b '(let* ((image (car (gimp-file-load RUN-NONINTERACTIVE "input.png" "input.png"))))
              (gimp-image-scale-full image 2048 2048 INTERPOLATION-LINEAR)
              (file-png-save RUN-NONINTERACTIVE image (car (gimp-image-get-active-drawable image)) "output.png" "output.png" 0 9 1 1 1 1 1)
              (gimp-quit 0))'
```

See [docs/mcp-servers/overview.md](../mcp-servers/overview.md) for the current status of GIMP MCP tooling.

## References

- [GIMP Documentation](https://docs.gimp.org)
- [GIMP Script-Fu Reference](https://docs.gimp.org/2.10/en/gimp-using-script-fu.html)
- [GIMP Python-Fu](https://docs.gimp.org/2.10/en/gimp-using-python-fu.html)
