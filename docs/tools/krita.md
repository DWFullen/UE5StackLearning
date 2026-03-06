# Krita

## Overview

Krita is a free, open-source digital painting application designed primarily for illustration, concept art, and texture painting. In the UE5 pipeline, it is used for creating concept art, hand-painted textures, character reference sheets, matte paintings, and stylized texture work.

## Key Features for UE5 Artists

- **Brush Engine** — physics-based, textured, and smear brushes ideal for concept work.
- **Layer System** — full Photoshop-compatible layer stack with blend modes.
- **Wrap-Around Mode** — paint seamless tiling textures that preview as tiles.
- **HDR Painting** — 32-bit per channel support for painting HDR/EXR textures.
- **Reference Images** — pin floating reference images while painting.
- **Symmetry Tools** — radial and mirrored symmetry for patterns and icons.
- **Animation** — frame-by-frame 2D animation with onion skinning.
- **Python Scripting** — full automation via Krita's Python API.

## Interface Overview

- **Toolbar** (left) — brushes, fill, selection, transform tools.
- **Docker panels** — Layers, Colors, Brushes, Channels, Palette.
- **Canvas** — primary painting area; supports canvas rotation and mirroring.
- **Popup Palette** — right-click on canvas for quick brush/color access.

## Common Workflows

### Creating Concept Art for UE5

1. **File > New** — set resolution to 300 DPI for print-quality; 72 DPI for screen reference.
2. Use a rough sketch layer with **Basic-1 Wet** brush.
3. Add color on new layers below the sketch using **Overlay** or **Multiply** blend modes.
4. Refine details with custom textured brushes.
5. Export as **PNG** or **PSD** for sharing with the team.

### Hand-Painted Textures

1. Create a **2048×2048** canvas (power-of-two for UE5).
2. Enable **View > Wrap Around Mode** — see the texture tile seamlessly while painting.
3. Use the **Texture Overlay** brush preset to add surface detail.
4. Export as **PNG** for import into UE5 as a Diffuse/Albedo texture.

### Painting Normal Maps

1. Install and enable the **Normal Map** plugin (Krita 5+).
2. Or use the **Mutator** feature to convert grayscale height detail to normal vectors.
3. Export as PNG with **Linear** color profile.
4. In UE5 import settings, set **Texture Type: Normal Map**.

### Painting on 3D Meshes (via Krita + Blender)

Krita doesn't support 3D painting natively. Workflow:
1. Unwrap UV in Blender.
2. Render a UV template image from Blender (UV > Export UV Layout).
3. Open template in Krita and paint the texture on top.
4. Export and apply back in Blender or UE5.

### Character Reference Sheets

1. Create multi-layer artwork with front, side, and back poses.
2. Use **Guide Layers** for consistent proportions.
3. Export as PSD or flattened PNG.
4. Reference sheet can be used in Blender as a background image for modeling.

### Exporting for UE5

| Asset Type | Format | Notes |
|------------|--------|-------|
| Diffuse/Albedo | PNG 8-bit (sRGB) | Flatten layers first |
| Emissive | PNG 8-bit or EXR 16-bit | Use HDR canvas for EXR |
| Masks / Roughness | PNG 8-bit (grayscale) | Desaturate final layer |
| Normal Map | PNG 8-bit (linear) | Set canvas color profile to sRGB→export as linear |
| HDR Panorama | EXR 32-bit | Requires 32-bit canvas |

## Python Scripting

```python
from krita import Krita

# Access the active document
app = Krita.instance()
doc = app.activeDocument()

# Get canvas size
print(f"Width: {doc.width()}, Height: {doc.height()}")

# Flatten and export
doc.flatten()
doc.exportImage("/tmp/output/texture.png", InfoObject())
```

Run scripts from **Tools > Scripts > Script Manager** or the **Python Scripter** plugin.

## Useful Krita Plugins

| Plugin | Purpose |
|--------|---------|
| Comics Manager | Page/panel management for concept art |
| Batch Exporter | Export multiple layers as separate files |
| Krita-AI-Diffusion | In-app Stable Diffusion image generation |
| Ten Brushes | Quick 10-brush shortcut palette |

## Krita-AI-Diffusion Plugin

The **Krita-AI-Diffusion** plugin integrates Stable Diffusion directly into Krita's canvas, enabling:
- Text-to-image generation on a new layer
- Inpainting on selected regions
- Upscaling and style transfer

Repository: [Acly/krita-ai-diffusion](https://github.com/Acly/krita-ai-diffusion)

This makes Krita particularly powerful for AI-assisted concept art generation.

## AI / MCP Integration

Krita does not currently have an official MCP server. However, its Python API allows automation:
- Batch texture creation and export
- Layer composition and blending
- Integration with Stable Diffusion via the AI Diffusion plugin

See [docs/mcp-servers/overview.md](../mcp-servers/overview.md) for current MCP status.

## References

- [Krita Documentation](https://docs.krita.org)
- [Krita Python API](https://api.kde.org/krita/html/index.html)
- [Krita-AI-Diffusion Plugin](https://github.com/Acly/krita-ai-diffusion)
