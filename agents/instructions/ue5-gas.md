# Agent Instructions: Gameplay Ability System (GAS)

Copy this file into a UE5 project that uses GAS as `AGENT.md` or `.cursorrules` to give AI coding agents full context about GAS architecture, conventions, and patterns.

---

## System: Gameplay Ability System (GAS)

**Plugin**: `GameplayAbilities` (built-in UE5 plugin, must be enabled)
**Engine Version**: UE5.x
**Module Dependencies**: `GameplayAbilities`, `GameplayTags`, `GameplayTasks`
**Reference Project**: Lyra Starter Game (Epic's production GAS sample)

---

## Core Class Hierarchy

```
UAbilitySystemComponent   — per-actor hub; manages all GAS state
UGameplayAbility          — a discrete action (attack, dash, cast spell)
UAttributeSet             — numeric attributes (Health, Mana, Damage)
UGameplayEffect           — data-only modifier applied to attributes/tags
UGameplayCueNotify        — cosmetic feedback (VFX, SFX) triggered by tags
UAbilityTask              — async operation running inside an ability
```

---

## Naming Conventions

| Asset Type | Prefix | Example |
|------------|--------|---------|
| Gameplay Ability Blueprint | `GA_` | `GA_Fireball` |
| Gameplay Effect Blueprint | `GE_` | `GE_FireDamage`, `GE_Cooldown_Fireball` |
| Attribute Set C++ class | `U` + `AttributeSet` suffix | `UHeroAttributeSet` |
| Gameplay Cue Notify | `GC_` | `GC_Impact_Fire` |
| Ability Task C++ class | `UAT_` | `UAT_WaitMontageEvent` |

---

## Gameplay Tags Convention

Use a consistent hierarchical namespace. Always define tags in `Config/DefaultGameplayTags.ini` — never hardcode tag strings in more than one place.

```
Ability.*         — ability identifiers (e.g. Ability.Fireball)
State.*           — character states (e.g. State.Dead, State.Stunned)
Effect.*          — effect categories (e.g. Effect.Burning, Effect.CC.Root)
Event.*           — gameplay events (e.g. Event.Damage.Physical)
Data.*            — Set-by-Caller magnitudes (e.g. Data.Damage, Data.HealAmount)
GameplayCue.*     — cue identifiers (e.g. GameplayCue.Impact.Fire)
```

---

## ASC Placement Rules

| Project Type | ASC Location | Why |
|-------------|-------------|-----|
| Single-player / co-op | `ACharacter` | Simpler; respawn handling is manual |
| Multiplayer (player characters) | `APlayerState` | Persists across respawns |
| AI enemies | `ACharacter` | No respawn persistence needed |

Always call `InitAbilityActorInfo(OwnerActor, AvatarActor)` on **both server and client**:
- Server: in `PossessedBy`
- Client: in `OnRep_PlayerState`

---

## Ability Lifecycle Rules

1. **Always call `EndAbility`** in every code path — including task cancel/interrupt delegates.
2. **Check `CommitAbility`** for cost+cooldown before performing the ability's action.
3. **Use `bRetriggerInstancedAbility`** only when re-triggering the same instance is intentional.
4. **Use `UAbilityTask` for all async work** — never use timers or latent nodes directly in ability code.
5. **Server abilities** should apply Gameplay Effects on the server; clients predict via `PredictionKey`.

```cpp
void UGA_Fireball::ActivateAbility(...)
{
    if (!CommitAbility(Handle, ActorInfo, ActivationInfo))
    {
        EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
        return;
    }
    // ... proceed with ability logic
}
```

---

## Attribute Set Rules

1. **Use `ATTRIBUTE_ACCESSORS` macro** for every attribute — generates `Get`, `Set`, `Init` functions.
2. **Clamp in `PreAttributeChange`** for immediate clamping (before the change is applied).
3. **React to effects in `PostGameplayEffectExecute`** for meta-attributes (e.g., `Damage` → drain `Health`).
4. **Every replicated attribute needs** `OnRep_` function + entry in `GetLifetimeReplicatedProps`.
5. **Meta-attributes** (e.g., `Damage`) should be `Transient` and not replicated — they are local computation buffers.

```cpp
void UHeroAttributeSet::PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)
{
    Super::PreAttributeChange(Attribute, NewValue);
    if (Attribute == GetHealthAttribute())
        NewValue = FMath::Clamp(NewValue, 0.f, GetMaxHealth());
}

void UHeroAttributeSet::PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data)
{
    Super::PostGameplayEffectExecute(Data);
    if (Data.EvaluatedData.Attribute == GetDamageAttribute())
    {
        const float LocalDamage = GetDamage();
        SetDamage(0.f);  // Clear meta-attribute
        SetHealth(FMath::Clamp(GetHealth() - LocalDamage, 0.f, GetMaxHealth()));
    }
}
```

---

## Gameplay Effect Rules

1. **Use `Instant`** for one-shot attribute changes (damage, healing snap).
2. **Use `Duration`** for temporary buffs/debuffs; use `Period` for tick-based effects (poison).
3. **Use `Infinite`** for permanent buffs; remove explicitly with `RemoveActiveEffectsWithGrantedTags`.
4. **Use Set-by-Caller magnitudes** (tagged with `Data.*`) when the magnitude is computed at runtime.
5. **Apply effects from abilities via `MakeOutgoingGameplayEffectSpec`** — never use `ApplyGameplayEffectToSelf` from outside an ability unless testing.

---

## Gameplay Cue Rules

1. **Never trigger cues from server RPCs directly** — trigger them via Gameplay Effects or `ExecuteGameplayCue` on the ASC; they replicate automatically.
2. **Use `UGameplayCueNotify_Static`** for instant, one-shot effects.
3. **Use `UGameplayCueNotify_Actor`** for looping or persistent effects.
4. **Cue tag root must be `GameplayCue.`** — the GAS subsystem only routes tags under this root.

---

## Replication Modes

| Mode | Use When |
|------|---------|
| `Full` | Single-player or small co-op (≤4 players) |
| `Mixed` | Competitive multiplayer — server sends minimal data to simulated proxies |
| `Minimal` | Large-scale — no per-effect replication; good for AI-controlled actors |

Set in code: `AbilitySystemComponent->SetReplicationMode(EGameplayEffectReplicationMode::Mixed);`

---

## Build.cs Dependency

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "GameplayAbilities",
    "GameplayTags",
    "GameplayTasks"
});
```

---

## Common Patterns

### Grant an Ability at Runtime

```cpp
FGameplayAbilitySpec Spec(UGA_Fireball::StaticClass(), Level);
AbilitySystemComponent->GiveAbility(Spec);
```

### Activate by Tag

```cpp
FGameplayTagContainer Tags;
Tags.AddTag(FGameplayTag::RequestGameplayTag("Ability.Fireball"));
AbilitySystemComponent->TryActivateAbilitiesByTag(Tags);
```

### Apply an Effect Programmatically

```cpp
FGameplayEffectContextHandle Ctx = AbilitySystemComponent->MakeEffectContext();
FGameplayEffectSpecHandle Spec = AbilitySystemComponent->MakeOutgoingSpec(
    UGE_Heal::StaticClass(), 1.f, Ctx);
Spec.Data->SetSetByCallerMagnitude(
    FGameplayTag::RequestGameplayTag("Data.HealAmount"), 50.f);
AbilitySystemComponent->ApplyGameplayEffectSpecToSelf(*Spec.Data.Get());
```

### Listen for Attribute Changes

```cpp
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(
    UHeroAttributeSet::GetHealthAttribute()
).AddUObject(this, &AMyHUD::OnHealthChanged);
```

### Check for a State Tag

```cpp
bool bIsStunned = AbilitySystemComponent->HasMatchingGameplayTag(
    FGameplayTag::RequestGameplayTag("State.Stunned"));
```

---

## Debugging Commands

| Command | Description |
|---------|-------------|
| `showdebug abilitysystem` | Real-time overlay of effects, tags, attributes |
| `AbilitySystem.Debug.PrintAttributes` | Log attribute values to output log |
| `AbilitySystem.Verbose 1` | Enable verbose GAS logging |
| `' ` (backtick) key in PIE | Open Gameplay Debugger (select actor first) |

---

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `InitAbilityActorInfo` not called on client | Call in `OnRep_PlayerState` |
| Attributes not replicating | Add `OnRep_` + `GetLifetimeReplicatedProps` |
| `EndAbility` not called | Handle all task cancel/interrupt delegates |
| Effects not applying | Verify attribute name; check GE modifier operation |
| Cues not playing on clients | Use ASC-level cue execution; never RPC cues manually |
| Tag not recognized | Register tag in `DefaultGameplayTags.ini` |

---

## MCP / AI Integration

This project uses a GAS MCP server for AI-assisted development. Key tools available:

| Tool | Description |
|------|-------------|
| `gas_get_attributes` | Read current attribute values from an actor's ASC |
| `gas_get_active_effects` | List active Gameplay Effects on an actor |
| `gas_get_active_tags` | Get all Gameplay Tags on an actor's ASC |
| `gas_activate_ability` | Trigger an ability by tag |
| `gas_apply_effect` | Apply a GE Blueprint to an actor |
| `gas_send_gameplay_event` | Send a Gameplay Event to trigger event-based abilities |
| `gas_list_granted_abilities` | List all granted abilities on an actor |

See [docs/mcp-servers/gameplay-ability-system.md](../../docs/mcp-servers/gameplay-ability-system.md) for server setup.

---

## References

- [Full GAS Docs](../../docs/tools/gameplay-ability-system.md)
- [GASDocumentation (Tranek)](https://github.com/tranek/GASDocumentation) — comprehensive community reference
- [Lyra Starter Game](https://docs.unrealengine.com/5.0/en-US/lyra-sample-game-in-unreal-engine/)
- [Gameplay Ability System Overview](https://docs.unrealengine.com/5.0/en-US/gameplay-ability-system-for-unreal-engine/)
