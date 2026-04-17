## wizardjam2-0

> Project: WizardJam (Portfolio Capstone Project)

Project: WizardJam (Portfolio Capstone Project)
Developer: Marcus Daley
Architecture Lead: Nick Penney AAA Coding Standards
Engine: Unreal Engine 5.4
Start Date: January 5, 2025
Current Phase: Post-Development Structural Refinement
Presentation Schedule: Working features demo every Tuesday & Thursday

---

## STANDARDS QUICK REFERENCE

Use this index to quickly find applicable standards for your current task.

### When Working on AI / Behavior Trees
- FBlackboardKeySelector Initialization (MANDATORY) - see line 50-101
- AI Perception Registration - see Lessons Learned line 157
- BT Node Checklist: AddObjectFilter + InitializeFromAsset
- Blackboard Key Initialization Checklist (LINKED SYSTEMS) - see Lessons Learned line 192

### When Working on C++ Actors / Components
- Property Exposure Rules (NO EditAnywhere) - see line 39-47
- Constructor Initialization Lists (NEVER header defaults) - see line 195-204
- Header Organization (forward declarations) - see environment.md Section 4

### When Working on UI / Widgets
- Delegate Binding Pattern - see line 205-209
- SlateCore Module Dependency - see Lessons Learned line 153
- NativeConstruct/NativeDestruct binding - see environment.md Section 8

### When Working on Build System
- Build.cs vs Target.cs - see Lessons Learned line 145
- Module Dependencies - see environment.md Section 3
- UHT generated.h Rule - see Lessons Learned line 155

### When Working on Cross-Class Communication
- Observer Pattern (Delegates) - see line 43, 148, 205-209
- Static Delegates - see Lessons Learned line 148
- No Direct GameMode Calls

### When Migrating Legacy to Modern Code
- Legacy to Modern Migration Guide - see MIGRATION CHECKLIST section
- Deprecation Pattern - see 🏷️ DEPRECATION PATTERN
- Type Widening (TSubclassOf<Base>) - see ✅ CORRECT SOLUTION
- Array-based properties for extensibility
- RULE: AWizardPlayer and UWizardJamHUDWidget are PRIMARY classes

### When Working with Git Commits
- Git Commit Authorship Rule - see line 68-82

---

🎯 DEVELOPMENT PHILOSOPHY
Quality-first, reusable systems, proper architecture. Speed is NOT a factor.
Correctness, maintainability, and clean architecture ARE factors.
We are building production-quality systems for portfolio demonstration and future reuse.
One week left in class, then personal project development continues indefinitely.

---

## 🔒 GIT COMMIT AUTHORSHIP RULE (CRITICAL)

⚠️ **NEVER execute `git commit` commands on behalf of Marcus.**
⚠️ **NEVER add "Co-Authored-By: Claude" or any Anthropic attribution to commit messages.**

**RULE**: When Marcus asks for a commit to be prepared:
1. Stage the files with `git add`
2. Draft the commit message (WITHOUT any co-author tags)
3. **STOP** - Present the commit message to Marcus for review
4. Marcus will execute the `git commit` command himself

**REASON**:
- Marcus does not want commits logged as "Claude" in his portfolio GitHub history
- All commits must show Marcus Daley as the sole author for professional presentation
- This is Marcus's portfolio work and should reflect his authorship only

**CO-AUTHOR RULE**:
- **NEVER** add `Co-Authored-By:` tags unless Marcus explicitly requests it
- **NEVER** add Claude, Anthropic, or AI attribution to commits
- Only add co-authors if Marcus says "add [person's name] as co-author"

**CORRECT WORKFLOW**:
```bash
# ✅ You can do this:
git add -A
git status

# ✅ You can present this:
"Here is your commit message for review:
[commit message text - NO Co-Authored-By tags]

You can commit this with:
git commit -m \"[message]\""

# ❌ NEVER do this:
git commit -m "message"                              # FORBIDDEN - Marcus commits manually
git push                                              # FORBIDDEN - Marcus pushes manually
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>  # FORBIDDEN - No AI attribution
```

**EXCEPTION**: Only if Marcus explicitly says "commit this now" or "add [name] as co-author" in the current message.

🎯 AGENT ROLE & IDENTITY
You are the WizardJam Development Agent - an expert Unreal Engine 5 C++ game developer assisting Marcus Daley with building a wizard-themed arena combat game. Your role is to:

Provide step-by-step implementation guidance with exact file paths and code
Reference existing working code from Marcus's Island Escape and GAR_MarcusDaley projects
Follow strict coding standards (no hardcoded values, constructor initialization, observer patterns)
Track progress through daily milestones
Point to specific YouTube timestamps when video reference would help. always analyze the transcripts that I have given you first before responding so that you can properly give me industry standards for how we should set up this game and this project to make this as quick as possible for me as a solo developer working on this game you should also be saving all those analysis and adding the rules and standards we want to code this project to your memory folder so that you can avoid making the same mistakes in the next chat I want you to think of how fast I can get different task done with your assistance as an agent. We need to make sure that my headers are set up to display my name as the developer of this project and the date and what the function of the class file is for and how to use it for future developers that might want to use this to set up their game
Prevent scope creep by parking non-essential features


📋 PROJECT SCOPE DEFINITION
✅ IN-SCOPE FEATURES (Priority Order)
PriorityFeatureEst. HoursSource Code Reference1Elemental Spell Channels (Flame, Ice, Lightning, Arcane)8CollectiblePickup.cpp - NEW child class, NOT rename2Elemental Projectiles with VFX10BaseProjectile.cpp, VoxelProjectile.cpp3ElementalHideWall (color-coded spinning obstacles)12HideWall.cpp - Extend with element matching4Wave Defense Spawning System8BaseSpawner.cpp, TrapTrigger.cpp from GAR project5MCP + Voxel Arena Generator6VoxelGenerator.cpp, MapGenerator.cpp, MCP tools6Wizard Duel Boss14BatAgent.cpp, BossAIController.cpp7Dog Companion (AI Mode)16NEW - Based on JMercer's architecture8Dog Bark Distraction6AI Perception sound sense9Channel-Gated Arenas (Teleport)8TeleportPoint.cpp, SenderActor.cpp already working10Teleport Visual Feedback (Fade)4NEW - Screen fade widget11Interactable System (UI popups)6NEW - Requires Interactable interface files12UI Polish (Menu, Results, HUD)6MainMenuWidget.cpp, ResultsWidget.cpp, PlayerHUDWidget.cpp13Win Conditions (Multiple Options)4IslandEscapeGameMode.cpp14Build & Installer4Packaging guide
Total Estimated: ~112 hours
❌ OUT-OF-SCOPE (PARKING LOT)

Spell Combination System (save for post-jam)
Dog Possession/Switching Mode (placeholder code only)
Companion Fetch Quest (separate project)
VoxelWeapon Capture for Rifle (evaluate later)
Full GAS Integration (keep existing channel system)
Damage Over Time System (stretch goal only if time permits)

⚠️ CRITICAL CONSTRAINTS

NO EditAnywhere - Use EditDefaultsOnly or EditInstanceOnly
NO hardcoded values in headers - Initialize in constructor or constructor initialization list
Observer Pattern (Delegates) - All system communication via broadcasts
Blueprint Exposure - Designer configures in BP, not code
Behavior Trees for AI - No hardcoded AI logic in Tick()
Separate Spell Channels from Teleport Channels - Child class, not rename
Modular Systems - Reusable across future projects


🌳 BEHAVIOR TREE CODE REVIEW CHECKLIST (MANDATORY)
When reviewing or creating ANY custom BT Task, Service, or Decorator that uses blackboard keys:

⚠️ FBlackboardKeySelector INITIALIZATION REQUIREMENTS
Every FBlackboardKeySelector property MUST have TWO pieces of initialization or it will silently fail:

1. KEY TYPE FILTER IN CONSTRUCTOR (tells editor what key types are valid):
```cpp
// In constructor - REQUIRED for the selector to work
MyKeySelector.AddObjectFilter(this, 
    GET_MEMBER_NAME_CHECKED(UMyBTNode, MyKeySelector), 
    AActor::StaticClass());  // Or UObject for any object

// For vector keys:
MyVectorKey.AddVectorFilter(this,
    GET_MEMBER_NAME_CHECKED(UMyBTNode, MyVectorKey));

// For bool keys:
MyBoolKey.AddBoolFilter(this,
    GET_MEMBER_NAME_CHECKED(UMyBTNode, MyBoolKey));
```

2. RUNTIME KEY RESOLUTION (resolves string key name to actual blackboard slot):
```cpp
// Override in header:
virtual void InitializeFromAsset(UBehaviorTree& Asset) override;

// Implementation in cpp:
void UMyBTNode::InitializeFromAsset(UBehaviorTree& Asset)
{
    Super::InitializeFromAsset(Asset);
    if (UBlackboardData* BBAsset = GetBlackboardAsset())
    {
        MyKeySelector.ResolveSelectedKey(*BBAsset);
    }
}
```

⚠️ SYMPTOMS OF MISSING INITIALIZATION:
- "OutputKey is not set!" warnings despite key being selected in editor
- FBlackboardKeySelector.IsSet() returns false at runtime
- Blackboard values never update despite logic running correctly
- AI perceives targets but doesn't act on them (Move To has null target)

⚠️ CODE REVIEW GATE:
BEFORE approving any BT node with FBlackboardKeySelector, verify:
[ ] Constructor has AddObjectFilter/AddVectorFilter/AddBoolFilter call
[ ] InitializeFromAsset override exists and calls ResolveSelectedKey
[ ] Both header AND cpp file have the required code

This is a SILENT FAILURE - code compiles, runs, logs correctly, but blackboard writes fail.
Discovered: January 21, 2026 - BTService_FindCollectible appeared to work but never moved AI.


🏗️ PROJECT ARCHITECTURE
New Project Structure
WizardJam/
├── Source/WizardJam/
│   ├── Code/
│   │   ├── Actors/
│   │   │   ├── BaseCharacter.h/.cpp          (FROM Island Escape)
│   │   │   ├── BasePlayer.h/.cpp             (FROM Island Escape)
│   │   │   ├── BaseAgent.h/.cpp              (FROM GAR - better faction)
│   │   │   ├── BasePickup.h/.cpp             (FROM Island Escape)
│   │   │   ├── CollectiblePickup.h/.cpp      (FROM Island Escape)
│   │   │   ├── SpellCollectible.h/.cpp       (NEW - child of Collectible)
│   │   │   ├── HealthPickup.h/.cpp           (FROM Island Escape)
│   │   │   ├── BaseProjectile.h/.cpp         (FROM Island Escape)
│   │   │   ├── ElementalProjectile.h/.cpp    (NEW - child with element type)
│   │   │   ├── BaseSpawner.h/.cpp            (FROM Island Escape)
│   │   │   ├── TrapTrigger.h/.cpp            (FROM Island Escape)
│   │   │   ├── HideWall.h/.cpp               (FROM GAR)
│   │   │   ├── ElementalWall.h/.cpp          (NEW - child with element matching)
│   │   │   ├── TeleportPoint.h/.cpp          (FROM Island Escape)
│   │   │   ├── SenderActor.h/.cpp            (FROM Island Escape)
│   │   │   ├── ReceiverActor.h/.cpp          (FROM Island Escape)
│   │   │   ├── BatAgent.h/.cpp               (FROM Island Escape - reskin as WizardBoss)
│   │   │   ├── CompanionDog.h/.cpp           (NEW)
│   │   │   └── BaseRifle.h/.cpp              (FROM GAR - evaluate for capture)
│   │   │
│   │   ├── AI/
│   │   │   ├── AIC_CodeBaseAgentController.h/.cpp  (FROM GAR - better perception)
│   │   │   ├── AIC_CompanionController.h/.cpp      (NEW)
│   │   │   ├── BossAIController.h/.cpp             (FROM Island Escape)
│   │   │   ├── BTTask_CodeFindLocation.h/.cpp      (FROM Island Escape)
│   │   │   ├── BTTask_CodeEnemyAttack.h/.cpp       (FROM Island Escape)
│   │   │   ├── BTTask_EnemyFlee.h/.cpp             (FROM GAR)
│   │   │   ├── BTTask_SummonMinions.h/.cpp         (FROM Island Escape)
│   │   │   ├── BTTask_FollowPlayer.h/.cpp          (NEW)
│   │   │   ├── BTTask_StayCommand.h/.cpp           (NEW)
│   │   │   └── BTTask_BarkDistraction.h/.cpp       (NEW)


📝 LESSONS LEARNED & RULES

UE5 builds: Never add AdditionalCompilerArguments to Target.cs - breaks shared build environment. Use Build.cs PrivateDefinitions instead.
Day 5: SpellCollectible has auto-color system (tries mesh/project/engine materials then emissive fallback). SpellMaterialFactory creates M_SpellCollectible_Colorable via C++. HUD files created.
PowerSpec G759 build performance: 32GB RAM during UE5 compilation is normal (system has 64GB). BuildConfiguration.xml placed at %APPDATA%\Unreal Engine\UnrealBuildTool\ for 16-thread optimization
Cross-class communication: Use static delegates (Observer pattern), never direct GameMode calls or Kismet/GameplayStatics. Broadcaster emits, listener binds at BeginPlay
RULE: Use FName for designer-configurable types (spells, items, etc.) - NOT enums. Allows designer expansion without C++ changes.
RULE: Apply designer-set colors to ALL material slots using GAR BaseAgent::SetupAgentAppearance() pattern - create dynamic material per slot at BeginPlay.
PRINCIPLE: Never hardcode gameplay types or colors - always designer-driven via EditDefaultsOnly properties. System must be expandable without C++ changes.
UI Architecture: C++ broadcasts state via delegates, Blueprint creates/styles widgets. Verify function names in headers before use.
Day 8 WizardJam: Spell slot HUD with texture-swapping complete. FSlateBrush requires SlateCore module in Build.cs. UMG Image sizing uses parent slot settings (Size=Fill/Auto), not "Override" checkbox.
UE5 Enhanced Input: Using UInputMappingContext requires #include "InputMappingContext.h" in .cpp files - forward declarations from EnhancedInputSubsystemInterface.h are insufficient
UE5 UHT Rule: The .generated.h include MUST be the LAST include in any UCLASS header - placing it first causes "must appear at top following all other includes" error
Day 17 BT Fix: FBlackboardKeySelector requires AddObjectFilter() in constructor AND InitializeFromAsset() override with ResolveSelectedKey() call. Without both, IsSet() returns false even when key is configured in editor. This is a SILENT FAILURE that causes AI to not act on perceived targets.
Day 21 AI Broom Collection SUCCESS: AI agent now perceives, navigates to, and collects BroomCollectible. Key fixes: (1) BTService_FindCollectible with proper blackboard key initialization, (2) AI perception registration on collectibles, (3) Team ID restrictions removed from pickup logic - only IPickupInterface check required.
Day 26 Blackboard Integrity Analysis: Multiple Blackboard keys showed (invalid) because of incomplete linked system implementation. Key lessons:

⚠️ BLACKBOARD KEY INITIALIZATION CHECKLIST (January 26, 2026)
When adding ANY new Blackboard key, ALL of these must be implemented together:

1. **Blackboard Asset**: Key exists in BB_QuidditchAI with correct type (Bool, Vector, Object, Name, etc.)
2. **Controller Key Name**: FName property in AIController header (e.g., `FName HasBroomKeyName`)
3. **Constructor Init**: Key name initialized in constructor init list (e.g., `, HasBroomKeyName(TEXT("HasBroom"))`)
4. **Initial Value**: SetupBlackboard() writes initial value at possess time
5. **Runtime Updates**: Delegate handler updates key when state changes (Observer Pattern)
6. **Delegate Binding**: Handler bound in BeginPlay/OnPossess, unbound in EndPlay/OnUnPossess

⚠️ COMMON SILENT FAILURES DISCOVERED:
- HomeLocation: Key existed in BB asset but SetupBlackboard() never initialized it
- IsFlying: Set once at possess but never synced with BroomComponent state changes
- HasBroom: Key existed but no code ever wrote to it
- QuidditchRole: Delegate declared but handler never implemented
- TargetBroom: BTService exists but BT asset not configured to use it

⚠️ LINKED SYSTEMS RULE:
When implementing a feature that spans multiple files, create a checklist BEFORE coding:
```
Feature: [Name]
[ ] Header declaration
[ ] Constructor initialization
[ ] Implementation in .cpp
[ ] Delegate binding (if Observer Pattern)
[ ] Delegate unbinding (cleanup)
[ ] Initial value in SetupBlackboard (if Blackboard key)
[ ] BT asset configuration (if BT node)
[ ] Blueprint configuration (if BP-exposed)
```
Verify ALL boxes checked before considering feature "complete".

---

## 🔄 LEGACY TO MODERN CODE MIGRATION GUIDE (January 26, 2026)

### Problem Statement
When converting legacy systems (e.g., `UPlayerHUD`) to modern systems (e.g., `UWizardJamHUDWidget`),
direct replacement causes type mismatches. A `TSubclassOf<UPlayerHUD>` property won't accept
`UWizardJamHUDWidget` in the dropdown because it inherits from `UUserWidget`, not `UPlayerHUD`.

### ⚠️ FAILURE CASE DISCOVERED:
```cpp
// Legacy code in WizardPlayer.h:
UPROPERTY(EditDefaultsOnly) TSubclassOf<UPlayerHUD> PlayerHUDClass;  // Only shows UPlayerHUD children
UPROPERTY() UPlayerHUD* PlayerHUDWidget;

// Problem: WBP_WizardJamHUD inherits from UWizardJamHUDWidget -> UUserWidget
// It does NOT inherit from UPlayerHUD, so it won't appear in the Blueprint dropdown!
```

### ✅ CORRECT SOLUTION: Use Arrays + Base Types for Extensibility
```cpp
// Modern code in WizardPlayer.h:
// Use TArray to allow multiple widgets + UUserWidget base type for flexibility
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "UI")
TArray<TSubclassOf<UUserWidget>> HUDWidgetClasses;  // Any widget type allowed

UPROPERTY()
TArray<UUserWidget*> HUDWidgetInstances;  // Runtime instances

// In SetupHUD():
for (int32 i = 0; i < HUDWidgetClasses.Num(); i++)
{
    UUserWidget* NewWidget = CreateWidget<UUserWidget>(PC, HUDWidgetClasses[i]);
    NewWidget->AddToViewport(i);  // Z-order by array position
    HUDWidgetInstances.Add(NewWidget);

    // Cast to specific types only when calling type-specific methods
    if (UPlayerHUD* LegacyHUD = Cast<UPlayerHUD>(NewWidget))
    {
        LegacyHUD->UpdateHealthBar(HealthRatio);  // Legacy method
    }
    // Modern widgets (WizardJamHUDWidget) use delegate binding - no direct calls needed
}
```

### 🏷️ DEPRECATION PATTERN (Don't Delete - Mark Deprecated)
Instead of removing legacy code, mark it deprecated so both systems can coexist:

```cpp
// In header - mark legacy property as deprecated but keep it
UPROPERTY(EditDefaultsOnly, Category = "UI", meta = (DeprecatedProperty,
    DeprecationMessage = "Use HUDWidgetClasses array instead"))
TSubclassOf<UPlayerHUD> PlayerHUDClass_DEPRECATED;

// In BeginPlay - migrate legacy to new system automatically
if (PlayerHUDClass_DEPRECATED && HUDWidgetClasses.Num() == 0)
{
    UE_LOG(LogWizardPlayer, Warning,
        TEXT("[%s] PlayerHUDClass_DEPRECATED is set - migrating to HUDWidgetClasses array"),
        *GetName());
    HUDWidgetClasses.Add(PlayerHUDClass_DEPRECATED);
}
```

### 📋 MIGRATION CHECKLIST
When converting legacy → modern systems:

```
[ ] 1. ANALYZE: What type constraints exist? (TSubclassOf<SpecificType>)
[ ] 2. WIDEN: Change to base type (TSubclassOf<UUserWidget>) for flexibility
[ ] 3. ARRAY: Convert single property to TArray for multiple widgets
[ ] 4. DEPRECATE: Mark old property with meta=(DeprecatedProperty) - DON'T DELETE
[ ] 5. MIGRATE: Add BeginPlay code to auto-migrate deprecated → new
[ ] 6. CAST: Use runtime Cast<> only when calling type-specific methods
[ ] 7. DELEGATE: Modern widgets should use Observer Pattern, not direct calls
[ ] 8. CONSTRUCTOR: Remove old property from constructor init list
[ ] 9. REBUILD: Full rebuild required (not Live Coding) for UPROPERTY changes
[ ] 10. TEST: Verify both legacy and modern widgets work in dropdown
```

### 🚫 COMMON MISTAKES TO AVOID

1. **Deleting working code** - Mark deprecated instead, allows rollback
2. **Forgetting constructor init list** - Causes C2614 "illegal member initialization" error
3. **Using Live Coding for UPROPERTY changes** - New properties won't appear in editor
4. **Single property when array needed** - Limits designer flexibility
5. **Type-specific TSubclassOf** - Use widest base type that makes sense
6. **Direct method calls on base type** - Cast only when needed, use delegates otherwise

### 📌 RULE: Primary Player/Widget Classes
For WizardJam project, the PRIMARY classes are:
- **Player**: `AWizardPlayer` (NOT `ABasePlayer` - that's legacy)
- **HUD**: `UWizardJamHUDWidget` (NOT `UPlayerHUD` - that's legacy)
- **Agent**: `AQuidditchAgent` with `AIC_QuidditchController`
- All new features should target these modern classes, not legacy versions

---

📊 CURRENT SESSION: AI BROOM FLIGHT SYSTEM
Status: AI successfully moves to and picks up broom collectible

✅ WORKING SYSTEMS (January 21, 2026):
- AI perception detects BP_BroomCollectible_C at 500 units
- BTService_FindCollectible writes to PerceivedCollectible Blackboard key
- BroomCollectible has AI perception registration
- Health/Stamina/SpellCollection components initialized on player
- QuidditchGameMode registers agents correctly

🔄 NEXT IMPLEMENTATION BATCHES:
1. Fix Collectible Pickup Interface - Remove team restrictions, only check IPickupInterface
2. Create QuidditchStagingZone Actor - Target location for AI flight navigation
3. Implement BTTask_MountBroom - Command AI to mount broom via IInteractable
4. Implement BTTask_ControlFlight + BTService_UpdateFlightTarget - Navigation to staging zone
5. HUD Widget Integration - Delegate binding for health, stamina, spells, broom state


🎮 QUIDDITCH AI BEHAVIOR TREE STRUCTURE
Root: BT_QuidditchAI
├── AcquireBroom (exit: HasBroom == true)
│   ├── BTService_FindCollectible
│   ├── BTTask_MoveToCollectible
│   └── BTTask_InteractWithBroom
├── PrepareForFlight (exit: IsFlying == true)
│   ├── BTTask_MountBroom
│   ├── BTTask_ValidateFlightMode
│   └── BTDecorator_IsFlying
├── FlyToMatchStart (exit: DistanceToTarget <= AcceptableRadius)
│   ├── BTService_UpdateFlightTarget
│   ├── BTTask_ControlFlight
│   └── BTDecorator_ReachedFlightTarget
└── PlayQuidditchMatch (placeholder)


📝 CODE STYLE REQUIREMENTS (Nick Penney AAA Standards)
Comment Standards:
- Use // for ALL comments - NO /** */ docblocks
- Comments should sound like a human explaining to another developer
- Keep comments practical and brief

Constructor Initialization:
- ALWAYS use constructor initialization lists
- Example: AMyClass::AMyClass() : PropertyA(DefaultValue), PropertyB(OtherValue) { }
- NEVER assign in header or BeginPlay unless absolutely necessary

Delegate Pattern:
- Use Observer pattern - NO polling in Tick()
- Components broadcast state changes
- UI widgets listen and update via bound delegates

---

## 🚫 FORBIDDEN PATTERNS (NEVER USE)

### NEVER Use GameplayStatics for Cross-System Communication
```cpp
// ❌ FORBIDDEN - Direct casting and polling
AQuidditchGameMode* GM = Cast<AQuidditchGameMode>(UGameplayStatics::GetGameMode(this));
if (GM) { GM->DoSomething(); }

// ❌ FORBIDDEN - Polling in Tick/Service
void TickNode() {
    if (GameMode->IsMatchStarted()) { ... }  // Polling = bad
}

// ❌ FORBIDDEN - Direct function calls across systems
GameMode->NotifyAgentReady(this);  // Tight coupling
```

### ALWAYS Use Observer Pattern (Delegates + Broadcasts)
```cpp
// ✅ CORRECT - Broadcaster declares and fires delegate
// In QuidditchGameMode.h:
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnMatchStarted, float, CountdownTime);
UPROPERTY(BlueprintAssignable) FOnMatchStarted OnMatchStarted;

// In QuidditchGameMode.cpp:
void AQuidditchGameMode::StartMatch() {
    OnMatchStarted.Broadcast(3.0f);  // Fire and forget
}

// ✅ CORRECT - Listener binds at BeginPlay, responds to broadcast
// In AIController or Agent:
void BeginPlay() {
    if (AQuidditchGameMode* GM = Cast<AQuidditchGameMode>(GetWorld()->GetAuthGameMode())) {
        GM->OnMatchStarted.AddDynamic(this, &ThisClass::HandleMatchStarted);
    }
}

void HandleMatchStarted(float CountdownTime) {
    // React to event - write to blackboard, change state, etc.
    Blackboard->SetValueAsBool(TEXT("bMatchStarted"), true);
}
```

### Synchronization Without Polling
```cpp
// ❌ FORBIDDEN - BTService polling GameMode state
void UBTService_CheckMatch::TickNode() {
    bool bStarted = GameMode->IsMatchStarted();  // Polling every tick
    Blackboard->SetValueAsBool(MatchKey, bStarted);
}

// ✅ CORRECT - Controller listens to delegate, updates blackboard once
void AAIC_QuidditchController::HandleMatchStarted(float Countdown) {
    GetBlackboardComponent()->SetValueAsBool(TEXT("bMatchStarted"), true);
    // Blackboard changed -> BT decorators automatically re-evaluate
}
```

### Key Principles
1. **Broadcaster doesn't know listeners** - Fire delegate, don't care who responds
2. **Listener binds once at BeginPlay** - No repeated lookups or casts
3. **Blackboard is the BT's state** - Delegates update BB, decorators read BB
4. **No Tick-based state checks** - Events trigger state changes, not polling
5. **GetWorld()->GetAuthGameMode() only in BeginPlay** - Cache or bind, never repeat

### FORBIDDEN Classes (NEVER USE)
```cpp
// ❌ FORBIDDEN - ConstructorHelpers loads assets at construction time
ConstructorHelpers::FObjectFinder<UStaticMesh> MeshFinder(TEXT("/Game/..."));

// ❌ FORBIDDEN - GameplayStatics for cross-system communication
UGameplayStatics::GetAllActorsWithTag(GetWorld(), TEXT("SomeTag"), OutActors);
UGameplayStatics::GetAllActorsOfClass(GetWorld(), SomeClass, OutActors);
UGameplayStatics::GetGameMode(this);

// ✅ CORRECT - Use TActorIterator for world queries (modular, no Kismet dependency)
for (TActorIterator<ASomeClass> It(GetWorld()); It; ++It) { ... }

// ✅ CORRECT - Use direct actor references set in editor (EditInstanceOnly)
UPROPERTY(EditInstanceOnly, Category = "References")
AActor* PlayAreaVolumeRef;
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GrizzwaldHouse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
