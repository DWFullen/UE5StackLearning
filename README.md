# UE5StackLearning

A documentation and knowledge repository for understanding the Unreal Engine 5 development stack and implementing AI tooling across the full pipeline.

## Purpose

This repo serves two goals:

1. **Learning Resource** — document workflows, best practices, and tool integrations for the UE5 development stack.
2. **Agent Instruction Source** — generate instruction files that AI agents (Copilot, Cursor, Claude, etc.) can reference when working in other project repos.

## Tools Covered

### Unreal Engine 5 Native Tools

| Tool | Description | Docs |
|------|-------------|------|
| Unreal Editor | Main IDE for UE5 projects | [docs/tools/unreal-editor.md](docs/tools/unreal-editor.md) |
| Sequencer | Cinematic and animation sequencer | [docs/tools/sequencer.md](docs/tools/sequencer.md) |
| Niagara | Visual effects and particle system | [docs/tools/niagara.md](docs/tools/niagara.md) |
| Modeling Mode | In-editor mesh modeling tools | [docs/tools/modeling-mode.md](docs/tools/modeling-mode.md) |
| Unreal Game Sync (UGS) | P4/Git sync and build tool for teams | [docs/tools/unreal-game-sync.md](docs/tools/unreal-game-sync.md) |
| Live Link | Real-time data streaming into UE5 | [docs/tools/live-link.md](docs/tools/live-link.md) |
| Gameplay Ability System (GAS) | Ability, attribute, and effect framework | [docs/tools/gameplay-ability-system.md](docs/tools/gameplay-ability-system.md) |

### External DCC & Creative Tools

| Tool | Description | Docs |
|------|-------------|------|
| Blender | 3D modeling, rigging, animation, and rendering | [docs/tools/blender.md](docs/tools/blender.md) |
| GIMP | Raster image editing and texture work | [docs/tools/gimp.md](docs/tools/gimp.md) |
| Krita | Digital painting and concept art | [docs/tools/krita.md](docs/tools/krita.md) |
| Material Maker | Procedural material/texture authoring | [docs/tools/material-maker.md](docs/tools/material-maker.md) |
| Natron | Node-based compositing and VFX | [docs/tools/natron.md](docs/tools/natron.md) |
| OBS Studio | Streaming, recording, and capture | [docs/tools/obs-studio.md](docs/tools/obs-studio.md) |

### Languages & Scripting

| Language | Description | Docs |
|----------|-------------|------|
| C++ | Core gameplay and engine programming | [docs/languages/cpp.md](docs/languages/cpp.md) |
| Blueprint Visual Scripting | UE5 node-based visual scripting | [docs/languages/blueprint.md](docs/languages/blueprint.md) |

## MCP Servers

Model Context Protocol (MCP) servers allow AI agents to directly interact with tools. See [docs/mcp-servers/overview.md](docs/mcp-servers/overview.md) for the full list and setup instructions.

## Agent Instructions

Instruction files for AI agents are located in the [agents/](agents/) directory. These can be copied into other repos to give AI assistants context about each tool and workflow.

## Repository Structure

```
UE5StackLearning/
├── README.md
├── docs/
│   ├── tools/              # Per-tool documentation
│   ├── languages/          # Language-specific docs (C++, Blueprint)
│   └── mcp-servers/        # MCP server setup guides
└── agents/
    └── instructions/       # Agent instruction files per tool
```
