# Gameplay Ability System (GAS)

## Overview

The **Gameplay Ability System (GAS)** is a production-ready UE5 plugin used to implement complex ability frameworks in games. It provides a unified way to define abilities, apply attribute-modifying effects, manage gameplay state via tags, and trigger cosmetic feedback — all with full support for replication in multiplayer games. GAS is used in Epic's own titles (Paragon, Fortnite, Lyra) and is the recommended ability architecture for serious UE5 projects.

## Core Concepts

### Ability System Component (ASC)

The `UAbilitySystemComponent` (ASC) is the central hub. It lives on an Actor (usually a Character or PlayerState) and manages:

- Granted abilities and their activation state
- Active gameplay effects and their duration/stacks
- Attribute sets and their current/base values
- Gameplay tags currently applied to the owner
- Gameplay event dispatching and listening

```cpp
// Typical placement: on ACharacter or APlayerState
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Abilities")
TObjectPtr<UAbilitySystemComponent> AbilitySystemComponent;
```

> **Multiplayer tip:** For player characters, place the ASC on `APlayerState` so it persists across respawns. For AI, placing it on the `ACharacter` is fine.

### Gameplay Abilities (UGameplayAbility)

`UGameplayAbility` represents a discrete action (e.g., "Fireball", "Dash", "Melee Attack"). Each ability:

- Has **Ability Tags** that identify it.
- Has **Cancel Abilities with Tag / Block Abilities with Tag** to control concurrency.
- Supports **Costs** (spend mana via a Gameplay Effect) and **Cooldowns** (block re-activation via a tag duration).
- Can run **Ability Tasks** for async operations (wait for animation, wait for input, etc.).
- Is fully replicable with `Instanced Per Actor` or `Instanced Per Execution` policies.

```cpp
UCLASS()
class MYGAME_API UGA_Fireball : public UGameplayAbility
{
    GENERATED_BODY()

public:
    UGA_Fireball();

    virtual bool CanActivateAbility(
        const FGameplayAbilitySpecHandle Handle,
        const FGameplayAbilityActorInfo* ActorInfo,
        const FGameplayTagContainer* SourceTags = nullptr,
        const FGameplayTagContainer* TargetTags = nullptr,
        FGameplayTagContainer* OptionalRelevantTags = nullptr) const override;

    virtual void ActivateAbility(
        const FGameplayAbilitySpecHandle Handle,
        const FGameplayAbilityActorInfo* ActorInfo,
        const FGameplayAbilityActivationInfo ActivationInfo,
        const FGameplayEventData* TriggerEventData) override;

protected:
    UPROPERTY(EditDefaultsOnly, Category = "Fireball")
    TSubclassOf<UGameplayEffect> DamageEffectClass;

    UPROPERTY(EditDefaultsOnly, Category = "Fireball")
    TObjectPtr<UAnimMontage> CastMontage;
};
```

### Attribute Sets (UAttributeSet)

`UAttributeSet` holds the numeric values (health, mana, stamina, damage, etc.) that Gameplay Effects modify.

```cpp
UCLASS()
class MYGAME_API UBaseAttributeSet : public UAttributeSet
{
    GENERATED_BODY()

public:
    // Creates the FGameplayAttribute accessor for use in effects/queries
    ATTRIBUTE_ACCESSORS(UBaseAttributeSet, Health)
    ATTRIBUTE_ACCESSORS(UBaseAttributeSet, MaxHealth)
    ATTRIBUTE_ACCESSORS(UBaseAttributeSet, Mana)
    ATTRIBUTE_ACCESSORS(UBaseAttributeSet, MaxMana)
    ATTRIBUTE_ACCESSORS(UBaseAttributeSet, Damage)  // Transient meta-attribute

    virtual void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue) override;
    virtual void PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data) override;

    // GetLifetimeReplicatedProps + OnRep_ functions required for replication
    virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;

protected:
    UPROPERTY(BlueprintReadOnly, ReplicatedUsing=OnRep_Health, Category="Attributes")
    FGameplayAttributeData Health;

    UPROPERTY(BlueprintReadOnly, ReplicatedUsing=OnRep_MaxHealth, Category="Attributes")
    FGameplayAttributeData MaxHealth;

    UPROPERTY(BlueprintReadOnly, ReplicatedUsing=OnRep_Mana, Category="Attributes")
    FGameplayAttributeData Mana;

    UPROPERTY(BlueprintReadOnly, ReplicatedUsing=OnRep_MaxMana, Category="Attributes")
    FGameplayAttributeData MaxMana;

    UPROPERTY(BlueprintReadOnly, Category="Attributes")
    FGameplayAttributeData Damage;

    UFUNCTION() void OnRep_Health(const FGameplayAttributeData& OldHealth);
    UFUNCTION() void OnRep_MaxHealth(const FGameplayAttributeData& OldMaxHealth);
    UFUNCTION() void OnRep_Mana(const FGameplayAttributeData& OldMana);
    UFUNCTION() void OnRep_MaxMana(const FGameplayAttributeData& OldMaxMana);
};
```

> Use the `ATTRIBUTE_ACCESSORS` macro (defined in `AttributeSet.h`) to generate `Get`, `Set`, and `Init` accessors for each attribute.

### Gameplay Effects (UGameplayEffect)

`UGameplayEffect` (GE) is a **data-only** Blueprint asset (no custom C++ logic) that describes how to modify attributes and apply/remove tags. GEs have three duration policies:

| Duration Policy | Behavior |
|----------------|----------|
| **Instant** | Applies once and is gone (e.g., deal damage) |
| **Duration** | Active for a set time, then removed (e.g., a 5-second slow) |
| **Infinite** | Active until explicitly removed (e.g., a permanent buff) |

**Key GE properties:**

- **Modifiers** — specify the Attribute, Operation (Add/Multiply/Override), and Magnitude.
- **Executions** — custom `UGameplayEffectExecutionCalculation` for complex formulas.
- **Conditional Effects** — apply additional GEs if certain tags are present.
- **Period** — makes a Duration/Infinite GE apply its modifiers on a timer (e.g., poison ticking).
- **Stacking** — aggregate multiple applications into a single effect with a stack count.
- **Granted Abilities** — GEs can grant abilities while they are active.
- **Granted Tags** — GEs apply tags to the ASC owner for the duration of the effect.
- **Removal Tags** — GEs remove tag-based effects when applied.

### Gameplay Tags

Gameplay Tags are hierarchical string identifiers (e.g., `Ability.Fireball`, `State.Dead`, `Effect.Burning`) that serve as a universal state vocabulary in GAS.

**Common tag uses:**

| Use | Example |
|-----|---------|
| Ability identification | `Ability.Attack.Melee` |
| Ability activation conditions | `Ability.Activation.CanActivate` |
| State tracking | `State.Stunned`, `State.Invulnerable` |
| Blocking/cancelling abilities | `Effect.CC.Root` blocks `Ability.Movement` |
| Gameplay Cue triggers | `GameplayCue.Impact.Fire` |
| Event payloads | `Event.Damage.Physical` |

Define tags in `DefaultGameplayTags.ini` or via `Edit > Project Settings > GameplayTags`:

```ini
; Config/DefaultGameplayTags.ini
[/Script/GameplayTags.GameplayTagsSettings]
GameplayTagList=(Tag="Ability.Fireball",DevComment="")
GameplayTagList=(Tag="State.Dead",DevComment="")
GameplayTagList=(Tag="Effect.Burning",DevComment="")
GameplayTagList=(Tag="GameplayCue.Impact.Fire",DevComment="")
```

### Gameplay Cues

`GameplayCue` is the cosmetic layer of GAS — sounds, particles, and camera shakes that are decoupled from gameplay logic and replicated efficiently via the ASC. Cues are triggered by tags under the `GameplayCue.` root.

```cpp
// Trigger a cue manually from a Gameplay Effect or from code
FGameplayCueParameters Params;
Params.Location = ImpactLocation;
Params.Normal = ImpactNormal;
AbilitySystemComponent->ExecuteGameplayCue(
    FGameplayTag::RequestGameplayTag(TEXT("GameplayCue.Impact.Fire")),
    Params
);
```

Implement the cue response by creating a `UGameplayCueNotify_Static` (fire-and-forget) or `UGameplayCueNotify_Actor` (spawned actor with lifetime):

| Cue Class | When to Use |
|-----------|-------------|
| `UGameplayCueNotify_Static` | One-shot effects: impact sparks, screen flash |
| `UGameplayCueNotify_Actor` | Looping/persistent effects: burning aura, shields |

### Ability Tasks

Ability Tasks (`UAbilityTask`) allow abilities to yield execution and wait for asynchronous events without blocking the game thread:

| Task Class | Waits For |
|-----------|-----------|
| `UAbilityTask_PlayMontageAndWait` | Animation montage to end or be interrupted |
| `UAbilityTask_WaitGameplayEvent` | A gameplay event tag to fire |
| `UAbilityTask_WaitInputPress` | Player to press the ability input again |
| `UAbilityTask_WaitDelay` | A time delay |
| `UAbilityTask_SpawnActor` | Actor spawn confirmation |
| `UAbilityTask_WaitTargetData` | Target selection (AGameplayAbilityTargetActor) |

```cpp
void UGA_Fireball::ActivateAbility(...)
{
    // 1. Play cast animation and wait for it to finish
    UAbilityTask_PlayMontageAndWait* MontageTask =
        UAbilityTask_PlayMontageAndWait::CreatePlayMontageAndWaitProxy(
            this, NAME_None, CastMontage);

    MontageTask->OnCompleted.AddDynamic(this, &UGA_Fireball::OnMontageCompleted);
    MontageTask->OnCancelled.AddDynamic(this, &UGA_Fireball::EndAbilityCallback);
    MontageTask->OnInterrupted.AddDynamic(this, &UGA_Fireball::EndAbilityCallback);
    MontageTask->ReadyForActivation();
}

void UGA_Fireball::OnMontageCompleted()
{
    // 2. Apply damage effect to target
    FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(DamageEffectClass);
    ApplyGameplayEffectSpecToTarget(GetCurrentAbilitySpecHandle(),
        CurrentActorInfo, CurrentActivationInfo, SpecHandle, TargetHandle);

    EndAbility(GetCurrentAbilitySpecHandle(), CurrentActorInfo,
        CurrentActivationInfo, true, false);
}
```

## Project Setup

### 1. Enable the Plugin

```
Edit > Plugins > search "Gameplay Abilities" → enable
```

Restart the editor when prompted.

### 2. Add Module Dependencies

```csharp
// Source/MyProject/MyProject.Build.cs
PublicDependencyModuleNames.AddRange(new string[]
{
    "Core", "CoreUObject", "Engine", "InputCore",
    "GameplayAbilities",
    "GameplayTags",
    "GameplayTasks"
});
```

### 3. Implement the IAbilitySystemInterface

```cpp
// MyCharacter.h
#include "AbilitySystemInterface.h"

UCLASS()
class AMyCharacter : public ACharacter, public IAbilitySystemInterface
{
    GENERATED_BODY()

public:
    virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;

protected:
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Abilities")
    TObjectPtr<UAbilitySystemComponent> AbilitySystemComponent;

    UPROPERTY()
    TObjectPtr<UBaseAttributeSet> AttributeSet;
};
```

### 4. Initialize on PossessedBy / OnRep_PlayerState

```cpp
// Server
void AMyCharacter::PossessedBy(AController* NewController)
{
    Super::PossessedBy(NewController);
    // Initialize ASC with owner and avatar actor
    AbilitySystemComponent->InitAbilityActorInfo(this, this);
    // Grant starting abilities
    for (const TSubclassOf<UGameplayAbility>& Ability : StartingAbilities)
    {
        FGameplayAbilitySpec Spec(Ability, 1);
        AbilitySystemComponent->GiveAbility(Spec);
    }
}

// Client (for PlayerState-hosted ASC)
void AMyCharacter::OnRep_PlayerState()
{
    Super::OnRep_PlayerState();
    AbilitySystemComponent->InitAbilityActorInfo(GetPlayerState(), this);
}
```

## Activation & Input Binding

### Activating by Class

```cpp
AbilitySystemComponent->TryActivateAbilityByClass(UGA_Fireball::StaticClass());
```

### Activating by Tag

```cpp
FGameplayTagContainer AbilityTags;
AbilityTags.AddTag(FGameplayTag::RequestGameplayTag("Ability.Fireball"));
AbilitySystemComponent->TryActivateAbilitiesByTag(AbilityTags);
```

### Enhanced Input Binding

```cpp
// In SetupPlayerInputComponent:
EIC->BindAction(FireballAction, ETriggerEvent::Started, this, &AMyCharacter::Input_Fireball);

void AMyCharacter::Input_Fireball()
{
    AbilitySystemComponent->TryActivateAbilityByClass(UGA_Fireball::StaticClass());
}
```

### Gameplay Event Activation

```cpp
FGameplayEventData Payload;
Payload.Target = TargetActor;
UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(
    CharacterActor,
    FGameplayTag::RequestGameplayTag("Event.Ability.Fireball"),
    Payload
);
```

## Monitoring Attributes

```cpp
// React to attribute changes (e.g., update health bar)
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(
    UBaseAttributeSet::GetHealthAttribute()
).AddUObject(this, &AMyHUD::OnHealthChanged);

void AMyHUD::OnHealthChanged(const FOnAttributeChangeData& Data)
{
    HealthBar->SetPercent(Data.NewValue / AttributeSet->GetMaxHealth());
}
```

## Gameplay Tag Queries

```cpp
// Check if the owner has a specific tag
bool bIsStunned = AbilitySystemComponent->HasMatchingGameplayTag(
    FGameplayTag::RequestGameplayTag("State.Stunned")
);

// Listen for tag changes
AbilitySystemComponent->RegisterGameplayTagEvent(
    FGameplayTag::RequestGameplayTag("State.Dead"),
    EGameplayTagEventType::NewOrRemoved
).AddUObject(this, &AMyCharacter::OnDeadTagChanged);
```

## Applying Gameplay Effects Programmatically

```cpp
// Create an effect context and spec, then apply
FGameplayEffectContextHandle ContextHandle =
    AbilitySystemComponent->MakeEffectContext();
ContextHandle.AddSourceObject(this);

FGameplayEffectSpecHandle SpecHandle =
    AbilitySystemComponent->MakeOutgoingSpec(
        UGE_FireDamage::StaticClass(), Level, ContextHandle);

if (SpecHandle.IsValid())
{
    SpecHandle.Data->SetSetByCallerMagnitude(
        FGameplayTag::RequestGameplayTag("Data.Damage"), DamageAmount);

    AbilitySystemComponent->ApplyGameplayEffectSpecToSelf(*SpecHandle.Data.Get());
}
```

## Replication

GAS handles its own replication. Key configuration points:

| Setting | Recommendation |
|---------|---------------|
| ASC replication mode | `Full` for single-player/co-op; `Mixed` for competitive multiplayer; `Minimal` for large-scale AI |
| Attribute replication | Must add `OnRep_` functions + `GetLifetimeReplicatedProps` |
| Ability activation replication | Controlled by ability's `ReplicationPolicy` |
| Gameplay Cues | Replicated via ASC — do not manually RPC |

```cpp
// In ASC setup (server)
AbilitySystemComponent->SetReplicationMode(EGameplayEffectReplicationMode::Mixed);
```

## Debugging

| Tool | How to Access |
|------|--------------|
| `showdebug abilitysystem` | Console command — overlay showing active effects, tags, abilities |
| Gameplay Debugger | `'` key during PIE — shows GAS info per actor |
| Gameplay Effect Log | `AbilitySystem.Debug.PrintAttributes` console command |
| GAS Logging | `AbilitySystem.Verbose 1` enables verbose GAS logging |
| Tag Inspector | `Edit > Project Settings > GameplayTags` — view all tags |

## Common Pitfalls

| Pitfall | Solution |
|---------|---------|
| `InitAbilityActorInfo` not called | Must be called on both server and client after possession |
| Attributes not replicating | Add `OnRep_` + `GetLifetimeReplicatedProps` for every replicated attribute |
| Abilities not activating | Check that the ability is granted, tags don't block it, and cost/cooldown pass |
| Gameplay Effect not applying | Verify the attribute name matches; check modifier operation type |
| Cues firing on server | Cues should fire via ASC — they auto-replicate, never fire them via server RPC directly |
| `EndAbility` not called | Always call `EndAbility` in all code paths including tasks' cancel/interrupt callbacks |

## AI / MCP Integration

GAS exposes rich introspection capabilities through the ASC, making it well-suited for AI agent control. Agents can:

- Query active abilities, effects, and attribute values via Python through the UE5 Remote Control API.
- Trigger ability activations and gameplay events programmatically.
- Monitor gameplay tag changes and attribute modifications as feedback signals.

See [docs/mcp-servers/gameplay-ability-system.md](../mcp-servers/gameplay-ability-system.md) for a dedicated MCP server guide for GAS workflows.

## References

- [GAS Documentation (Tranek's GASDocumentation)](https://github.com/tranek/GASDocumentation) — the most complete community reference for GAS
- [Lyra Starter Game](https://docs.unrealengine.com/5.0/en-US/lyra-sample-game-in-unreal-engine/) — Epic's production GAS reference project
- [Gameplay Ability System Overview](https://docs.unrealengine.com/5.0/en-US/gameplay-ability-system-for-unreal-engine/)
- [GameplayEffect Reference](https://docs.unrealengine.com/5.0/en-US/gameplay-effects-for-the-gameplay-ability-system-in-unreal-engine/)
- [AttributeSet Reference](https://docs.unrealengine.com/5.0/en-US/gameplay-attributes-and-attribute-sets-for-the-gameplay-ability-system-in-unreal-engine/)
