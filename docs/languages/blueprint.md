# Blueprint Visual Scripting

## Overview

Blueprint Visual Scripting is Unreal Engine 5's node-based programming system. It allows designers and programmers to create gameplay logic, UI interactions, animations, and more without writing C++ code. Blueprints compile to bytecode interpreted by the UE5 VM and can interoperate seamlessly with C++.

## Blueprint Types

| Type | Description |
|------|-------------|
| **Blueprint Class** | The most common type; extends a C++ or Blueprint base class |
| **Level Blueprint** | One-per-level; handles level-specific events |
| **Blueprint Interface** | Defines a set of function signatures (no implementation) |
| **Blueprint Function Library** | Pure static utility functions callable from any Blueprint |
| **Blueprint Macro Library** | Reusable node groups with inputs and outputs |
| **Animation Blueprint** | Controls skeletal mesh animation logic |
| **Widget Blueprint** | UMG-based UI screens |
| **Editor Utility Blueprint** | Extends the Unreal Editor with custom tools |

## Blueprint Editor Layout

- **Graph Editor** — main node graph canvas; `Right-click` to add nodes.
- **My Blueprint** panel — lists variables, functions, macros, event graphs, and components.
- **Details** panel — properties of the selected node or variable.
- **Components** panel — component hierarchy for Blueprint Class actors.
- **Toolbar** — Compile, Save, Play-in-Editor, Find (search in graph).

## Event Graph vs. Functions vs. Macros

| Concept | Has Latent Nodes | Has Multiple Outputs | Purpose |
|---------|-----------------|---------------------|---------|
| **Event Graph** | ✅ Yes (Delay, timelines) | ✅ Yes | Async gameplay logic |
| **Function** | ❌ No | ❌ Single output | Pure synchronous logic |
| **Macro** | ✅ Yes | ✅ Yes | Code reuse with flexible pins |

## Core Node Types

### Execution Flow

| Node | Description |
|------|-------------|
| Event Tick | Runs every frame |
| Event BeginPlay | Runs when game starts |
| Event Overlap / Hit | Physics callbacks |
| Branch | If/else |
| Sequence | Execute multiple outputs in order |
| For Loop | Indexed iteration |
| For Each Loop | Iterate an array |
| While Loop | Loop with condition |
| Flip Flop | Alternates between A and B |
| Gate | Open/close execution flow |
| Delay | Wait N seconds (latent) |
| Timeline | Animate values over time (latent) |
| DoOnce | Execute only first time |
| DoN | Execute only N times |

### Variables & Data

| Node | Description |
|------|-------------|
| Get/Set Variable | Read/write Blueprint variables |
| Make / Break Struct | Pack/unpack struct values |
| Make Array | Create an array literal |
| Select | Switch-case for data |
| Cast To | Type-safe downcasting |
| Is Valid | Null/validity check |

### Math & Utilities

- Arithmetic: Add, Subtract, Multiply, Divide
- Comparison: Equal, Not Equal, Greater Than, etc.
- Lerp, Clamp, Abs, Round, Floor, Ceil
- Vector math: Normalize, Dot Product, Cross Product, Distance
- Random: Random Float, Random Int, Random Bool, Random Unit Vector

## Variables

### Creating a Variable

1. In **My Blueprint** panel, click **+** next to Variables.
2. Set Name, Type, and Default Value in the Details panel.
3. Toggle **Instance Editable** (`E`) to expose to the editor and allow per-instance customization.
4. Toggle **Blueprint Read Only** (`R`) to prevent modification at runtime.

### Variable Types

| Category | Types |
|----------|-------|
| Primitive | Boolean, Integer, Integer64, Float, Double, Byte, String, Name, Text |
| Object | Actor, Component, and any UObject-derived type |
| Struct | Vector, Rotator, Transform, LinearColor, custom structs |
| Enum | Any `UENUM` or Blueprint enum |
| Array | `TArray` of any type |
| Set | `TSet` of any type |
| Map | `TMap<Key, Value>` |

## Functions

### Creating a Function

1. Click **+Function** in My Blueprint panel.
2. Add inputs/outputs in the Details panel of the function entry node.
3. Build the logic graph between the inputs and outputs nodes.
4. Mark as **Pure** to create a function without execution pins (like a getter).

### Calling C++ Functions from Blueprint

Any `UFUNCTION(BlueprintCallable)` or `UFUNCTION(BlueprintPure)` C++ function is automatically available in Blueprint:

```
Right-click in graph → search for the function name
```

## Events

### Custom Events

- Right-click in graph → "Add Custom Event".
- Can be called from other Blueprints with **Call [EventName]** on an object reference.
- Can be made `Reliable` or `Unreliable` for replication.

### Dispatchers (Event Delegates)

1. Create an **Event Dispatcher** in My Blueprint.
2. Call it with **Call [DispatcherName]** to fire it.
3. Other Blueprints can **Bind** to it to receive the event.
4. Similar to C++ multicast delegates.

## Blueprint Communication Patterns

### Direct Reference

```
Get reference to actor → Cast → Call function
```

Best for: Known, persistent references between specific actors.

### Blueprint Interface

1. Create a Blueprint Interface with function signatures.
2. Implement it in target Blueprints.
3. Call interface functions on any actor — works even without knowing the concrete type.

Best for: Decoupled communication where the caller doesn't need to know the receiver's type.

### Event Dispatchers / Delegates

Bind→Call pattern for one-to-many event notifications.

Best for: UI updating when game state changes, achievement triggers, etc.

### Cast

```
Get Actor of Class → Cast To SpecificClass → Use specific functions
```

Best for: Simple one-off queries where performance is not a concern.

## Animation Blueprint

Animation Blueprints control skeletal mesh animation:

- **Event Graph** — updates variables (speed, falling state) each frame.
- **AnimGraph** — defines the final pose:
  - **State Machine** — states (Idle, Walk, Run, Jump) with transition rules.
  - **Blend Space** — blend between animations based on float inputs (speed, direction).
  - **Layered Blend** — overlay upper-body animations on lower-body.
  - **Control Rig** — procedural bone manipulation.

## UMG Widget Blueprints

For UI development:

1. Create a **Widget Blueprint** (User Interface > Widget Blueprint).
2. Design layout in the **Designer** tab (drag-and-drop UMG widgets).
3. Add logic in the **Graph** tab (bindings, event handlers).
4. Spawn in-game:
   ```
   Create Widget → [Widget Class] → Add to Viewport
   ```

## Blueprint vs. C++

| Situation | Use Blueprint | Use C++ |
|-----------|--------------|---------|
| Rapid prototyping | ✅ | |
| Designer-owned logic | ✅ | |
| UI / HUD | ✅ | |
| Performance-critical loops | | ✅ |
| Complex data structures | | ✅ |
| Engine subsystems | | ✅ |
| Reusable library code | | ✅ |
| Large teams with version control | | ✅ (text-based) |
| Network replication | Both | |

> **Best practice**: Implement core systems in C++ and expose them to Blueprint for designer customization.

## Blueprint Nativization (Deprecated)

UE5 no longer supports Blueprint Nativization. For performance-critical Blueprint code, manually convert to C++.

## Debugging Blueprints

- **Breakpoints** — right-click a node → Add Breakpoint.
- **Watch Values** — right-click a variable pin → Watch This Value.
- **Blueprint Debugger** — Window > Blueprint Debugger.
- **Print String** — quick debug output to viewport and log.
- **Draw Debug** nodes — visualize vectors, spheres, lines in the viewport.

## AI / MCP Integration

Blueprints can be generated and manipulated by AI agents through:
- **Unreal Python** — `unreal.EditorAssetLibrary` to create and modify Blueprint assets.
- **Editor Utility Widgets** — AI agent builds Blueprint-based tools.
- **Blueprint callable AI functions** — Expose AI/ML model inference as `UFUNCTION(BlueprintCallable)`.

See [docs/mcp-servers/overview.md](../mcp-servers/overview.md) for MCP tools.

## References

- [Blueprint Visual Scripting Overview](https://docs.unrealengine.com/5.0/en-US/introduction-to-blueprints-visual-scripting-in-unreal-engine/)
- [Blueprint Best Practices](https://docs.unrealengine.com/5.0/en-US/blueprint-best-practices-in-unreal-engine/)
- [Animation Blueprint](https://docs.unrealengine.com/5.0/en-US/animation-blueprints-in-unreal-engine/)
- [UMG UI Designer](https://docs.unrealengine.com/5.0/en-US/umg-ui-designer-overview/)
