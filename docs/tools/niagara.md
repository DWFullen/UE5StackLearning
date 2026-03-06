# Niagara

## Overview

Niagara is Unreal Engine 5's visual effects and particle simulation framework. It replaces the legacy Cascade system and provides a highly customizable, data-driven approach to creating real-time particle effects, including GPU-accelerated simulations.

## Key Concepts

### Asset Hierarchy

```
Niagara System (.NS)
└── Niagara Emitter (.NE)   (one or more)
    ├── Emitter Properties
    ├── Emitter Summary
    └── Modules (stacked)
        ├── Emitter Update
        ├── Particle Spawn
        ├── Particle Update
        └── Render (Sprite, Mesh, Ribbon, Light)
```

### Core Building Blocks

| Asset | Purpose |
|-------|---------|
| **Niagara System** | Top-level container; holds one or more emitters |
| **Niagara Emitter** | Defines a population of particles |
| **Niagara Module** | Reusable node (function) applied to particles |
| **Niagara Parameter Collection** | Shared parameters across multiple systems |
| **Niagara Script** | Custom module logic (HLSL + visual scripting) |

### Execution Stages

| Stage | Runs When |
|-------|-----------|
| **System Spawn** | Once when the system is first spawned |
| **System Update** | Every frame |
| **Emitter Spawn** | Once per emitter lifetime |
| **Emitter Update** | Every frame per emitter |
| **Particle Spawn** | Once per new particle |
| **Particle Update** | Every frame per live particle |

## Opening Niagara

1. **Content Browser > Add New > FX > Niagara System**.
2. Choose a template (e.g., Empty System, Fountain, GPU Sprite) or create from emitter.
3. Double-click the `.NS` asset to open Niagara Editor.

## Common Workflows

### Creating a Basic Particle Effect

1. Create a **Niagara System** from the **Fountain** template.
2. In the emitter, adjust **Spawn Rate** module to set particles per second.
3. Edit **Initialize Particle** to set lifetime, size, and color.
4. Add **Add Velocity** or **Add Velocity from Point** modules.
5. Add a **Gravity Force** module for downward pull.
6. Use the **Sprite Renderer** module and assign a particle material.

### GPU Simulation

- In the emitter settings, set **Sim Target** to **GPU Compute Sim**.
- GPU sims support millions of particles but have restrictions:
  - No Skeletal Mesh sampling (use pre-baked data).
  - Limited collision options (use **Depth Buffer** or **Distance Fields**).
- Enable `r.Niagara.GPUSimulation 1` in console if not active.

### Collision

| Method | Best For |
|--------|---------|
| **Collision (CPU)** | Accurate, scene query-based; few particles |
| **Depth Buffer Collision** | Fast GPU; screen-space only |
| **Distance Field Collision** | GPU; global, accurate off-screen |

### Blueprint / C++ Control

```cpp
// Spawn a Niagara System at runtime
#include "NiagaraFunctionLibrary.h"
UNiagaraFunctionLibrary::SpawnSystemAtLocation(
    GetWorld(),
    NiagaraSystemAsset,
    SpawnLocation,
    FRotator::ZeroRotator
);
```

```cpp
// Set a parameter on a spawned component
UNiagaraComponent* NiagaraComp = GetNiagaraComponent();
NiagaraComp->SetVariableFloat(TEXT("User.SpeedScale"), 2.0f);
NiagaraComp->SetVariableLinearColor(TEXT("User.Color"), FLinearColor::Red);
```

### Fluid Simulation

- UE5 includes **Fluid Ninja Live** integration and the **Grid3D** Niagara data interface for volumetric fluid sims.
- For production fluid sims, integrate with Houdini Engine (via Houdini Niagara plugin).

## Performance Optimization

- Use **GPU simulation** for high-count particle systems.
- Enable **Significance Manager** — Niagara can automatically cull or LOD distant effects.
- Set **Fixed Bounds** on emitters to avoid per-frame bounds recalculation.
- Use **Scalability** settings per emitter to reduce particle counts at lower quality levels.
- Profile with **Niagara Debug HUD** — set `fx.NiagaraDebugDraw.Enabled 1`.

## Debugging

| Tool | Access |
|------|--------|
| Niagara Debugger | Window > Debug > Niagara Debugger |
| Debug HUD | `fx.Niagara.DebugHUDEnabled 1` in console |
| Attribute Spreadsheet | Inside Niagara Editor, Window > Attribute Spreadsheet |
| Script Preview | Click a module to preview its HLSL |

## Common Modules Reference

| Module | Description |
|--------|-------------|
| Spawn Rate | Continuous emission over time |
| Spawn Burst Instantaneous | One-shot burst of particles |
| Initialize Particle | Set initial lifetime, size, color, mass |
| Add Velocity | Apply initial velocity |
| Gravity Force | Apply gravity vector |
| Drag | Air resistance |
| Curl Noise Force | Turbulence-based movement |
| Scale Color | Animate color over lifetime |
| Scale Sprite Size | Animate size over lifetime |
| Collision | Particle-world collision |
| Sprite Renderer | Render as camera-facing quads |
| Mesh Renderer | Render as 3D meshes |
| Ribbon Renderer | Render as a continuous ribbon |

## AI / MCP Integration

Niagara systems can be parameterized extensively via **User Exposed Variables**, making them ideal for AI-driven control. An agent can adjust spawn rates, forces, and colors at runtime by setting Niagara parameters through Blueprint or C++. See [docs/mcp-servers/overview.md](../mcp-servers/overview.md).

## References

- [Niagara Overview](https://docs.unrealengine.com/5.0/en-US/overview-of-niagara-effects-for-unreal-engine/)
- [Niagara Key Concepts](https://docs.unrealengine.com/5.0/en-US/key-concepts-in-niagara-effects-for-unreal-engine/)
- [GPU Simulation](https://docs.unrealengine.com/5.0/en-US/gpu-particle-simulations-in-unreal-engine/)
