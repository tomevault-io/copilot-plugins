## cplus

> > **Destination:** `.windsurf/rules/architecture.md`

# Unreal Engine Architecture

> **Destination:** `.windsurf/rules/architecture.md`

---
trigger: always_on
description: Unreal Engine 5.7 C++ architecture for cplus project
---

# Architecture

> Unreal Engine 5.7 C++ game architecture

## Project Structure

```
cplus/
├── Source/
│   └── cplus/
│       ├── cplus.Build.cs           # Module dependencies
│       ├── cplus.h/cpp              # Module definition
│       │
│       ├── QuestSystem/             # Quest system (our new system)
│       │   ├── Core/
│       │   │   ├── QuestDefinition.h/cpp
│       │   │   ├── QuestTask.h/cpp
│       │   │   └── QuestSubsystem.h/cpp
│       │   ├── Components/
│       │   │   ├── QuestTrackerComponent.h/cpp
│       │   │   ├── QuestGiverComponent.h/cpp
│       │   │   └── QuestTargetComponent.h/cpp
│       │   ├── Tasks/
│       │   │   ├── QuestTask_Kill.h/cpp
│       │   │   ├── QuestTask_Collect.h/cpp
│       │   │   └── QuestTask_Location.h/cpp
│       │   └── Interfaces/
│       │       ├── QuestInteractable.h/cpp
│       │       └── QuestKillable.h/cpp
│       │
│       ├── Variant_Shooter/         # Shooter game variant
│       │   ├── ShooterCharacter.h/cpp
│       │   ├── ShooterGameMode.h/cpp
│       │   ├── AI/
│       │   │   ├── ShooterNPC.h/cpp
│       │   │   └── ShooterAIController.h/cpp
│       │   ├── Weapons/
│       │   │   ├── ShooterWeapon.h/cpp
│       │   │   └── ShooterProjectile.h/cpp
│       │   └── UI/
│       │   └── ShooterUI.h/cpp
│       │
│       └── Variant_Horror/          # Horror game variant
│           ├── HorrorCharacter.h/cpp
│           └── HorrorGameMode.h/cpp
│
├── Content/                         # Unreal assets
│   ├── Blueprints/
│   ├── Maps/
│   ├── Materials/
│   ├── Meshes/
│   └── Textures/
│
├── Config/                          # Configuration files
├── Intermediate/                    # Build artifacts (gitignored)
├── Binaries/                        # Compiled DLLs/EXEs
└── Saved/                           # Saved data (gitignored)
```

---

## Core Hierarchy

### Unreal Engine Base Classes

```
UObject (Unreal root)
├── AActor (Placeable in world)
│   ├── ACharacter (Movable character)
│   │   ├── AcplusCharacter (Base first-person)
│   │   │   ├── AShooterCharacter (Player)
│   │   │   └── AShooterNPC (AI enemies)
│   │   └── AHorrorCharacter
│   ├── AGameModeBase
│   │   ├── AShooterGameMode
│   │   └── AHorrorGameMode
│   └── APlayerController
│       ├── AShooterPlayerController
│       └── AHorrorPlayerController
│
├── UActorComponent (Attachable to actors)
│   ├── UQuestTrackerComponent
│   ├── UQuestGiverComponent
│   └── UQuestTargetComponent
│
├── UGameInstanceSubsystem
│   └── UQuestSubsystem
│
└── UPrimaryDataAsset
    └── UQuestDefinition
```

---

## Design Patterns

| Pattern | Usage | Unreal Implementation |
|---------|-------|----------------------|
| **Component** | Actor functionality | UActorComponent |
| **Subsystem** | Global managers | UGameInstanceSubsystem |
| **Interface** | Polymorphic behavior | IInterface (UINTERFACE) |
| **DataAsset** | Data-driven design | UPrimaryDataAsset |
| **Delegate** | Event system | DECLARE_DYNAMIC_MULTICAST_DELEGATE |
| **Gameplay Tags** | Categorization | FGameplayTagContainer |

---

## Module Dependencies

### cplus.Build.cs

```csharp
PublicDependencyModuleNames.AddRange(new string[] 
{
    "Core",              // Core Unreal types
    "CoreUObject",       // UObject system
    "Engine",            // Game engine
    "InputCore",         // Input handling
    "EnhancedInput",     // Enhanced input system
    "AIModule",          // AI system
    "StateTreeModule",   // State Tree AI
    "GameplayTags",      // Quest categorization
    "UMG",               // UI widgets
    "Slate"              // UI framework
});
```

---

## System Architecture

### Quest System Flow

```
Player (AShooterCharacter)
└── UQuestTrackerComponent
    │
    ├─ AcceptQuest() ──────────► UQuestSubsystem
    │                             │
    │                             ├─ ActiveQuests (TMap)
    │                             ├─ OnQuestStarted (Delegate)
    │                             └─ OnQuestCompleted (Delegate)
    │
    └─ GetActiveQuests() ◄────────┘

NPC (AShooterNPC)
├── UQuestGiverComponent ──────► Offers quests to player
└── UQuestTargetComponent ─────► Notifies quest system on kill/interact
```

### Event Flow

```
1. Player interacts with NPC (IQuestInteractable)
2. UQuestGiverComponent::OfferQuest()
3. Player accepts quest
4. UQuestTrackerComponent::AcceptQuest()
5. UQuestSubsystem::AcceptQuest()
6. Quest added to ActiveQuests
7. OnQuestStarted.Broadcast()
8. UI updates via delegate binding
```

---

## Unreal Engine Conventions

### Naming Prefixes

| Prefix | Type | Example |
|--------|------|---------|
| `A` | Actor | `AShooterCharacter` |
| `U` | UObject, Component, Widget | `UQuestTrackerComponent` |
| `F` | Struct, non-UObject | `FQuestObjective` |
| `E` | Enum | `EQuestState` |
| `I` | Interface | `IQuestInteractable` |
| `T` | Template | `TArray`, `TMap` |

### Member Prefixes

| Prefix | Type | Example |
|--------|------|---------|
| `b` | bool | `bIsMandatory` |
| `m_` | Member variable (optional) | `m_Position` |
| No prefix | UPROPERTY | `CurrentCount` |

---

## Build Configurations

| Config | Optimization | Debug Info | Usage |
|--------|-------------|------------|-------|
| **DebugGame** | None | Full | Development with debugging |
| **Development** | Some | Partial | Daily development |
| **Shipping** | Full | None | Final release |
| **Test** | Full | Some | QA testing |

---

## Compilation Workflow

### Live Coding (Hot Reload)

- **CTRL+ALT+F11** - Compile .cpp changes
- **Editor Restart** - Required for .h changes
- **Limitations:** Cannot add new UCLASS, UPROPERTY, UFUNCTION

### Full Rebuild

```powershell
# Via Unreal Build Tool
& "C:\ue5.7\UE_5.7\Engine\Build\BatchFiles\Build.bat" cplusEditor Win64 Development "C:\Users\Calle\Documents\Unreal Projects\cplus\cplus.uproject"

# Via Visual Studio
# Open cplus.sln → Build Solution (Ctrl+Shift+B)
```

---

## Asset Pipeline

### Content Browser Organization

```
Content/
├── Blueprints/
│   ├── Characters/
│   ├── AI/
│   └── Quests/
│       └── BP_QuestDefinition_KillBandits
├── Maps/
│   ├── Shooter/
│   └── Horror/
├── UI/
│   └── Widgets/
└── Data/
    └── Quests/
        └── DA_Quest_KillBandits (UQuestDefinition)
```

---

## Performance Considerations

### Tick vs Event-Driven

```cpp
// ❌ BAD - Unnecessary tick
UPROPERTY()
bool bCanEverTick = true;

void Tick(float DeltaTime) {
    // Runs every frame - expensive!
}

// ✅ GOOD - Event-driven
UPROPERTY(BlueprintAssignable)
FOnQuestCompleted OnQuestCompleted;

void CompleteQuest() {
    OnQuestCompleted.Broadcast(QuestID, Rewards);
}
```

### Memory Management

- Use `TObjectPtr<>` for UPROPERTY references
- Use `TArray<>` instead of std::vector
- Use `FString` instead of std::string
- Avoid raw pointers for UObjects

---

## Debugging Tools

| Tool | Purpose | Access |
|------|---------|--------|
| **Output Log** | Console logging | Window → Developer Tools → Output Log |
| **Visual Logger** | Visual debugging | ' key (apostrophe) |
| **Blueprint Debugger** | Blueprint debugging | Alt+Shift+D |
| **Stat Commands** | Performance profiling | ~ → stat fps, stat unit |
| **Visual Studio** | C++ debugging | F5 to attach |

---

## Best Practices

### DO

- ✅ Use UPROPERTY() for garbage collection
- ✅ Use UFUNCTION() for Blueprint exposure
- ✅ Use GameplayTags for categorization
- ✅ Use Subsystems for global managers
- ✅ Use Components for modular functionality
- ✅ Use DataAssets for data-driven design
- ✅ Use Delegates for event communication

### DON'T

- ❌ Use raw new/delete (use NewObject, ConstructObject)
- ❌ Store UObject* without UPROPERTY (garbage collection!)
- ❌ Use static variables for UObjects
- ❌ Tick every frame unnecessarily
- ❌ Use std::string/std::vector (use FString/TArray)
- ❌ Hardcode data (use DataAssets)

---

## Quest System Integration Points

### Player Character

```cpp
UCLASS()
class AShooterCharacter : public AcplusCharacter
{
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
    UQuestTrackerComponent* QuestTracker;
};
```

### NPC

```cpp
UCLASS()
class AShooterNPC : public AcplusCharacter, public IQuestKillable
{
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
    UQuestGiverComponent* QuestGiver;
    
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
    UQuestTargetComponent* QuestTarget;
    
    // IQuestKillable
    virtual void OnKilledForQuest_Implementation(AActor* Killer) override;
};
```

---

## References

- [Unreal Engine Documentation](https://docs.unrealengine.com/)
- [Unreal C++ Coding Standard](https://docs.unrealengine.com/5.7/en-US/epic-cplusplus-coding-standard-for-unreal-engine/)
- [Gameplay Framework](https://docs.unrealengine.com/5.7/en-US/gameplay-framework-in-unreal-engine/)
- [Subsystems](https://docs.unrealengine.com/5.7/en-US/programming-subsystems-in-unreal-engine/)
- [Gameplay Tags](https://docs.unrealengine.com/5.7/en-US/using-gameplay-tags-in-unreal-engine/)

---

**Version:** 1.0  
**Last Updated:** 2026-01-13  
**Project:** cplus (Unreal Engine 5.7)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/denker-systems) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
