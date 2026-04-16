## wipshmup2

> System architecture, autoload singletons, and integration patterns - Comprehensive guide to the event-driven architecture


# System Architecture Overview

## Autoload Singleton Hierarchy

### Load Order and Dependencies
The autoloads are loaded in this specific order (defined in `project.godot`):

1. **EventBus** - Global event system (no dependencies)
2. **GameState** - Game state management (depends on EventBus)
3. **EntityFactory** - Object pooling and spawning (depends on EventBus, GameState)
4. **EnemyTemplateManager** - Enemy definitions (no dependencies)
5. **BossTemplateManager** - Boss definitions (no dependencies)
6. **StageTemplateManager** - Stage/wave definitions (depends on template managers)

**CRITICAL**: This order MUST be maintained. Do not add autoloads between these.

### EventBus - Global Event Hub

#### Purpose
Central publish-subscribe system for decoupled cross-system communication.

#### Event Categories
```gdscript
# Player Events
signal player_spawned(player: Player)
signal player_died(position: Vector2, lives_remaining: int)
signal player_bomb_used(bomb_count_remaining: int)
signal player_hit(damage: int, invulnerable_time: float)
signal player_power_changed(new_power: int, max_power: int)

# Enemy Events
signal enemy_spawned(enemy: Enemy, template_id: String)
signal enemy_defeated(enemy_id: int, position: Vector2, score_value: int)
signal enemy_hit(enemy_id: int, damage: int, remaining_health: int)

# Boss Events
signal boss_spawned(boss: Boss, boss_id: String)
signal boss_phase_changed(boss_id: String, phase_number: int, phase_name: String)
signal boss_defeated(boss_id: String, completion_time: float, no_miss: bool)
signal boss_warning(boss_id: String, warning_duration: float)

# Combat Events
signal bullet_fired(bullet: Bullet, source: Node, bullet_type: String)
signal bullet_hit(bullet: Bullet, target: Node, damage: int)
signal item_dropped(item: Item, position: Vector2, item_type: String)
signal item_collected(item_type: String, value: Variant)

# Stage Events
signal stage_started(stage_number: int, stage_name: String)
signal stage_cleared(stage_number: int, time: float, score: int)
signal wave_started(wave_number: int, enemy_count: int)
signal wave_cleared(wave_number: int, time: float)

# Visual Events
signal screen_shake_requested(intensity: float, duration: float)
signal slow_motion_requested(time_scale: float, duration: float)
signal flash_effect_requested(color: Color, duration: float)
signal background_change_requested(background_id: String, transition_time: float)

# Audio Events
signal music_change_requested(track_id: String, fade_time: float)
signal sfx_play_requested(sfx_id: String, position: Vector2, volume: float)
signal music_intensity_changed(intensity: float)  # For dynamic music layers
```

#### Usage Patterns
```gdscript
# ✅ GOOD: Emit events instead of direct calls
func _on_enemy_health_depleted() -> void:
    EventBus.enemy_defeated.emit(
        enemy_id,
        global_position,
        score_value
    )

# Subscribe to events in _ready()
func _ready() -> void:
    if not EventBus.player_died.is_connected(_on_player_died):
        EventBus.player_died.connect(_on_player_died)

# Disconnect in cleanup
func _exit_tree() -> void:
    if EventBus.player_died.is_connected(_on_player_died):
        EventBus.player_died.disconnect(_on_player_died)

# ❌ BAD: Direct cross-system calls
func _on_enemy_health_depleted() -> void:
    get_node("/root/ScoreManager").add_score(100)  # Tight coupling!
    get_node("/root/AudioManager").play_sfx("explosion")  # Fragile!
```

#### Event Naming Conventions
- Past tense for completed actions: `player_died`, `enemy_defeated`
- Present tense for requests: `screen_shake_requested`, `music_change_requested`
- Include relevant data as signal parameters (strongly typed)
- Avoid generic names like `event`, `signal`, `update`

### GameState - Centralized State Management

#### Purpose
Single source of truth for player stats, game progress, and global state.

#### State Variables
```gdscript
# Player Stats (Read via getters, write via setters that emit signals)
var lives: int = 3
var bombs: int = 3
var score: int = 0
var continues_used: int = 0
var power_level: int = 1
var max_power: int = 4

# Game Progress
var current_stage: int = 1
var current_wave: int = 0
var difficulty: Difficulty = Difficulty.NORMAL
var rank: float = 0.0  # Dynamic difficulty from 0.0 to 10.0

# Chain and Combo System
var chain_count: int = 0
var chain_timer: float = 0.0
var score_multiplier: float = 1.0

# Game Flow Flags
var is_paused: bool = false
var is_game_over: bool = false
var is_boss_active: bool = false
var player_is_alive: bool = true
```

#### Access Patterns
```gdscript
# ✅ GOOD: Read via getters
var current_lives: int = GameState.get_lives()
var player_score: int = GameState.get_score()

# ✅ GOOD: Write via setters (emits change signals)
GameState.add_score(100)  # Emits score_changed signal
GameState.consume_bomb()  # Emits bomb_used signal, updates count
GameState.lose_life()     # Emits life_lost signal, may trigger game over

# ✅ GOOD: Query flags
if GameState.is_boss_active:
    _adjust_spawning_logic()

# ❌ BAD: Direct state duplication
class_name Player extends Node2D
var lives: int = 3  # Don't duplicate GameState.lives!

# ❌ BAD: External modification without setters
func take_damage() -> void:
    GameState.lives -= 1  # Bypasses validation and signals!
```

#### State Change Signals
```gdscript
# GameState emits these when state changes:
signal score_changed(new_score: int, delta: int)
signal lives_changed(new_lives: int, delta: int)
signal bombs_changed(new_bombs: int, delta: int)
signal power_changed(new_power: int, max_power: int)
signal rank_changed(new_rank: float, old_rank: float)
signal chain_broken(final_chain: int, bonus_score: int)
signal game_over()
signal continue_used(continues_remaining: int)
```

#### State Persistence
```gdscript
# ✅ GOOD: Save/load state for continues or checkpoints
func save_checkpoint() -> Dictionary:
    return GameState.serialize_state()

func restore_checkpoint(state_data: Dictionary) -> void:
    GameState.deserialize_state(state_data)

# State serialization includes:
# - score, lives, bombs, power
# - current stage and wave
# - rank and chain data
# - used continues
```

### EntityFactory - Object Pooling System

#### Purpose
Centralized spawning with object pooling for performance. Manages lifecycle of all dynamic entities.

#### Pool Configuration
```gdscript
# Pre-configured pool limits (defined in EntityFactory)
const POOL_LIMITS: Dictionary = {
    "player_bullet_basic": 200,
    "player_bullet_focused": 150,
    "enemy_bullet_basic": 300,
    "enemy_bullet_fast": 200,
    "enemy_popcorn": 50,
    "enemy_tank": 20,
    "particles_explosion": 30,
    "particles_hit": 50,
}
```

#### Spawn Methods
```gdscript
# ✅ GOOD: Spawn player bullets
var bullet: Bullet = EntityFactory.spawn_bullet(
    "player_basic",           # Bullet type from pool
    global_position,           # Spawn position
    Vector2.UP                 # Direction vector
)
if bullet != null:
    bullet.speed = 300.0
    bullet.damage = 10

# ✅ GOOD: Spawn enemies
var enemy: Enemy = EntityFactory.spawn_enemy(
    "popcorn_01",             # Template ID
    spawn_position,
    Vector2.DOWN
)
if enemy != null:
    enemy.movement_speed = 100.0

# ✅ GOOD: Spawn with return-to-pool callback
var explosion: GPUParticles2D = EntityFactory.spawn_effect(
    "explosion_large",
    explosion_position
)
if explosion != null:
    explosion.emitting = true
    # Factory automatically returns to pool when finished

# ❌ BAD: Direct instantiation bypasses pooling
var bullet_scene := preload("res://scenes/bullet/bullet.tscn")
var bullet := bullet_scene.instantiate()
get_tree().current_scene.add_child(bullet)
```

#### Return to Pool
```gdscript
# ✅ GOOD: Return via factory
func _on_bullet_hit() -> void:
    EntityFactory.return_to_pool(self, "player_bullet_basic")

# ✅ GOOD: Factory handles cleanup automatically
# Bullets auto-return when leaving screen bounds
# Enemies auto-return when health reaches zero

# ⚠️ MANUAL POOLING: Only needed for custom entities
func return_to_pool() -> void:
    # Disconnect signals
    _disconnect_all_signals()
    
    # Reset state
    position = Vector2.ZERO
    velocity = Vector2.ZERO
    health = max_health
    
    # Disable processing
    visible = false
    process_mode = Node.PROCESS_MODE_DISABLED
    
    # Notify factory
    EntityFactory.return_to_pool(self, pool_id)
```

#### Rate Limiting
```gdscript
# Factory enforces rate limits to prevent performance spikes
# If pool is exhausted, spawn returns null

var bullet := EntityFactory.spawn_bullet("enemy_basic", pos, dir)
if bullet == null:
    # Pool exhausted or rate limited
    # Degrade gracefully - skip this bullet
    return

# ✅ GOOD: Check for null after spawn
# ❌ BAD: Assume spawn always succeeds
```

### EnemyTemplateManager - Enemy Definitions

#### Purpose
Centralized enemy type definitions with data-driven configuration.

#### Template Structure
```gdscript
# Enemy template data structure
class_name EnemyTemplate extends Resource

@export var template_id: String = ""
@export var enemy_name: String = "Unknown"
@export var max_health: int = 10
@export var score_value: int = 100
@export var movement_type: String = "straight"
@export var movement_speed: float = 100.0
@export var attack_pattern: String = "none"
@export var attack_cooldown: float = 2.0
@export var sprite_texture: Texture2D = null
@export var collision_radius: float = 8.0
@export var item_drop_chance: float = 0.1
@export var item_drop_types: Array[String] = []
```

#### Usage Patterns
```gdscript
# ✅ GOOD: Load template and apply to enemy
func spawn_enemy_from_template(template_id: String, position: Vector2) -> Enemy:
    var template := EnemyTemplateManager.get_template(template_id)
    if template == null:
        push_error("[Spawner] Template not found: %s" % template_id)
        return null
    
    var enemy := EntityFactory.spawn_enemy(template_id, position, Vector2.DOWN)
    if enemy == null:
        return null
    
    # Apply template data
    enemy.apply_template(template)
    return enemy

# ✅ GOOD: Register custom templates in _ready
func _ready() -> void:
    var custom_template := EnemyTemplate.new()
    custom_template.template_id = "custom_boss_minion"
    custom_template.max_health = 50
    custom_template.score_value = 500
    EnemyTemplateManager.register_template(custom_template)

# ❌ BAD: Hardcoded enemy stats
func spawn_enemy(position: Vector2) -> Enemy:
    var enemy := Enemy.new()
    enemy.health = 50  # Should be in template!
    enemy.speed = 100  # Should be in template!
    return enemy
```

#### Template Registration
```gdscript
# Templates are loaded from resources and JSON config
# See wipshmup2/config/enemies.json

# ✅ GOOD: Templates loaded at startup
func _ready() -> void:
    EnemyTemplateManager.load_templates_from_config("res://config/enemies.json")
    
    # Verify templates loaded
    var available_templates := EnemyTemplateManager.get_all_template_ids()
    print("[Game] Loaded %d enemy templates: %s" % [
        available_templates.size(),
        str(available_templates)
    ])
```

### BossTemplateManager - Boss Definitions

#### Purpose
Boss-specific templates with multi-phase support and advanced behavior configuration.

#### Boss Template Structure
```gdscript
class_name BossTemplate extends Resource

@export var boss_id: String = ""
@export var boss_name: String = "Unknown Boss"
@export var total_health: int = 1000
@export var phases: Array[BossPhase] = []
@export var warning_duration: float = 3.0
@export var defeat_score: int = 10000
@export var sprite_texture: Texture2D = null
@export var music_track: String = "boss_theme_01"

# Boss Phase structure
class_name BossPhase extends Resource
@export var phase_number: int = 1
@export var phase_name: String = "Phase 1"
@export var health_threshold: float = 1.0  # 1.0 = 100%, 0.5 = 50%
@export var movement_pattern: String = "circle"
@export var attack_patterns: Array[String] = []
@export var attack_interval: float = 2.0
@export var speed_multiplier: float = 1.0
```

#### Phase Management
```gdscript
# ✅ GOOD: Boss phases are triggered by health thresholds
func _on_health_changed(new_health: int, max_health: int) -> void:
    var health_percent: float = float(new_health) / float(max_health)
    
    var next_phase := BossTemplateManager.get_next_phase(boss_id, health_percent)
    if next_phase != null and next_phase.phase_number > current_phase:
        _enter_phase(next_phase)

func _enter_phase(phase: BossPhase) -> void:
    current_phase = phase.phase_number
    
    # Clear old behaviors
    _clear_behaviors()
    
    # Apply new phase behaviors (NOT deferred - boss is already in tree)
    var movement_behavior := _create_movement_behavior(phase.movement_pattern)
    add_child(movement_behavior)
    
    for attack_pattern_id in phase.attack_patterns:
        var attack_behavior := _create_attack_behavior(attack_pattern_id)
        add_child(attack_behavior)
    
    # Emit phase change event
    EventBus.boss_phase_changed.emit(boss_id, phase.phase_number, phase.phase_name)

# ⚠️ CRITICAL: Do NOT use call_deferred for behavior attachment
# Boss is already in tree, behaviors need to start immediately
```

### StageTemplateManager - Stage and Wave Definitions

#### Purpose
Defines stage progression, wave composition, and enemy formation patterns.

#### Stage Definition Structure
```gdscript
class_name StageDefinition extends Resource

@export var stage_number: int = 1
@export var stage_name: String = "Stage 1"
@export var background_id: String = "space_01"
@export var music_track: String = "stage_01"
@export var waves: Array[WaveDefinition] = []
@export var boss_id: String = ""  # Empty if no boss
@export var scroll_speed: float = 30.0

# Wave Definition structure
class_name WaveDefinition extends Resource
@export var wave_number: int = 1
@export var spawn_entries: Array[SpawnEntry] = []
@export var wave_duration: float = 30.0

# Spawn Entry structure
class_name SpawnEntry extends Resource
@export var template_id: String = ""
@export var spawn_time: float = 0.0
@export var spawn_position: Vector2 = Vector2.ZERO
@export var formation: String = "single"  # single, line, v_formation, circle
@export var formation_count: int = 1
@export var formation_spacing: float = 32.0
```

#### Stage Flow Integration
```gdscript
# StageController uses StageTemplateManager
func start_stage(stage_number: int) -> void:
    var stage_def := StageTemplateManager.get_stage(stage_number)
    if stage_def == null:
        push_error("[StageController] Stage %d not found" % stage_number)
        # Loop back to stage 1 if no more stages
        stage_def = StageTemplateManager.get_stage(1)
    
    _current_stage = stage_def
    _current_wave_index = 0
    
    # Emit stage start event
    EventBus.stage_started.emit(stage_def.stage_number, stage_def.stage_name)
    
    # Change background and music
    EventBus.background_change_requested.emit(stage_def.background_id, 1.0)
    EventBus.music_change_requested.emit(stage_def.music_track, 2.0)
    
    # Start first wave
    _start_next_wave()

func _start_next_wave() -> void:
    if _current_wave_index >= _current_stage.waves.size():
        _start_boss_phase()
        return
    
    var wave_def := _current_stage.waves[_current_wave_index]
    _spawn_wave_enemies(wave_def)
    
    EventBus.wave_started.emit(wave_def.wave_number, wave_def.spawn_entries.size())
```

## Component-Based Enemy System

### Movement Behaviors
```gdscript
# Base movement behavior (extended by specific patterns)
class_name MovementBehavior extends Node

@export var speed: float = 100.0
@export var target_node: Node2D = null

func _physics_process(delta: float) -> void:
    var parent := get_parent() as Node2D
    if parent == null:
        return
    
    var direction := calculate_direction(parent.global_position)
    parent.position += direction * speed * delta

func calculate_direction(current_pos: Vector2) -> Vector2:
    # Override in subclasses
    return Vector2.DOWN

# Example: Straight movement
class_name StraightMovement extends MovementBehavior

func calculate_direction(current_pos: Vector2) -> Vector2:
    return Vector2.DOWN.rotated(rotation_offset)

# Example: Homing movement
class_name HomingMovement extends MovementBehavior

@export var turn_speed: float = 2.0

func calculate_direction(current_pos: Vector2) -> Vector2:
    if not is_instance_valid(target_node):
        return Vector2.DOWN
    
    var desired_dir := (target_node.global_position - current_pos).normalized()
    var current_dir := Vector2.DOWN  # Or store as state
    return current_dir.lerp(desired_dir, turn_speed * get_physics_process_delta_time())
```

### Attack Behaviors
```gdscript
# Base attack behavior
class_name AttackBehavior extends Node

@export var cooldown: float = 2.0
@export var bullet_type: String = "enemy_basic"
@export var bullet_speed: float = 150.0

var _cooldown_timer: float = 0.0

func _process(delta: float) -> void:
    _cooldown_timer -= delta
    if _cooldown_timer <= 0.0:
        _execute_attack()
        _cooldown_timer = cooldown

func _execute_attack() -> void:
    # Override in subclasses
    pass

# Example: Single shot attack
class_name SingleShotAttack extends AttackBehavior

func _execute_attack() -> void:
    var parent := get_parent() as Node2D
    if parent == null:
        return
    
    var bullet := EntityFactory.spawn_bullet(
        bullet_type,
        parent.global_position,
        Vector2.DOWN
    )
    if bullet != null:
        bullet.speed = bullet_speed

# Example: Spread shot attack
class_name SpreadShotAttack extends AttackBehavior

@export var bullet_count: int = 5
@export var spread_angle: float = 45.0

func _execute_attack() -> void:
    var parent := get_parent() as Node2D
    if parent == null:
        return
    
    var angle_step := spread_angle / float(bullet_count - 1)
    var start_angle := -spread_angle / 2.0
    
    for i in range(bullet_count):
        var angle := start_angle + angle_step * i
        var direction := Vector2.DOWN.rotated(deg_to_rad(angle))
        
        var bullet := EntityFactory.spawn_bullet(
            bullet_type,
            parent.global_position,
            direction
        )
        if bullet != null:
            bullet.speed = bullet_speed
```

## Integration Rules

### Never Bypass Managers
```gdscript
# ❌ FORBIDDEN: Direct scene instantiation
var enemy_scene := load("res://scenes/enemy/enemy.tscn")
var enemy := enemy_scene.instantiate()

# ✅ REQUIRED: Use EntityFactory
var enemy := EntityFactory.spawn_enemy("popcorn_01", position, direction)

# ❌ FORBIDDEN: Direct state modification
player.lives = 5

# ✅ REQUIRED: Use GameState setters
GameState.set_lives(5)

# ❌ FORBIDDEN: Direct cross-system calls
score_manager.add_score(100)

# ✅ REQUIRED: Use EventBus
EventBus.enemy_defeated.emit(enemy_id, position, 100)
```

### Event Flow Example (Complete)
```gdscript
# 1. Enemy dies
func _on_health_depleted() -> void:
    EventBus.enemy_defeated.emit(enemy_id, global_position, score_value)
    EntityFactory.return_to_pool(self, "enemy_popcorn")

# 2. ScoreManager listens to EventBus
func _ready() -> void:
    EventBus.enemy_defeated.connect(_on_enemy_defeated)

func _on_enemy_defeated(enemy_id: int, position: Vector2, score: int) -> void:
    GameState.add_score(score)
    _check_for_item_drop(position)

# 3. GameState emits score_changed
func add_score(amount: int) -> void:
    var old_score := score
    score += amount
    score_changed.emit(score, amount)

# 4. UI listens to GameState
func _ready() -> void:
    GameState.score_changed.connect(_on_score_changed)

func _on_score_changed(new_score: int, delta: int) -> void:
    $ScoreLabel.text = "SCORE: %08d" % new_score
    _play_score_increase_animation()
```

### Safe Defaults for Missing Data
```gdscript
# ✅ GOOD: Provide defaults when templates missing
func spawn_enemy(template_id: String) -> Enemy:
    var template := EnemyTemplateManager.get_template(template_id)
    if template == null:
        push_error("[Spawner] Template '%s' not found, using default" % template_id)
        template = EnemyTemplateManager.get_default_template()
    
    var enemy := EntityFactory.spawn_enemy(template_id, position, direction)
    if enemy == null:
        return null  # Pool exhausted, handle gracefully
    
    enemy.apply_template(template)
    return enemy

# ✅ GOOD: Safe dictionary access
func load_wave_config(config: Dictionary) -> WaveDefinition:
    var wave := WaveDefinition.new()
    wave.wave_number = config.get("wave_number", 1)
    wave.wave_duration = config.get("duration", 30.0)
    wave.spawn_entries = config.get("spawns", [])
    return wave
```

## Checklist: Architecture Compliance

Before submitting code changes:
- [ ] All spawns use `EntityFactory` (no direct instantiation)
- [ ] All cross-system communication uses `EventBus` (no direct calls)
- [ ] All state read/write uses `GameState` (no duplicated state)
- [ ] All entity configs use Template Managers (no hardcoded stats)
- [ ] Boss behaviors added directly, not deferred (boss is in-tree)
- [ ] Signals connected with duplicate checks
- [ ] Signals disconnected in cleanup/pooling
- [ ] Safe defaults provided for missing data
- [ ] Error messages use `push_error()` with system prefix
- [ ] Component-based design for enemies (no deep inheritance)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/4t0m15) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
