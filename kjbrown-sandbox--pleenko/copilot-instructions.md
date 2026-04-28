## pleenko

> - I am an experienced programmer but brand new to Godot. I have no prior knowledge of Godot-specific concepts, APIs, functions, node types, signals, or documentation.

# CLAUDE.md

## Developer Context

- I am an experienced programmer but brand new to Godot. I have no prior knowledge of Godot-specific concepts, APIs, functions, node types, signals, or documentation.
- When explaining Godot concepts, provide clear explanations rather than assuming familiarity.
- The developer is rebuilding this project from scratch to learn Godot hands-on. Provide guidance and explain approaches rather than writing large blocks of code unless asked.

## Guidelines

- When I propose a feature or approach, validate it against Godot best practices and game industry conventions before implementing. If my suggestion conflicts with established patterns, flag it and explain the recommended alternative.
- Prefer idiomatic Godot solutions (e.g., using signals over polling, scene composition over deep inheritance, built-in nodes over custom reimplementations).
- When making modifications, make as many edits to the .tscn file as possible before relying on .gd for functionality.

## Game Description

This is an incremental/idle Plinko game. The player operates an ever-growing Plinko machine. Coins are dropped from the top and land in slots at the bottom. Each slot grants a different reward. As the player progresses, the Plinko machine expands with more pegs, slots, and reward types.

### Art Style

This is a minimalist 3D game. Use simple primitive shapes (spheres, cylinders, boxes) — no complex meshes or high-poly models. Keep visuals clean and lightweight.

### Core Physics Approach

Do NOT use a real physics engine for coin movement. The game needs to scale to tens of thousands of coins, so all coin paths are simulated/predetermined. The pegs are purely visual. This keeps performance stable regardless of coin volume.

Coins should calculate their path **row by row**, not all at once. This way if the board changes mid-drop (e.g., rows added), the coin dynamically adapts to the new layout and always lands in the correct bucket position. The coin picks left/right randomly at each row, queries the board for the next waypoint, and determines its final bucket value at landing time.

### Key Godot Patterns to Follow

- **Signals up, calls down.** Children emit signals to notify parents. Parents call methods on children to command them. Never the reverse.
- **Autoloads (singletons)** for managers that need global access: CurrencyManager, LevelManager, SaveManager. Register these in Project > Project Settings > Autoload.
- **Resources for data.** Upgrade definitions (cost, cap, scaling formula) should be Resource subclasses or data dictionaries, not hardcoded if/else chains. One generic `buy(upgrade_id)` function replaces many individual upgrade functions.
- **Each node manages its own children.** The board builds its own pegs/buckets. The UI builds its own buttons. The drop manager owns its own timers.
- **Scenes are self-contained.** Each `.tscn` + `.gd` pair handles its own initialization, state, and cleanup.

### System Responsibilities

> **Living documentation.** This section is the authoritative map of how systems own state, emit signals, and call into each other. It is kept in sync with the code — each time a feature branch is ready to merge to `main`, the relevant entries below are updated to reflect the new behavior, signals, data flows, and cross-system relations. New systems get new entries. Removed systems are deleted. The goal is that reading this section alone is enough to understand how the systems fit together without diving into the code.

#### Project layout

- `autoloads/` — singleton managers. One subdirectory per autoload.
- `entities/` — scenes (`.tscn` + `.gd` pairs). Each is self-contained.
- `scripts/` — shared data classes, utilities (enums, reward/tier data, format utils, offline earnings).
- `style_lab/` — `VisualTheme` resource, presets under `style_lab/presets/*.tres`, plus the in-editor style lab scene.
- `assets/` — icons, sounds, fonts.

Autoload init order is set in `project.godot` and matters: `TierRegistry → CurrencyManager → UpgradeManager → LevelManager → PrestigeManager → SaveManager → SceneManager → ChallengeManager → ThemeProvider → ModeManager → ChallengeProgressManager → AudioManager`. Later autoloads may subscribe to earlier ones in `_ready`.

#### Autoloads

**TierRegistry** — `autoloads/tier_registry/tier_registry.gd`

- Pure data lookup over the ordered tier chain (gold, orange, red, ...). No mutable state.
- Consumed by nearly every manager for per-board currency, drop costs, tier indices.
- Methods: `get_tier(board_type)`, `get_tier_index`, `get_next_tier`, `primary_currency`, `raw_currency`, `get_drop_costs`.
- Emits nothing.

**CurrencyManager** — `autoloads/currency_manager/currency_manager.gd`

- Owns balances + caps for all currencies.
- Methods: `add`, `spend`, `can_afford`, `get_balance`, `get_cap`, `buy_cap_raise`, `reset`, `serialize/deserialize`.
- Emits: `currency_changed(type, new_balance, new_cap)` on every mutation.
- LevelManager, UpgradeManager (for cap-raise unlocks), and ChallengeTracker listen to `currency_changed`.

**UpgradeManager** — `autoloads/upgrade_manager/upgrade_manager.gd`

- Owns per-board, per-upgrade state (level, cost, delta, caps, unlocked flag).
- `upgrade_gate: Callable` — optional gate set by `ChallengeManager` to block purchases during a challenge.
- Methods: `buy`, `can_buy`, `get_level`, `get_cost`, `unlock`, `is_cap_raise_available`, `buy_cap_raise`, `force_apply` (used for starting conditions), `serialize/deserialize`.
- Emits: `upgrade_purchased(upgrade_type, board_type, level)`, `upgrade_unlocked(upgrade_type, board_type)`, `cap_raise_unlocked(board_type)`, `autodropper_unlocked`, `advanced_autodropper_unlocked`.
- Listens: `LevelManager.rewards_claimed` (to unlock upgrades from level rewards), `CurrencyManager.currency_changed` (to flip cap-raise availability when the tier's raw currency is first earned).

**LevelManager** — `autoloads/level_manager/level_manager.gd`

- Owns `current_level` and the level table (thresholds, messages, rewards per slot).
- Level table is rebuilt per tier based on `TierRegistry` + `PrestigeManager` unlock state.
- Methods: `claim_rewards`, `get_active_currency`, `get_next_threshold`, `get_progress`, `get_tier_for_level`, `serialize/deserialize`.
- Emits: `level_changed(level)`, `level_up_ready(level, data)` (VFX layer listens), `rewards_claimed(level, rewards: Array[RewardData])`.
- Listens: `CurrencyManager.currency_changed` (to detect threshold crossings).

**PrestigeManager** — `autoloads/prestige_manager/prestige_manager.gd`

- Owns per-board prestige counts (0 = locked, ≥1 = permanently unlocked) and the current `PrestigePhase` (NONE, SLOW_MO, FREEZE, EXPAND, TRANSITION) which sets `Engine.time_scale`.
- Methods: `can_prestige`, `is_board_unlocked_permanently`, `get_multi_drop(board_type)`, `trigger_prestige`, `claim_prestige`, `enter_phase(phase)`, `serialize/deserialize`.
- Emits: `prestige_triggered(board_type)`, `prestige_claimed(board_type)`, `prestige_phase_changed(phase)`.
- Reads `ThemeProvider.theme` inside `enter_phase` for time-scale values. BoardManager queries multi-drop; LevelManager checks unlock state when rebuilding the level table.

**SaveManager** — `autoloads/save_manager/save_manager.gd`

- Orchestrates save/load to `user://save.json`. No signals, no state beyond an optional autosave timer.
- Methods: `setup(board_manager, autosave)`, `save_game`, `load_game`, `save_challenge_progress`, `load_prestige_only`, `reset_game`, `reset_game_without_reload`, `has_save`.
- Deserialization order (strict): `PrestigeManager → ChallengeProgressManager → LevelManager → CurrencyManager → UpgradeManager → BoardManager`. Order matters so signals fire against fully-initialized state.
- Calls `OfflineCalculator` (`scripts/offline/`) to credit earnings accumulated since last save.

**SceneManager** — `autoloads/scene_manager/scene_manager.gd`

- Thin scene-transition helper. `set_new_scene(packed_scene, instant)` — optional 1s fade overlay.

**ChallengeManager** — `autoloads/challenge_manager/challenge_manager.gd` (+ child `ChallengeTracker`)

- Lifecycle manager for active challenges. Owns `is_active_challenge`, the current `ChallengeData`, and a child `ChallengeTracker` node that runs live tracking.
- Methods: `set_challenge(challenge)` (marks active, emits state_changed), `setup(board_manager)` (creates tracker, applies starting conditions, sets gates on UpgradeManager + BoardManager), `clear_challenge`, `is_upgrade_allowed`, `is_board_allowed`, `get_time_remaining`, `get_total_seconds`, `get_total_drops`, `get_time_taken`, `get_objective_text`, `get_objective_progress`, `activate_survive_autodroppers`.
- Emits: `challenge_completed`, `challenge_failed(reason)`, `challenge_state_changed` (on set/clear — AudioManager listens), `tick(seconds_remaining)` (emitted by `ChallengeTracker` each integer-second crossing — AudioManager and ChallengeClock listen).
- Challenge start flow: caller calls `set_challenge`, then `ThemeProvider.set_theme(CHALLENGE)`, then `get_tree().reload_current_scene()`. After reload, `Main._setup_challenge` calls `ChallengeManager.setup(board_manager)` which creates the tracker.

**ChallengeTracker** (child node of ChallengeManager) — `autoloads/challenge_manager/challenge_tracker.gd`

- Runs one challenge: tracks coin landings, checks constraints and objectives, decrements `time_remaining`.
- Handles two-phase Survive objectives (WAITING → SURVIVING; activates autodroppers at transition via `ChallengeManager.activate_survive_autodroppers`).
- Emits per-integer-second `ChallengeManager.tick(seconds_remaining)` from both regular and SURVIVING phase timers.
- Emits `completed` / `failed(reason)` up to ChallengeManager.
- Listens: per-board `coin_landed`, `coin_dropped`, `autodrop_failed`; `BoardManager.board_switched`; `CurrencyManager.currency_changed`.

**ThemeProvider** — `autoloads/theme_provider/theme_provider.gd`

- Owns the active `VisualTheme` resource. Swaps `normal_theme ↔ challenge_theme` via `set_theme(kind)`. Configures the shared `WorldEnvironment` and (optionally) a `DirectionalLight3D` based on `theme.unshaded`.
- Emits: `theme_changed` (sync, on any theme swap).
- AudioManager reads `theme.audio_style` on `theme_changed` to route arcade audio; PrestigeManager reads theme fields for prestige time-scale values; every Bucket/Coin/Peg reads their colors/materials on setup.

**ModeManager** — `autoloads/mode_manager/mode_manager.gd`

- Tracks `MAIN` vs `CHALLENGES` mode.
- Methods: `switch_to_challenges`, `switch_to_main`, `is_main`, `is_challenges`, `are_challenges_unlocked` (queries `PrestigeManager`).
- Emits: `mode_changed(new_mode)`.

**ChallengeProgressManager** — `autoloads/challenge_progress/challenge_progress_manager.gd`

- Persistent state for challenge completion, unlock flags, starting modifiers, permanent upgrades. Survives a prestige reset (alongside PrestigeManager).
- Methods: `initialize(buttons)`, `get_state(id)`, `is_unlocked(unlock_type)`, `get_bonus_multi_drop`, `get_advanced_coin_multiplier_bonus`, `get_bucket_value_percent_bonus`, `get_permanent_upgrade_level`, `complete_challenge`, `serialize/deserialize`.
- Emits: `challenge_state_changed(id, state)`, `unlock_granted(unlock_type)`.
- Read by `PlinkoBoard.setup` to apply per-board bonus multipliers and permanent upgrade levels.

**AudioManager** — `autoloads/audio_manager/audio_manager.gd`

- Owns every sound in the game: legacy SFX pools, procedural harp (multi-sample), procedural arcade square-wave + kick, ambient pad per board, drone pool shared by bucket drones + harp sparkles, bucket/coin chimes, click sounds, drum pools (currently disabled).
- Owns the beat grid (`_beat_period`, `_beat_phase`, `_motif_position`, `_beat_armed`) and per-board harmonic progression state (`_board_progressions`, `_current_chord_index`). Arcade mode swaps in the AudioStyle's own progression via `_active_audio_style` + `_style_chord_index`.
- Methods: `play_bucket`, `play_peg_sparkle`, `play_peg_click`, `play_bucket_hit`, `set_active_board`, `on_coin_dropped`, `on_coin_landed`, `play_manual_drop_drum`, `play_autodropper_drum`, `play(sound_name, pitch, max_duration)`, `play_prestige`, `notify_autodropper_beat(interval)`, `should_sparkle(bt)`, `is_active_board(bt)`, `get_time_until_next_chord()`, `get_chord_duration()`, `get_chord_phase()`, `try_consume_bucket_activation(is_advanced)`.
- Emits: `chord_changed(chord_index)` — fired on every chord advance in both the default harp path and the AudioStyle path, and also from `_reset_harmonic_state` when silence resets the progression. PlinkoBoard listens to fade chord-activated buckets back to their faded color. The signal is visual-only; audio is not faded by a chord change (see chord-gated bucket lifecycle below).
- Listens: `ThemeProvider.theme_changed` (update lofi bus effects + re-select AudioStyle), `ChallengeManager.challenge_state_changed` (re-select style), `ChallengeManager.tick` (phase-lock beat grid + fire kick; final 10s schedules a second kick +0.5s later via `create_timer`).
- AudioStyle routing: when `ThemeProvider.theme.audio_style` is set AND (not `active_during_challenge_only` OR a challenge is active), that style replaces the default harp path — different chord progression, beat-period from `beats_per_tick`, square-wave timbre via `_tonal_stream_and_pitch`. On any style transition, `_fade_all_drones(1s)` clears lingering drones so the outgoing world doesn't bleed into the new one.
- Arcade-specific: kick is the only audible backing layer (once per second, doubled to 2/sec in the final 10 seconds). Peg sparkle audio is currently fully disabled (`play_peg_sparkle` is an unconditional early-return) because it clashed with the chord-gated bucket drone layer; peg ring VFX still fires through `should_sparkle`. A follow-up feature will redesign the sparkle voice. Bucket drones play as usual.
- Chord-gated bucket drones use a three-state machine (`DroneState` enum on the drone dict entries): `ACTIVE` (current chord, gates re-hits on same bucket), `LINGERING` (previous chord still ringing naturally — no gating, pool slot released when the synthesized decay ends or when a new coin lands), and `SPARKLE` (timer-decayed peg sparkles, unchanged from before). `play_bucket` allocates ACTIVE drones with `timer = max(_chord_timer, MIN_BUCKET_RING_SECONDS)`; `_update_bucket_drones` skips ACTIVE drones so the timer never cuts them off mid-chord. `_handle_chord_advance` flips ACTIVE → LINGERING and sets `timer = HARP_DECAY_SECONDS` so `_update_bucket_drones` releases the pool slot once the sample has decayed. `_fade_lingering_drones` runs at the top of every `play_bucket` call: the new coin hands off from the old chord's tail by fading every LINGERING drone over `VisualTheme.linger_fade_duration`. Fade tweens are tracked in `_drone_fade_tweens` keyed by pool idx and killed before a slot is reused. `_make_drone_entry` factory centralizes dict-literal construction across allocation sites.
- Drone density control: drones are routed to a dedicated `Drones` audio bus (separate from `Melody` which carries cellos/chimes/kicks) with a compressor (threshold -18 dB, ratio 3, attack 10 ms, release 500 ms) followed by a small-room reverb (room 0.4, wet 0.2, hipass 0.2) — effect order matters, compressor tames the dry stack first, reverb blooms off the controlled signal. Per-coin-type voice caps: `MAX_NORMAL_DRONES = 5`, `MAX_ADVANCED_DRONES = 3`. Normal coins (melodic top layer) and advanced coins (deeper bass punctuation) are tracked in independent pools, with per-type eviction via `_evict_oldest_drone_if_full(is_advanced)` + `_count_drones_of_type(is_advanced)` (priority: LINGERING → SPARKLE → ACTIVE, oldest-first tiebreak, fades over `VisualTheme.eviction_fade_duration`). Per-voice exponential attenuation is also filtered by coin type — normals only dim behind other normals, advanced only behind other advanced. Compressor params are co-tuned with `VOICE_ATTENUATION_RATIO = 0.75` (~2.5 dB/voice) — retune both together.
- Note: because drones live on the `Drones` bus, they are **not** affected by any effects on the `Melody` bus (such as the lofi low-pass filter). If lofi is re-enabled, mirror any Melody-bus effects onto the Drones bus too.
- Activation rate-limit (`try_consume_bucket_activation(is_advanced)`) is per-coin-type: `NORMAL_RATE_DIVISOR = 4.0` (~375 ms at default autodrop), `ADVANCED_RATE_DIVISOR = 1.0` (~1500 ms — one advanced hit per cycle). Independent timestamps so a fast normal cadence doesn't block advanced hits. A one-shot `HARMONY_GRACE_WINDOW` (200 ms) admits a second voice of the same type inside the cooldown so multi-drop coins — which take different peg paths and land ~100-200 ms apart — produce a two-note chord. The grace flag is consumed on that second voice; a third hit in the burst hits the normal cooldown. Flag resets on either a fresh accept outside cooldown OR in the rejection path once elapsed leaves the grace sub-window (prevents sustained sub-cooldown activity from permanently killing harmony). Visual `mark_active` is intentionally coupled to this gate in `PlinkoBoard.finalize_coin_landing`: a rate-limited hit leaves the bucket faded so the next tone-producing coin owns the activation.
- `get_chord_phase()` returns the 0..1 position within the current chord (0 = chord just started, 1 = about to change). `get_chord_duration()` returns the full chord length (AudioStyle override if active, else `CHORD_DURATION`). Drives Bucket's chord-synced scale pulse — all active buckets read the same global phase so they stay in visual sync no matter when each was activated.

#### Scene-level systems

**BoardManager** — `entities/board_manager/board_manager.gd`

- Orchestrates all `PlinkoBoard` instances. Owns `_boards[]`, `_active_index`, autodropper pool + assignments, camera tweening, and the autodropper timer.
- Methods: `setup`, `switch_board`, `unlock_board`, `get_active_board`, `get_boards`.
- Emits: `board_switched(board)`, `board_unlocked(type)`.
- Per-tick: `_autodrop_timer` (wait=1.5s) fires `_on_autodrop_tick`, which calls `AudioManager.notify_autodropper_beat(wait_time)` to sync the beat grid, then dispatches to assigned boards.
- Listens: `UpgradeManager.{autodropper_unlocked, advanced_autodropper_unlocked, upgrade_purchased}`, `CurrencyManager.currency_changed`, `LevelManager.rewards_claimed`, per-board `board_rebuilt` / `autodropper_adjust_requested`.

Each `PlinkoBoard` additionally listens to `AudioManager.chord_changed` (AudioStyle mode) and fades all its buckets to their faded color on every chord advance.

**PlinkoBoard** — `entities/plinko_board/plinko_board.gd`

- Per-board gameplay: peg + bucket multimesh rendering, coin spawning, drop queue, drop timer, per-board upgrade multipliers, bucket marking API for challenges.
- Methods: `setup(board_type)`, `request_drop(costs, coin_type, is_manual)`, `try_autodrop`, `add_two_rows`, `flash_nearest_peg(coin_pos, currency_type)`, `get_bucket(index)`, `get_nearest_bucket(x)`, `mark_bucket_*` (target/hit/unhit/forbidden).
- Emits: `coin_dropped`, `coin_landed(board_type, bucket_index, currency_type, amount, multiplier)`, `board_rebuilt`, `autodropper_adjust_requested`, `prestige_coin_landed`, `autodrop_failed(board_type)`.
- On coin land: calls `AudioManager.play_bucket(board_type, distance, is_advanced)` and `AudioManager.on_coin_landed`. On peg contact: `flash_nearest_peg` calls `AudioManager.should_sparkle` and `play_peg_sparkle`, and fires peg VFX (flash + pulse always, halo always, ring on beat-armed sparkles in coin color).
- Peg VFX split: the expanding ring uses the coin's color and only fires when `AudioManager.should_sparkle` returns true (beat-gated). Flash, halo, pulse always fire on peg contact.

**Coin** — `entities/coin/coin.gd`

- Individual coin animation. Picks left/right at each row, queries the board for the next waypoint, determines final bucket at landing time.
- Owned state: `coin_type`, `multiplier`, `impact_squash_remaining`, multimesh index.
- Methods: `start(target)`, `kill_tweens`, `set_color`, `set_mesh_visible`.
- Emits: `final_bounce_started(coin, predicted_bucket)` (triggers prestige handover when applicable), `landed`.

**Bucket** — `entities/bucket/bucket.gd`

- Per-bucket visual: `MeshInstance3D` with a per-instance `StandardMaterial3D`, `Label3D` showing value.
- Fields: `value` (setter updates label), `currency_type`, `is_prestige_bucket`, `_base_material`, `_is_hit`.
- Methods: `setup(currency_type, position, value)`, `mark_hit`, `mark_target`, `mark_unhit`, `mark_forbidden`, `pulse`, `mark_active`, `mark_inactive(duration)`.
- No signals (pure view).
- Buckets always start in the faded color and light up only while activated by a coin landing. `mark_active` snaps to the full main color, then schedules a delayed color tween that holds full color for most of the chord and fades to faded over `VisualTheme.bucket_fade_duration` in the final window — so the bucket is fully faded at the moment the next chord starts. It also flips `_is_pulsing = true` and enables `_process`, which reads `AudioManager.get_chord_phase()` each frame and sets `scale = lerp(bucket_active_scale_peak, 1.0, ease_out(phase))`. All active buckets read the same global phase so they share a uniform scale at any instant regardless of activation time. `mark_inactive(duration)` is a backstop called from `PlinkoBoard._on_chord_changed`: it kills any in-flight tweens, stops pulsing, and tweens color + scale back to the faded baseline. All `mark_*` methods go through `_apply_color(color)` and `_kill_color_tween()` / `_stop_pulsing()` for consistency. Both `mark_active` and `mark_inactive` no-op when `_is_hit` is true so challenge markers win. Visual activation is coupled to the audio rate-limit gate: `PlinkoBoard.finalize_coin_landing` only calls `bucket.mark_active()` when `AudioManager.try_consume_bucket_activation(is_advanced)` succeeds. A rate-limited hit leaves the bucket faded so the next tone-producing coin owns the activation.

**ChallengeClock** — `entities/challenge_clock/challenge_clock.gd` + `.tscn`

- White pie-slice countdown inside `ChallengeHUD`. Instanced by `challenge_hud.gd` on challenge start.
- Updates shape **only on** `ChallengeManager.tick` (discrete once-per-second steps — reinforces the audio kick). Reads `ChallengeManager.get_total_seconds()` to compute fraction.
- Hides on `challenge_completed` / `challenge_failed`.

**ChallengeHUD** — `entities/main/challenge_hud.gd` + nodes in `entities/main/main.tscn`

- Challenge UI container: timer label, objective label, progress label, result label, embedded `ChallengeClock`.
- Methods: `start(challenge)`, `show_result(text)`, `refresh_progress`.
- Polls `ChallengeManager.get_time_remaining` + `get_objective_progress` each frame for label updates.

**Main** — `entities/main/main.gd`

- Root scene orchestrator. Wires up BoardManager, ChallengeHUD, challenge dialogs, UI panels, prestige animator. On `_ready` decides between `_setup_normal()` and `_setup_challenge()` based on `ChallengeManager.is_active_challenge` (set before scene reload).
- Listens: `ModeManager.mode_changed`, `PrestigeManager.{prestige_claimed, prestige_phase_changed}`, `BoardManager.{board_switched, board_unlocked}`, `UpgradeManager.upgrade_unlocked`, `ChallengeManager.{challenge_completed, challenge_failed}`.

**DropSection** — `entities/drop_section/`

- Contains `DropButton` instances (normal + advanced). Each `DropButton` emits `drop_pressed` (wired to `PlinkoBoard.request_drop()`) and `autodropper_adjust_requested` (wired up to `BoardManager` via the board's matching signal).

#### Resources (data)

**VisualTheme** — `style_lab/visual_theme.gd`, presets in `style_lab/presets/*.tres`

- Huge bundle of visual configuration: background shades, per-currency colors, coin/bucket/label materials, VFX toggles (peg flash, halo, ring, pulse), coin physics timings, audio flags. Consumed by every visual system through `ThemeProvider.theme`.
- Audio-related fields: `audio_lofi_enabled` (gates lofi bus effects — currently parked), `audio_style: AudioStyle` (optional override; null = main harp).
- Helpers: `get_coin_color`, `get_bucket_color`, `make_bucket_mesh`, `make_bucket_material`, `make_coin_material`, `pulse_node3d`, plus palette resolution.

**AudioStyle** — `autoloads/audio_manager/audio_style.gd`

- Data-only resource attached to a `VisualTheme`. Describes an alternate audio world.
- Fields: `display_name`, `active_during_challenge_only`, `beats_per_tick`, `has_backing_kick`, `has_backing_bass`, `timbre` (`"square" | "harp"`), `progression[]` (array of `{ root, chord, motif }`), `chord_duration`, `bucket_accent_motif[]`.
- Current preset: `style_lab/presets/arcade_audio_style.tres` — square timbre, i-VI-VII-i in A minor, kick backing only (bass layer is defined but arcade currently doesn't fire it).

**ChallengeData** — `autoloads/challenge_manager/challenge_data.gd`

- Per-challenge metadata: `id`, `display_name`, `time_limit_seconds`, `objectives[]`, `constraints[]`, `starting_conditions[]`, `rewards[]`.

**Objective types** (files under `autoloads/challenge_manager/objectives/`): `Survive`, `LandInEveryBucket`, `HitBucketsInOrder`, `HitXBucketYTimes`, `GetSameBucketXTimes`, `EarnWithinXDrops`, `BoardGoal`, `CoinGoal`. Evaluated by `ChallengeTracker`.

**StartingCondition** subclasses (also under challenge_manager/): `StartingCap`, `StartingCoins`, `StartingUpgrades`, `StartingBoards`, `StartingDropDelay`. Applied by `ChallengeManager._apply_starting_conditions` when a challenge begins.

**RewardData** — `scripts/reward_data.gd`

- Unified reward container used by `LevelManager` level rewards and by challenge completion rewards.
- `type: RewardType` enum covers `UNLOCK_UPGRADE`, `DROP_COINS`, `UNLOCK_AUTODROPPER`, `UNLOCK_ADVANCED_AUTODROPPER`, `UNLOCK_ADVANCED_BUCKET`, with per-type fields (`upgrade_type`, `board_type`, `coin_count`, `coin_type`, `target_board`).

**BaseUpgradeData** — `autoloads/upgrade_manager/base_upgrade_data.gd`, presets in `autoloads/upgrade_manager/data/*.tres`

- Per-upgrade economy: `type`, `display_name`, `base_cost`, `max_level`, `cost_delta`. UpgradeManager reads max_level as the starting cap.

**TierData** — `scripts/tier_data.gd`, presets in `autoloads/tier_registry/data/*.tres`

- Per-tier config: `board_type`, `display_name`, `primary_currency`, `raw_currency`, economy caps, drop costs.

#### Cross-cutting data flows (quick map)

- **Currency → Progression:** `CurrencyManager.currency_changed` → `LevelManager` (threshold crossings) → `rewards_claimed` → `UpgradeManager.unlock` / reward dispatch.
- **Cap raises:** `CurrencyManager.currency_changed` on a tier's raw currency → `UpgradeManager.cap_raise_unlocked(board_type)`.
- **Challenge gates:** `ChallengeManager.setup` installs `upgrade_gate` on `UpgradeManager` and `board_gate` on `BoardManager`. Cleared in `clear_challenge`.
- **Challenge pulse:** `ChallengeTracker._process` → `ChallengeManager.tick(seconds_remaining)` (once per integer second). `AudioManager` uses it to fire the arcade kick + phase-lock the beat grid; `ChallengeClock` redraws the pie slice.
- **Theme → Audio:** `ThemeProvider.theme_changed` → `AudioManager._on_theme_changed` → `_reselect_audio_style` picks up `theme.audio_style`.
- **Challenge → Audio:** `ChallengeManager.challenge_state_changed` → `AudioManager._reselect_audio_style` (activates arcade if theme has a style with `active_during_challenge_only`). Any style transition triggers `AudioManager._fade_all_drones(1s)`.
- **Autodropper → Audio beat:** `BoardManager._on_autodrop_tick` → `AudioManager.notify_autodropper_beat(wait_time)` to sync the harp beat grid to the autodropper timer.
- **Chord-gated buckets (all themes):** coin landing → `PlinkoBoard.finalize_coin_landing` → `AudioManager.try_consume_bucket_activation(is_advanced)` gate. On accept: `Bucket.mark_active` on both the hit bucket and its mirror (symmetric buckets share a note) + `AudioManager.play_bucket` (drone allocated as `ACTIVE`; per-type cap — 5 normal / 3 advanced — enforced via eviction; per-type attenuation filter applies; any `LINGERING` drones from the previous chord are hand-faded first over `linger_fade_duration`). On reject: the coin still earns currency + fires pulse/floating-text, but no visual or audio activation. Chord advance → `AudioManager._handle_chord_advance` flips ACTIVE drones to LINGERING (audio keeps ringing via the synthesized harp tail) and emits `chord_changed(chord_index)` → every `PlinkoBoard._on_chord_changed` → `Bucket.mark_inactive(bucket_fade_duration)` on each bucket as a backstop. While active, each bucket's `_process` polls `AudioManager.get_chord_phase()` and eases scale from `bucket_active_scale_peak` to 1.0 — uniform across all active buckets. Idle silence (no activity for `CHORD_IDLE_RESET` seconds) fires `_reset_harmonic_state` which also calls `_handle_chord_advance` so visuals fade even when the player stops dropping.
- **Coin lifecycle:** `PlinkoBoard.request_drop` → `Coin.start(target)` → per-row board queries → `Coin.final_bounce_started` → `PlinkoBoard.finalize_coin_landing` → `coin_landed` signal (ChallengeTracker, BoardManager listen) + `AudioManager.play_bucket` + `AudioManager.on_coin_landed`.
- **Peg VFX:** `Coin._on_peg_hit` → `PlinkoBoard.flash_nearest_peg` → flash + pulse + halo (always) + ring (coin color, gated on `AudioManager.should_sparkle`).
- **Save:** `SaveManager.save_game` serializes each manager in order. `load_game` deserializes in the order `PrestigeManager → ChallengeProgressManager → LevelManager → CurrencyManager → UpgradeManager → BoardManager` so signals fire against a consistent state.

### Three-Currency Economy

| Currency         | Earned On                       | Used For                                          |
| ---------------- | ------------------------------- | ------------------------------------------------- |
| Gold             | Gold board buckets              | Gold board upgrades                               |
| Unrefined Orange | Gold board (orange buckets)     | Refined by dropping on gold board (3x multiplier) |
| Orange           | Orange board buckets            | Orange upgrades + raise gold upgrade caps         |
| Unrefined Red    | Gold/orange board (red buckets) | Refined by dropping (9x on gold, 3x on orange)    |
| Red              | Red board buckets               | Red upgrades + raise orange upgrade caps          |

### Board Progression Flow

1. Start with gold board (2 rows)
2. Add rows -> orange buckets appear at edges (requires ORANGE_ROW_GATE rows)
3. Earn unrefined orange -> orange board unlocks
4. Add rows to orange -> red buckets appear (requires RED_ROW_GATE rows on gold, ORANGE_ROW_GATE on orange)
5. Earn unrefined red -> red board unlocks

### Lessons from the Prototype

- **Coin paths must be dynamic.** The prototype pre-calculated all waypoints at drop time. When rows were added mid-drop, coins landed at stale positions. Row-by-row waypoint resolution fixes this.
- **Upgrade data should be data, not code.** The prototype had 42 separate upgrade handler functions. A data-driven approach (array of upgrade definitions) with one generic buy function is far more maintainable.
- **Cooldown timers per board.** Gold = 2s base, Orange = 4s base, Red = 8s base. Drop rate upgrades multiply by 0.8 per level.
- All queues can overfill during a level up
- **Shared mesh resources matter for performance.** Create CylinderMesh, BoxMesh, and materials once, reuse for all pegs/buckets. Don't instantiate new meshes per peg.
- **Tween patterns.** Use create_tween() for fire-and-forget animations. Chain tween_property() calls for sequential movement. Use .set_ease(EASE_IN) + .set_trans(TRANS_QUAD) for gravity-like feel. End with tween_callback() for cleanup.
- **Node cleanup.** Always queue_free() nodes when done (coins after landing, old pegs/buckets on rebuild). Nodes that aren't freed are memory leaks.
- **Save format versioning.** Plan for save format changes from the start. The prototype had to handle migration of old gold_queue format (int vs array of multipliers).

## Feature Planning Process (Plan Mode Only)

When the user enters plan mode and describes a feature, run a multi-agent review before writing any code. Five personalities evaluate the feature in parallel, debate concerns in rounds, and produce a consensus plan.

### The Five Personalities

Each personality evaluates proposed features through their specific lens. They should raise concerns, propose alternatives, and flag risks — all oriented toward **future code that will be written**, not auditing existing code.

**1. The Janitor — Code Cleanliness**

- Will this feature introduce duplication with existing code?
- Can it reuse or extend something that already exists?
- Will it create oversized files or tangled responsibilities?
- Does the proposed structure keep things easy to clean up later?

**2. The Godot Guru — Engine Best Practices**

- Is this using the right Godot nodes, patterns, and APIs?
- Does it follow "signals up, calls down"?
- Are there performance concerns (node count, per-frame work, memory)?
- Is the lifecycle correct (ready, enter_tree, exit_tree, queue_free)?
- Are tweens, timers, and resources handled properly?

**3. The Architect — Dependencies & Connections**

- How does this feature connect to existing systems?
- What signals need to be added or modified?
- What's the ripple effect — if this changes, what else breaks?
- Does this introduce circular dependencies or tight coupling?
- Is the data flow clear and traceable?

**4. The Newcomer — Readability & Clarity**

- Will a developer picking this up cold understand what it does?
- Are there magic numbers, cryptic names, or undocumented business logic?
- Is the control flow straightforward or entangled?
- Are naming conventions consistent with the rest of the codebase?

**5. The Consistency Lover — Standardization**

- Does this follow established codebase patterns (signal naming, typing, init patterns)?
- Are connection patterns consistent (direct method refs, not inline lambdas)?
- Does it use the same error handling approach as similar code?
- Are type annotations, naming conventions, and file structure consistent?
- Are we using theme variables instead of new ones like Color.WHITE (which is wrong)?

### Process

1. **Parallel analysis:** Spin up all 5 agents simultaneously. Each receives the feature description and analyzes it through their lens.
2. **Round 1 — Concerns:** Collect all concerns from the 5 agents. Present a summary to the user showing each personality's key concerns.
3. **Round 2+ — Resolution:** If there are conflicts between agents, run another round where each agent sees the others' concerns and responds. Continue for up to 3 rounds. Do not ask the user to resolve disagreements during this process — let the agents work it out.
4. **Escalation:** If no consensus after 3 rounds, present the unresolved disagreements to the user for a decision.
5. **Approval:** Present the final plan to the user. Only begin implementation after explicit approval.

### Logging

All feature deliberations are logged to `agent-logs/<feature-name>.md` at the project root. Each log contains:

1. **Feature description** — what was proposed
2. **Round-by-round concerns** — each personality's input per round
3. **Disagreements** — where personalities conflicted
4. **Resolutions** — how conflicts were resolved (by consensus or user decision)
5. **Final plan** — the agreed-upon implementation approach

### When This Applies

This process runs **only when the user enters plan mode** for a new feature. It does not apply to:

- Simple bug fixes
- One-line tweaks
- Questions or explanations
- Work that doesn't involve plan mode

## Branch Workflow

### Plan Mode Creates a Branch

When the user enters plan mode for a feature, **create a new git branch** before any implementation begins:

1. **Branch naming:** Use `feature/<kebab-case-feature-name>` (e.g., `feature/juicy-prestige-animation`).
2. **Create the branch** from `main` after the plan is approved but before writing any code.
3. **All implementation work** for this feature happens on the feature branch.
4. **Commit regularly** on the feature branch as work progresses.

### Post-Implementation Review

After the user confirms the implementation looks good, run a **post-implementation review** using the same five personalities before merging to main. This mirrors the pre-implementation plan review but evaluates the actual code changes.

#### Process

1. **Collect the diff:** Run `git diff main...HEAD` to get all changes on the feature branch.
2. **Parallel review:** Spin up all 5 agents simultaneously. Each receives the full diff and reviews it through their lens:
   - **The Janitor** — Did the implementation introduce duplication, oversized files, or dead code? Is anything left behind that should be cleaned up?
   - **The Godot Guru** — Are Godot patterns correct in the actual code? Proper node lifecycle, signal usage, resource handling, performance?
   - **The Architect** — Do the actual connections match the plan? Any unplanned coupling, missing signal disconnections, or ripple effects?
   - **The Newcomer** — Is the implemented code readable? Magic numbers, unclear names, confusing control flow?
   - **The Consistency Lover** — Does the code match existing codebase patterns? Naming, typing, structure?
3. **Round 1 — Concerns:** Collect and present all concerns, noting which are blocking (must fix before merge) vs. advisory (nice to fix, not required).
4. **Round 2+ — Resolution:** Same multi-round debate as the planning phase. Agents see each other's concerns and resolve conflicts. Up to 3 rounds.
5. **Escalation:** Unresolved disagreements after 3 rounds go to the user.
6. **Fix:** Address all blocking concerns on the feature branch. Advisory concerns are listed but do not block the merge.
7. **Update living documentation:** Before merging, edit the "System Responsibilities" section of this `CLAUDE.md` so it reflects the state of the code on the branch. Scope of the edit:
   - For every system touched, update its ownership/methods/signals/data-flow bullets to match the actual implementation. If a signal was added, renamed, or removed, that change must appear here.
   - Add a new subsection for any new system (autoload, manager, resource, major scene) introduced by the branch.
   - Remove or rewrite subsections for systems that were deleted or fundamentally restructured.
   - Capture cross-system relations explicitly — "X emits foo_changed, which Y and Z listen to" is the kind of line that belongs here.
   - Prefer behavior over implementation detail: readers should come away understanding *what each system owns and how it talks to others*, not line-by-line specifics.

   Commit this documentation update as its own commit (e.g. `docs: update system responsibilities for <feature>`) so the living-docs change is easy to spot in history. If a branch made no meaningful system-level change, explicitly note that in the commit message rather than skipping the check.
8. **Merge:** Once all blocking concerns are resolved and the living docs are updated, merge the feature branch into `main` and delete the feature branch.

#### Logging

Post-implementation reviews are appended to the same `agent-logs/<feature-name>.md` file used during planning, under a `## Post-Implementation Review` heading. This keeps the full lifecycle — plan, concerns, implementation review, and merge — in one place.

#### When This Applies

The post-implementation review runs when:

- Work was done on a feature branch created through plan mode
- The user confirms the implementation is complete and ready for review

It does not run for:

- Work done directly on `main` (bug fixes, tweaks)
- Incomplete work (user hasn't confirmed it's ready)

## Final notes

The old code from the prototype can be found under `deprecated`. This was how things used to work.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kjbrown-sandbox) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
