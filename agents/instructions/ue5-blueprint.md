# Agent Instructions: Blueprint Visual Scripting

Copy this file into a UE5 Blueprint-focused project as `AGENT.md` or `.cursorrules` to give AI agents context about Blueprint conventions and patterns.

---

## Tool: Blueprint Visual Scripting (UE5)

**Engine Version**: UE5.x
**Type**: Node-based visual programming compiled to UE5 bytecode
**Interop**: Full C++ interoperability via UFUNCTION/UPROPERTY reflection

---

## Blueprint Types Quick Reference

| Type | When to Use |
|------|-------------|
| **Blueprint Class** | Actor with custom behavior |
| **Animation Blueprint** | Skeletal mesh animation logic |
| **Widget Blueprint** | UI screens and HUD |
| **Blueprint Interface** | Decoupled multi-actor communication |
| **Blueprint Function Library** | Shared pure utility functions |
| **Level Blueprint** | Level-specific scripting |
| **Editor Utility Blueprint** | Custom editor tools |

---

## Naming Conventions

| Blueprint Type | Prefix | Example |
|---------------|--------|---------|
| Blueprint Class | `BP_` | `BP_Door` |
| Animation Blueprint | `ABP_` | `ABP_Character` |
| Widget Blueprint | `WBP_` | `WBP_HUD` |
| Blueprint Interface | `BPI_` | `BPI_Interactable` |
| Blueprint Function Library | `BPFL_` | `BPFL_MathUtils` |

---

## Variable Best Practices

1. **Name clearly** — `PlayerHealth` not `h` or `var1`.
2. **Set Category** to organize variables in the Details panel.
3. **Mark Instance Editable** (`E`) for per-instance customization.
4. **Use proper types** — avoid converting between types unnecessarily.
5. **Initialize in Event BeginPlay** — don't rely on default values for references.

---

## Graph Organization Rules

1. Add **comment boxes** around logical groups of nodes.
2. Use **reroute nodes** to prevent spaghetti connections.
3. **Collapse complex logic** into named functions or macros.
4. Keep the main Event Graph **high-level** — delegate detail work to functions.
5. Avoid **Event Tick** for logic that can be event-driven.

---

## Communication Patterns

### Direct Reference (known targets)
```
GetPlayerCharacter → Cast To BP_MyCharacter → Call Function
```

### Blueprint Interface (unknown targets)
```
// In BPI_Interactable: define Interact() function
// In BP_Door: implement BPI_Interactable interface → fill Interact graph
// Caller: Get Actor → Does Implement Interface → Interact (message)
```

### Event Dispatcher (one-to-many)
```
// In the broadcaster:
Event Dispatcher: OnStateChanged

// To fire:
Call OnStateChanged

// In listeners:
Bind to OnStateChanged → [callback event]
```

---

## Animation Blueprint Structure

```
Event Graph:
  Event Blueprint Update Animation
    → Get Owning Pawn → Cast to BP_Character
    → Get Velocity → VectorLength → Set "Speed"
    → Get Is Falling → Set "bIsInAir"

AnimGraph:
  State Machine:
    - Idle       (Speed < 10)
    - Walk       (Speed 10–200)
    - Run        (Speed > 200)
    - Jump/Fall  (bIsInAir = true)
  └── Output Pose
```

---

## UMG Widget Blueprint Patterns

### Binding Data to Widget

```
// In Widget Blueprint "Graph":
Event Construct
  → Get Game Instance → Cast to GI_MyGame
  → Bind Event to OnScoreChanged → Update Score Display

// OnScoreChanged callback:
[Score (int)] → Format Text "{0}" → Set Text (ScoreText widget)
```

### Showing/Hiding Widgets

```
// Show:
Create Widget (WBP_HUD, owning player) → Add to Viewport

// Hide:
Get WBP_HUD reference → Remove from Parent
```

---

## Performance Guidelines

1. **Do NOT** run expensive logic in `Event Tick` every frame — use timers instead.
2. **Cache** actor/component references in `BeginPlay`; don't use `GetAllActorsOfClass` per-frame.
3. **Avoid Cast** in tight loops — prefer interfaces.
4. Blueprint is ~10x slower than equivalent C++ for CPU-intensive code; offload heavy computation.
5. Use **Branch** (if) sparingly; group conditions with boolean logic before branching.

---

## Debugging Techniques

| Technique | How |
|-----------|-----|
| Print String | Add `Print String` node; shows on screen and log |
| Breakpoints | Right-click node → Add Breakpoint; pause during PIE |
| Watch Values | Right-click variable pin → Watch This Value |
| Draw Debug | `Draw Debug Sphere/Line/String` for visual debugging |
| Blueprint Debugger | Window > Blueprint Debugger |

---

## Calling C++ from Blueprint

Any `UFUNCTION(BlueprintCallable)` or `UFUNCTION(BlueprintPure)` function in C++ appears automatically in Blueprint:

```
Right-click in graph → search function name → select it
```

Cast C++ Actor reference to access its Blueprint-exposed functions:
```
Get Actor → Cast To AMyActor → My C++ Function
```

---

## Common Pitfalls

| Pitfall | Solution |
|---------|---------|
| Object reference is not valid | Always check `Is Valid` before using references |
| Blueprint not updating after C++ change | Recompile Blueprint (Compile button) |
| Tick running when not needed | Uncheck `Start with Tick Enabled` in Class Defaults |
| Cast failing silently | The exec pin on Cast's "Cast Failed" output will fire — handle it |
| Event Dispatcher not firing | Ensure `Bind` is called before `Call` |

---

## References

- [Full Blueprint Docs](../../docs/languages/blueprint.md)
- [Blueprint Overview](https://docs.unrealengine.com/5.0/en-US/introduction-to-blueprints-visual-scripting-in-unreal-engine/)
- [Blueprint Best Practices](https://docs.unrealengine.com/5.0/en-US/blueprint-best-practices-in-unreal-engine/)
- [UMG UI Designer](https://docs.unrealengine.com/5.0/en-US/umg-ui-designer-overview/)
