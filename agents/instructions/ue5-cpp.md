# Agent Instructions: UE5 C++ Development

Copy this file into a UE5 C++ project as `AGENT.md` or add its contents to `.cursorrules` to give AI coding agents full context about UE5 C++ conventions and patterns.

---

## Language: C++ for Unreal Engine 5

**Engine Version**: UE5.x
**Build System**: Unreal Build Tool (UBT)
**Reflection System**: UObject / UHT (Unreal Header Tool)
**Standard**: C++17 with UE5 macro extensions

---

## File Structure

```
Source/
└── [ProjectName]/
    ├── [ProjectName].Build.cs    # Module dependencies
    ├── Public/                   # .h header files
    └── Private/                  # .cpp source files
```

---

## Class Naming Conventions

| Prefix | Type | Example |
|--------|------|---------|
| `A` | Actor | `AMyCharacter` |
| `U` | UObject (component, subsystem, etc.) | `UMyComponent` |
| `F` | Plain struct (no UObject) | `FMyStruct` |
| `E` | Enum | `EMyState` |
| `I` | Interface | `IMyInterface` |
| `T` | Template class | `TMyContainer<T>` |
| `G` | Global variable | `GEngine` |

---

## Required Macros

Every reflected class must have:

```cpp
// .h
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyActor.generated.h"  // ALWAYS last include

UCLASS()
class MYGAME_API AMyActor : public AActor
{
    GENERATED_BODY()
    // ...
};
```

---

## UPROPERTY Rules

```cpp
// Editable in editor + Blueprint read/write
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Config")
float Speed = 300.f;

// Exposed to Blueprint but not editor
UPROPERTY(BlueprintReadOnly)
int32 Health;

// Replicated (multiplayer)
UPROPERTY(Replicated)
float ReplicatedValue;

// With RepNotify
UPROPERTY(ReplicatedUsing=OnRep_State)
EMyState State;

UFUNCTION()
void OnRep_State();
```

---

## UFUNCTION Rules

```cpp
// Callable from Blueprint
UFUNCTION(BlueprintCallable, Category="Actions")
void Activate();

// Implemented in Blueprint (define in C++, override in BP)
UFUNCTION(BlueprintImplementableEvent)
void OnActivated();

// C++ default + Blueprint override
UFUNCTION(BlueprintNativeEvent)
void OnDamaged(float Amount);
virtual void OnDamaged_Implementation(float Amount);  // Required C++ impl

// Editor-only button
UFUNCTION(CallInEditor)
void DebugSpawn();

// Server RPC
UFUNCTION(Server, Reliable)
void Server_Action();
void Server_Action_Implementation();  // Required implementation
```

---

## Memory Management Rules

```cpp
// ✅ UObject pointers MUST be UPROPERTY to avoid GC
UPROPERTY()
UMyComponent* Comp;

// ✅ UE5 preferred: TObjectPtr instead of raw pointer
UPROPERTY()
TObjectPtr<UStaticMesh> Mesh;

// ✅ Weak reference (does not prevent GC)
TWeakObjectPtr<AActor> WeakRef;

// ✅ Non-UObject heap memory
TUniquePtr<FMyData> Data;
TSharedPtr<FSharedResource> Shared;

// ❌ NEVER: raw UObject pointer without UPROPERTY
UMyThing* Dangerous;  // Will become invalid!
```

---

## Component Creation Pattern

```cpp
// In constructor
AMyActor::AMyActor()
{
    // Create components with CreateDefaultSubobject
    MeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
    RootComponent = MeshComp;

    CollisionComp = CreateDefaultSubobject<UCapsuleComponent>(TEXT("Capsule"));
    CollisionComp->SetupAttachment(RootComponent);

    // Enable tick only if needed
    PrimaryActorTick.bCanEverTick = true;
}
```

---

## Common Patterns

### Cast (type-safe downcasting)

```cpp
// Check and cast
if (AMyCharacter* Char = Cast<AMyCharacter>(OtherActor))
{
    Char->TakeDamage(10.f);
}

// Fatal crash if wrong type (use sparingly)
AMyCharacter* Char = CastChecked<AMyCharacter>(OtherActor);
```

### Timer

```cpp
// In .h
FTimerHandle SpawnTimer;

// In .cpp
GetWorldTimerManager().SetTimer(
    SpawnTimer,
    this,
    &AMyActor::SpawnEnemy,
    5.0f,   // interval in seconds
    true    // loop
);
```

### Delegates

```cpp
// Declare in .h
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnHealthChanged, float, NewHealth);
UPROPERTY(BlueprintAssignable)
FOnHealthChanged OnHealthChanged;

// Fire in .cpp
OnHealthChanged.Broadcast(Health);
```

### Line Trace

```cpp
FHitResult HitResult;
FVector Start = GetActorLocation();
FVector End = Start + GetActorForwardVector() * 1000.f;

FCollisionQueryParams Params;
Params.AddIgnoredActor(this);

if (GetWorld()->LineTraceSingleByChannel(HitResult, Start, End, ECC_Visibility, Params))
{
    if (AActor* HitActor = HitResult.GetActor())
    {
        UE_LOG(LogTemp, Log, TEXT("Hit: %s"), *HitActor->GetName());
    }
}
```

---

## Build.cs Module Dependencies

```csharp
// Source/MyProject/MyProject.Build.cs
public class MyProject : ModuleRules
{
    public MyProject(ReadOnlyTargetRules Target) : base(Target)
    {
        PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
        
        PublicDependencyModuleNames.AddRange(new string[] {
            "Core", "CoreUObject", "Engine", "InputCore",
            "EnhancedInput"  // Add for Enhanced Input
        });
        
        PrivateDependencyModuleNames.AddRange(new string[] {
            "Niagara",       // For Niagara FX
            "UMG",           // For Widget Blueprints in C++
            "Http",          // For HTTP requests
            "Json",          // For JSON parsing
        });
    }
}
```

---

## Logging

```cpp
// Quick debug
UE_LOG(LogTemp, Log, TEXT("Value: %f"), MyValue);
UE_LOG(LogTemp, Warning, TEXT("Warning: %s"), *MyString);
UE_LOG(LogTemp, Error, TEXT("Error at %s"), *GetName());

// Custom log category (in .h)
DECLARE_LOG_CATEGORY_EXTERN(LogMyGame, Log, All);
// (in .cpp)
DEFINE_LOG_CATEGORY(LogMyGame);
// Usage
UE_LOG(LogMyGame, Log, TEXT("Custom log message"));

// Screen message
if (GEngine)
    GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Green, TEXT("Hello"));
```

---

## References

- [Full C++ Docs](../../docs/languages/cpp.md)
- [UE5 C++ Programming Guide](https://docs.unrealengine.com/5.0/en-US/programming-with-cplusplus-in-unreal-engine/)
- [UPROPERTY Specifiers](https://docs.unrealengine.com/5.0/en-US/unreal-engine-uproperty-specifiers/)
- [UFUNCTION Specifiers](https://docs.unrealengine.com/5.0/en-US/ufunctions-in-unreal-engine/)
