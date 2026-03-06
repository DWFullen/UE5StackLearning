# Agent Instruction Files

This directory contains instruction files that can be copied into other repositories to give AI agents (GitHub Copilot, Cursor, Claude, etc.) context about the UE5 development stack.

## Available Instruction Files

| File | Use Case | Copy to |
|------|----------|---------|
| [ue5-full-stack.md](instructions/ue5-full-stack.md) | Full UE5 project with all tools | Root `AGENT.md` |
| [ue5-cpp.md](instructions/ue5-cpp.md) | C++ heavy UE5 project | Root `AGENT.md` or `.cursorrules` |
| [ue5-blueprint.md](instructions/ue5-blueprint.md) | Blueprint-focused UE5 project | Root `AGENT.md` or `.cursorrules` |
| [ue5-asset-pipeline.md](instructions/ue5-asset-pipeline.md) | Asset creation and import workflow | Root `AGENT.md` |
| [blender-ue5.md](instructions/blender-ue5.md) | Blender work targeting UE5 | Root `AGENT.md` in Blender project |
| [ue5-gas.md](instructions/ue5-gas.md) | Projects using the Gameplay Ability System | Root `AGENT.md` or `.cursorrules` |

## How to Use

### Option 1: AGENT.md (GitHub Copilot)

Copy the relevant file to your project root as `AGENT.md`:

```bash
cp agents/instructions/ue5-full-stack.md /path/to/my-project/AGENT.md
```

GitHub Copilot Coding Agent automatically reads `AGENT.md` from the repository root.

### Option 2: .cursorrules (Cursor IDE)

Copy the content into `.cursorrules` in your project root:

```bash
cp agents/instructions/ue5-cpp.md /path/to/my-project/.cursorrules
```

### Option 3: System Prompt (Any Agent)

Paste the content directly into an AI assistant's system prompt or conversation context.

### Option 4: Claude Project Instructions

Paste the content into a Claude Project's **Project Instructions** field.

## Combining Multiple Files

For projects that use multiple tools, combine relevant instruction files:

```bash
cat agents/instructions/ue5-cpp.md agents/instructions/ue5-asset-pipeline.md > /path/to/my-project/AGENT.md
```

## Contributing New Instruction Files

When adding a new instruction file:

1. Create `agents/instructions/<tool-name>.md`.
2. Structure it with:
   - Tool overview and version
   - MCP server configuration (if available)
   - Key conventions and rules
   - Common agent task examples
   - References to detailed docs
3. Add an entry to this README.
