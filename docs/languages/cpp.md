# C++ for Unreal Engine 5

## Overview

C++ is the primary programming language for Unreal Engine 5 gameplay systems, engine extensions, and performance-critical code. UE5 extends standard C++ with its own macro system, garbage collector, and reflection framework (UObject system), which provides seamless Blueprint integration and editor tooling.

## UE5 C++ Essentials

### UObject System

All engine-integrated classes inherit from `UObject` or its subclasses:

| Base Class | Purpose |
|------------|---------|
| `UObject` | Base of all UE reflectable objects |
| `AActor` | Things placed in the world |
| `UActorComponent` | Components attached to actors |
| `USceneComponent` | Components with transforms |
| `UPrimitiveComponent` | Renderable/collidable components |
| `UGameInstance` | Per-game-session singleton |
| `UGameMode` | Game rules and state |
| `ACharacter` | Pawn with CharacterMovementComponent |
| `APlayerController` | Handles input and camera |

### Core Macros

| Macro | Purpose |
|-------|---------|
| `UCLASS()` | Marks a class for UE reflection |
| `UPROPERTY()` | Marks a variable for reflection, GC, Blueprint, editor |
| `UFUNCTION()` | Marks a function for reflection, Blueprint, RPC |
| `USTRUCT()` | Marks a struct for reflection |
| `UENUM()` | Marks an enum for reflection |
| `GENERATED_BODY()` | Required boilerplate in every reflected class/struct |

### Example Actor

```cpp
// MyActor.h
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyActor.generated.h"

UCLASS()
class MYGAME_API AMyActor : public AActor
{
    GENERATED_BODY()

public:
    AMyActor();

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Config")
    float Speed = 300.f;

    UFUNCTION(BlueprintCallable, Category = "Actions")
    void ActivateEffect();

protected:
    virtual void BeginPlay() override;
    virtual void Tick(float DeltaTime) override;

private:
    UPROPERTY()
    UStaticMeshComponent* MeshComponent;
};
```

```cpp
// MyActor.cpp
#include "MyActor.h"

AMyActor::AMyActor()
{
    PrimaryActorTick.bCanEverTick = true;
    MeshComponent = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
    RootComponent = MeshComponent;
}

void AMyActor::BeginPlay()
{
    Super::BeginPlay();
    UE_LOG(LogTemp, Log, TEXT("MyActor started: %s"), *GetName());
}

void AMyActor::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);
    AddActorLocalOffset(FVector(Speed * DeltaTime, 0.f, 0.f));
}

void AMyActor::ActivateEffect()
{
    // Effect logic here
}
```

## UPROPERTY Specifiers

| Specifier | Description |
|-----------|-------------|
| `EditAnywhere` | Editable in editor (archetype + instance) |
| `EditDefaultsOnly` | Editable only on class defaults |
| `EditInstanceOnly` | Editable only on level instances |
| `BlueprintReadWrite` | Read/write from Blueprint |
| `BlueprintReadOnly` | Read-only from Blueprint |
| `VisibleAnywhere` | Shown in editor but not editable |
| `Replicated` | Replicated to clients in multiplayer |
| `ReplicatedUsing=FuncName` | Replicated with a RepNotify callback |
| `Category="Name"` | Groups property in editor |
| `Transient` | Not saved to disk |

## UFUNCTION Specifiers

| Specifier | Description |
|-----------|-------------|
| `BlueprintCallable` | Callable from Blueprint |
| `BlueprintImplementableEvent` | Implemented in Blueprint |
| `BlueprintNativeEvent` | Has C++ default, overridable in Blueprint |
| `CallInEditor` | Shows "Call in Editor" button in Details panel |
| `Server` | RPC called on client, executes on server |
| `Client` | RPC called on server, executes on owning client |
| `NetMulticast` | RPC broadcast to all clients |
| `Reliable` | Guaranteed RPC delivery |
| `Unreliable` | Best-effort RPC delivery |

## Memory Management

UE5 uses a **garbage collector** for `UObject`-based types. Follow these rules:

```cpp
// ✅ Correct: Raw pointer for UPROPERTY-tracked references
UPROPERTY()
UMyComponent* MyComp;

// ✅ Correct: TWeakObjectPtr for non-owning references
TWeakObjectPtr<AActor> WeakRef;

// ✅ Correct: TObjectPtr (UE5 preferred over raw pointer)
UPROPERTY()
TObjectPtr<UStaticMesh> Mesh;

// ❌ Wrong: Non-UPROPERTY raw pointer (GC can collect it)
UMyObject* DanglingPtr;  // Will become invalid!

// ✅ Non-UObject memory: TUniquePtr / TSharedPtr
TUniquePtr<FMyPlainStruct> OwnedData;
TSharedPtr<FSharedData> SharedData;
```

## Common UE5 Types

| Type | Description |
|------|-------------|
| `FString` | Heap-allocated string |
| `FName` | Hashed, immutable name (fast comparison) |
| `FText` | Localizable display text |
| `FVector` | 3D vector (float) |
| `FRotator` | Euler rotation (Pitch, Yaw, Roll) |
| `FQuat` | Quaternion |
| `FTransform` | Translation + Rotation + Scale |
| `TArray<T>` | Dynamic array |
| `TMap<K,V>` | Hash map |
| `TSet<T>` | Hash set |
| `TOptional<T>` | Optional value |
| `TSubclassOf<T>` | Class reference (for spawning) |

## Gameplay Framework

```
AGameMode          → Game rules; server-only
AGameState         → Replicated game state (scores, time)
APlayerController  → Per-player input; bridge to pawn
APlayerState       → Replicated per-player data (name, score)
APawn              → Possessable actor; receives input
ACharacter         → Pawn with movement, capsule, mesh
AHUD               → Per-player HUD (legacy; prefer UMG)
UUserWidget        → UMG UI Widget
```

## Enhanced Input System (UE5)

```cpp
// In PlayerController or Character
#include "EnhancedInputComponent.h"
#include "EnhancedInputSubsystems.h"

void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
    Super::SetupPlayerInputComponent(PlayerInputComponent);
    
    UEnhancedInputComponent* EIC = Cast<UEnhancedInputComponent>(PlayerInputComponent);
    if (EIC)
    {
        EIC->BindAction(MoveAction, ETriggerEvent::Triggered, this, &AMyCharacter::Move);
        EIC->BindAction(JumpAction, ETriggerEvent::Started, this, &ThisClass::Jump);
    }
}

void AMyCharacter::Move(const FInputActionValue& Value)
{
    FVector2D Input = Value.Get<FVector2D>();
    AddMovementInput(GetActorForwardVector(), Input.Y);
    AddMovementInput(GetActorRightVector(), Input.X);
}
```

## Building and Compiling

### Live Coding

- Press **Ctrl+Alt+F11** in the editor for in-editor Hot Reload (Live Coding).
- Works for function body changes; requires full recompile for UPROPERTY/UFUNCTION changes.

### Build from Command Line

```bash
# Windows: Using UnrealBuildTool
Engine\Build\BatchFiles\Build.bat MyProjectEditor Win64 Development "C:\Projects\MyProject\MyProject.uproject" -WaitMutex -FromMsBuild

# Mac/Linux
Engine/Build/BatchFiles/Mac/Build.sh MyProjectEditor Mac Development "/path/to/MyProject.uproject"
```

### Build Targets

| Target | Suffix | Use |
|--------|--------|-----|
| Development | None | Standard development |
| Debug | Debug | Full debug symbols, slowest |
| DebugGame | DebugGame | Debug symbols for game code only |
| Shipping | Shipping | Optimized release build |

## Useful Macros and Logging

```cpp
// Logging
UE_LOG(LogTemp, Log, TEXT("Value: %f"), MyFloat);
UE_LOG(LogTemp, Warning, TEXT("Warning: %s"), *MyString);
UE_LOG(LogTemp, Error, TEXT("Error occurred"));

// Screen debug messages
GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Red, TEXT("Hello"));

// Assert macros
check(Pointer != nullptr);       // Fatal crash if false
ensure(Value > 0);               // Log and continue if false
checkNoEntry();                  // Marks unreachable code
```

## AI / MCP Integration

C++ code in UE5 can integrate with AI agents through:
- **Unreal Python** — agents script the Editor; Python calls C++ via reflected `UFUNCTION`s.
- **REST APIs** — expose game state via HTTP using `FHttpModule`.
- **Blueprints** — expose C++ to Blueprint for agent-friendly node-based control.

See [docs/mcp-servers/overview.md](../mcp-servers/overview.md) for MCP tools.

## References

- [UE5 C++ Programming Guide](https://docs.unrealengine.com/5.0/en-US/programming-with-cplusplus-in-unreal-engine/)
- [UPROPERTY Specifiers](https://docs.unrealengine.com/5.0/en-US/unreal-engine-uproperty-specifiers/)
- [UFUNCTION Specifiers](https://docs.unrealengine.com/5.0/en-US/ufunctions-in-unreal-engine/)
- [Enhanced Input System](https://docs.unrealengine.com/5.0/en-US/enhanced-input-in-unreal-engine/)
