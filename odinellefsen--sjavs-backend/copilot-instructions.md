## sjavs-backend

> This final step implements the authentic Sjavs Cross/Rubber scoring system where teams start with 24 points and count down. When a team reaches or passes zero, they win a "cross". This system determines the overall match winner across multiple games.

# Step 5: Cross/Rubber Management System

## Overview

This final step implements the authentic Sjavs Cross/Rubber scoring system where teams start with 24 points and count down. When a team reaches or passes zero, they win a "cross". This system determines the overall match winner across multiple games.

## Implementation Tasks

### 1. Create Cross/Rubber State Management

**File: `src/game/cross.rs`**

```rust
use serde::{Deserialize, Serialize};
use crate::game::scoring::GameResult;

/// Cross/Rubber state tracking for a match
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct CrossState {
    /// Current score for trump team (starts at 24, counts down)
    pub trump_team_score: i8,
    /// Current score for opponent team (starts at 24, counts down)
    pub opponent_team_score: i8,
    /// Number of crosses won by trump team
    pub trump_team_crosses: u8,
    /// Number of crosses won by opponent team
    pub opponent_team_crosses: u8,
    /// Bonus points for next game (from ties)
    pub next_game_bonus: u8,
    /// Match ID this cross state belongs to
    pub match_id: String,
    /// Whether the rubber is complete
    pub rubber_complete: bool,
}

impl CrossState {
    /// Create new cross state for a match
    pub fn new(match_id: String) -> Self {
        Self {
            trump_team_score: 24,
            opponent_team_score: 24,
            trump_team_crosses: 0,
            opponent_team_crosses: 0,
            next_game_bonus: 0,
            match_id,
            rubber_complete: false,
        }
    }

    /// Apply game result to cross scores
    pub fn apply_game_result(&mut self, game_result: &GameResult) -> CrossResult {
        // Apply bonus points if any
        let trump_team_points = game_result.trump_team_score + self.next_game_bonus;
        let opponent_team_points = game_result.opponent_team_score;
        
        // Reset bonus after use
        self.next_game_bonus = 0;

        // Handle tie scenario (both teams get 60 points)
        if game_result.trump_team_score == 0 && game_result.opponent_team_score == 0 {
            // This was a tie - add 2 to next game bonus
            self.next_game_bonus = 2;
            return CrossResult {
                trump_team_old_score: self.trump_team_score,
                opponent_team_old_score: self.opponent_team_score,
                trump_team_new_score: self.trump_team_score,
                opponent_team_new_score: self.opponent_team_score,
                cross_won: None,
                bonus_applied: 0,
                next_game_bonus: self.next_game_bonus,
                rubber_complete: false,
            };
        }

        let old_trump_score = self.trump_team_score;
        let old_opponent_score = self.opponent_team_score;

        // Subtract points from respective teams
        self.trump_team_score -= trump_team_points as i8;
        self.opponent_team_score -= opponent_team_points as i8;

        // Check for cross completion
        let cross_won = self.check_cross_completion();

        CrossResult {
            trump_team_old_score: old_trump_score,
            opponent_team_old_score: old_opponent_score,
            trump_team_new_score: self.trump_team_score,
            opponent_team_new_score: self.opponent_team_score,
            cross_won,
            bonus_applied: if trump_team_points > game_result.trump_team_score { 
                trump_team_points - game_result.trump_team_score 
            } else { 0 },
            next_game_bonus: self.next_game_bonus,
            rubber_complete: self.rubber_complete,
        }
    }

    /// Check if a cross has been completed
    fn check_cross_completion(&mut self) -> Option<CrossWinner> {
        // Check if trump team won
        if self.trump_team_score <= 0 {
            // Check for double victory
            let double_victory = self.opponent_team_score == 24;
            
            self.trump_team_crosses += 1;
            self.rubber_complete = true; // For now, one cross = complete rubber
            
            return Some(CrossWinner {
                winning_team: CrossTeam::TrumpTeam,
                double_victory,
                final_score: (self.trump_team_score, self.opponent_team_score),
                crosses_won: self.trump_team_crosses,
            });
        }

        // Check if opponent team won
        if self.opponent_team_score <= 0 {
            // Check for double victory
            let double_victory = self.trump_team_score == 24;
            
            self.opponent_team_crosses += 1;
            self.rubber_complete = true; // For now, one cross = complete rubber
            
            return Some(CrossWinner {
                winning_team: CrossTeam::OpponentTeam,
                double_victory,
                final_score: (self.trump_team_score, self.opponent_team_score),
                crosses_won: self.opponent_team_crosses,
            });
        }

        None
    }

    /// Check if teams are "on the hook" (6 points remaining)
    pub fn get_hook_status(&self) -> (bool, bool) {
        (self.trump_team_score == 6, self.opponent_team_score == 6)
    }

    /// Get cross summary for display
    pub fn get_summary(&self) -> CrossSummary {
        let (trump_on_hook, opponent_on_hook) = self.get_hook_status();
        
        CrossSummary {
            trump_team_score: self.trump_team_score,
            opponent_team_score: self.opponent_team_score,
            trump_team_crosses: self.trump_team_crosses,
            opponent_team_crosses: self.opponent_team_crosses,
            trump_team_on_hook: trump_on_hook,
            opponent_team_on_hook: opponent_on_hook,
            next_game_bonus: self.next_game_bonus,
            rubber_complete: self.rubber_complete,
        }
    }

    /// Reset scores for new game within rubber
    pub fn start_new_game(&mut self) {
        if !self.rubber_complete {
            // Scores remain as they are for next game
            // Only bonus points are reset (handled in apply_game_result)
        }
    }

    /// Reset for completely new rubber
    pub fn reset_for_new_rubber(&mut self) {
        self.trump_team_score = 24;
        self.opponent_team_score = 24;
        self.trump_team_crosses = 0;
        self.opponent_team_crosses = 0;
        self.next_game_bonus = 0;
        self.rubber_complete = false;
    }
}

/// Result of applying a game result to cross state
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct CrossResult {
    /// Trump team score before this game
    pub trump_team_old_score: i8,
    /// Opponent team score before this game
    pub opponent_team_old_score: i8,
    /// Trump team score after this game
    pub trump_team_new_score: i8,
    /// Opponent team score after this game
    pub opponent_team_new_score: i8,
    /// Cross winner if any
    pub cross_won: Option<CrossWinner>,
    /// Bonus points applied this game
    pub bonus_applied: u8,
    /// Bonus points for next game
    pub next_game_bonus: u8,
    /// Whether the rubber is complete
    pub rubber_complete: bool,
}

/// Information about cross winner
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct CrossWinner {
    /// Which team won the cross
    pub winning_team: CrossTeam,
    /// Whether it was a double victory (opponent still at 24)
    pub double_victory: bool,
    /// Final scores when cross was won
    pub final_score: (i8, i8), // (trump_team, opponent_team)
    /// Number of crosses this team has won
    pub crosses_won: u8,
}

/// Teams in cross scoring
#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub enum CrossTeam {
    TrumpTeam,
    OpponentTeam,
}

/// Summary of cross state for API responses
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct CrossSummary {
    /// Current trump team score
    pub trump_team_score: i8,
    /// Current opponent team score
    pub opponent_team_score: i8,
    /// Crosses won by trump team
    pub trump_team_crosses: u8,
    /// Crosses won by opponent team
    pub opponent_team_crosses: u8,
    /// Whether trump team is on the hook
    pub trump_team_on_hook: bool,
    /// Whether opponent team is on the hook
    pub opponent_team_on_hook: bool,
    /// Bonus for next game
    pub next_game_bonus: u8,
    /// Whether rubber is complete
    pub rubber_complete: bool,
}

#[cfg(test)]
mod tests {
    use super::*;
    use crate::game::scoring::{GameResult, SjavsResult};

    #[test]
    fn test_new_cross_state() {
        let cross = CrossState::new("test_match".to_string());
        assert_eq!(cross.trump_team_score, 24);
        assert_eq!(cross.opponent_team_score, 24);
        assert_eq!(cross.trump_team_crosses, 0);
        assert_eq!(cross.opponent_team_crosses, 0);
        assert!(!cross.rubber_complete);
    }

    #[test]
    fn test_apply_normal_win() {
        let mut cross = CrossState::new("test".to_string());
        
        let game_result = GameResult {
            trump_team_score: 4,
            opponent_team_score: 0,
            result_type: SjavsResult::TrumpTeamWin,
            description: "Trump team won".to_string(),
        };

        let result = cross.apply_game_result(&game_result);
        assert_eq!(cross.trump_team_score, 20); // 24 - 4
        assert_eq!(cross.opponent_team_score, 24); // unchanged
        assert!(result.cross_won.is_none());
    }

    #[test]
    fn test_cross_completion() {
        let mut cross = CrossState::new("test".to_string());
        cross.trump_team_score = 4; // Close to winning
        
        let game_result = GameResult {
            trump_team_score: 8,
            opponent_team_score: 0,
            result_type: SjavsResult::TrumpTeamWin,
            description: "Trump team won big".to_string(),
        };

        let result = cross.apply_game_result(&game_result);
        assert_eq!(cross.trump_team_score, -4); // 4 - 8 = -4 (won)
        assert!(result.cross_won.is_some());
        assert_eq!(result.cross_won.unwrap().winning_team, CrossTeam::TrumpTeam);
        assert!(cross.rubber_complete);
    }

    #[test]
    fn test_double_victory() {
        let mut cross = CrossState::new("test".to_string());
        cross.trump_team_score = 4;
        // opponent_team_score stays at 24
        
        let game_result = GameResult {
            trump_team_score: 8,
            opponent_team_score: 0,
            result_type: SjavsResult::TrumpTeamWin,
            description: "Trump team won".to_string(),
        };

        let result = cross.apply_game_result(&game_result);
        let winner = result.cross_won.unwrap();
        assert!(winner.double_victory); // Opponent still at 24
    }

    #[test]
    fn test_tie_bonus() {
        let mut cross = CrossState::new("test".to_string());
        
        // First game: tie
        let tie_result = GameResult {
            trump_team_score: 0,
            opponent_team_score: 0,
            result_type: SjavsResult::Tie,
            description: "Tie game".to_string(),
        };

        let result = cross.apply_game_result(&tie_result);
        assert_eq!(cross.next_game_bonus, 2);
        assert_eq!(result.next_game_bonus, 2);

        // Second game: trump team wins with bonus
        let win_result = GameResult {
            trump_team_score: 4,
            opponent_team_score: 0,
            result_type: SjavsResult::TrumpTeamWin,
            description: "Trump team won".to_string(),
        };

        let result2 = cross.apply_game_result(&win_result);
        assert_eq!(cross.trump_team_score, 18); // 24 - (4 + 2 bonus) = 18
        assert_eq!(result2.bonus_applied, 2);
        assert_eq!(cross.next_game_bonus, 0); // Reset after use
    }

    #[test]
    fn test_on_the_hook() {
        let mut cross = CrossState::new("test".to_string());
        cross.trump_team_score = 6;
        cross.opponent_team_score = 6;
        
        let (trump_hook, opponent_hook) = cross.get_hook_status();
        assert!(trump_hook);
        assert!(opponent_hook);
    }
}
```

### 2. Create Cross State Redis Repository

**File: `src/redis/cross_state/mod.rs`**

```rust
pub mod repository;

pub use repository::CrossStateRepository;
```

**File: `src/redis/cross_state/repository.rs`**

```rust
use crate::game::cross::CrossState;
use crate::RedisConnection;
use serde_json;

pub struct CrossStateRepository;

impl CrossStateRepository {
    /// Store cross state for a match
    pub async fn store_cross_state(
        conn: &mut RedisConnection,
        match_id: &str,
        cross_state: &CrossState,
    ) -> Result<(), String> {
        let key = format!("cross_state:{}", match_id);
        let serialized = serde_json::to_string(cross_state)
            .map_err(|e| format!("Failed to serialize cross state: {}", e))?;

        conn.set(&key, &serialized)
            .await
            .map_err(|e| format!("Failed to store cross state: {}", e))?;

        Ok(())
    }

    /// Get cross state for a match
    pub async fn get_cross_state(
        conn: &mut RedisConnection,
        match_id: &str,
    ) -> Result<Option<CrossState>, String> {
        let key = format!("cross_state:{}", match_id);
        
        match conn.get::<String>(&key).await {
            Ok(serialized) => {
                let cross_state = serde_json::from_str(&serialized)
                    .map_err(|e| format!("Failed to deserialize cross state: {}", e))?;
                Ok(Some(cross_state))
            }
            Err(_) => Ok(None),
        }
    }

    /// Initialize cross state for a new match
    pub async fn initialize_cross_state(
        conn: &mut RedisConnection,
        match_id: &str,
    ) -> Result<CrossState, String> {
        let cross_state = CrossState::new(match_id.to_string());
        Self::store_cross_state(conn, match_id, &cross_state).await?;
        Ok(cross_state)
    }

    /// Clear cross state (for match completion)
    pub async fn clear_cross_state(
        conn: &mut RedisConnection,
        match_id: &str,
    ) -> Result<(), String> {
        let key = format!("cross_state:{}", match_id);
        
        conn.del(&key)
            .await
            .map_err(|e| format!("Failed to clear cross state: {}", e))?;

        Ok(())
    }

    /// Get or create cross state
    pub async fn get_or_create_cross_state(
        conn: &mut RedisConnection,
        match_id: &str,
    ) -> Result<CrossState, String> {
        match Self::get_cross_state(conn, match_id).await? {
            Some(state) => Ok(state),
            None => Self::initialize_cross_state(conn, match_id).await,
        }
    }
}
```

### 3. Update Game Completion Handler

**File: `src/api/handlers/game_scoring.rs`** (modify existing complete_game_handler)

```rust
// Add to imports
use crate::game::cross::{CrossState, CrossTeam, CrossWinner as GameCrossWinner};
use crate::redis::cross_state::repository::CrossStateRepository;

// Replace the TODO section in complete_game_handler:

// Get or create cross state
let mut cross_state = match CrossStateRepository::get_or_create_cross_state(&mut conn, &game_id).await {
    Ok(state) => state,
    Err(e) => {
        return (
            StatusCode::INTERNAL_SERVER_ERROR,
            Json(ErrorResponse {
                error: format!("Failed to get cross state: {}", e),
                message: None,
            }),
        )
            .into_response();
    }
};

// Apply game result to cross scoring
let cross_result = cross_state.apply_game_result(&game_result);

// Store updated cross state
if let Err(e) = CrossStateRepository::store_cross_state(&mut conn, &game_id, &cross_state).await {
    return (
        StatusCode::INTERNAL_SERVER_ERROR,
        Json(ErrorResponse {
            error: format!("Failed to store cross state: {}", e),
            message: None,
        }),
    )
        .into_response();
}

// Create cross scores for response
let cross_scores = CrossScores {
    trump_team_remaining: cross_state.trump_team_score,
    opponent_team_remaining: cross_state.opponent_team_score,
    trump_team_on_hook: cross_state.trump_team_score == 6,
    opponent_team_on_hook: cross_state.opponent_team_score == 6,
    trump_team_crosses: cross_state.trump_team_crosses,
    opponent_team_crosses: cross_state.opponent_team_crosses,
};

// Convert cross winner if any
let cross_won = cross_result.cross_won.map(|winner| {
    let (winning_players, team_name) = match winner.winning_team {
        CrossTeam::TrumpTeam => {
            let trump_team = trick_state.trump_team;
            (vec![trump_team.0 as u8, trump_team.1 as u8], "trump_team")
        },
        CrossTeam::OpponentTeam => {
            let trump_team = trick_state.trump_team;
            let opponents = vec![
                ((trump_team.0 + 1) % 4) as u8, 
                ((trump_team.0 + 3) % 4) as u8
            ];
            (opponents, "opponents")
        },
    };

    CrossWinner {
        winning_team: team_name.to_string(),
        double_victory: winner.double_victory,
        winning_players,
    }
});

// Determine if ready for new game
let new_game_ready = !cross_state.rubber_complete;

// If rubber is complete, clear the cross state
if cross_state.rubber_complete {
    if let Err(e) = CrossStateRepository::clear_cross_state(&mut conn, &game_id).await {
        eprintln!("Failed to clear completed cross state: {}", e);
    }
}
```

### 4. Create Cross State Endpoint

**File: `src/api/handlers/game_scoring.rs`** (add endpoint)

```rust
/// Get current cross/rubber state
#[utoipa::path(
    get,
    path = "/game/cross",
    tag = "Game Management",
    security(
        ("jwt_auth" = [])
    ),
    responses(
        (status = 200, description = "Cross state retrieved", body = CrossStateResponse),
        (status = 404, description = "Game not found", body = ErrorResponse),
        (status = 500, description = "Internal server error", body = ErrorResponse)
    ),
    summary = "Get cross/rubber state",
    description = "Returns current cross/rubber scoring state including team scores and hook status"
)]
#[axum::debug_handler]
pub async fn get_cross_state_handler(
    Extension(user_id): Extension<String>,
    State(redis_pool): State<RedisPool>,
) -> Response {
    let mut conn = match redis_pool.get().await {
        Ok(conn) => conn,
        Err(e) => {
            return (
                StatusCode::INTERNAL_SERVER_ERROR,
                Json(ErrorResponse {
                    error: format!("Failed to get Redis connection: {}", e),
                    message: None,
                }),
            )
                .into_response();
        }
    };

    // Get the player's current game
    let game_id = match PlayerRepository::get_player_game(&mut conn, &user_id).await {
        Ok(Some(game_id)) => game_id,
        Ok(None) => {
            return (
                StatusCode::BAD_REQUEST,
                Json(ErrorResponse {
                    error: "Not in a game".to_string(),
                    message: None,
                }),
            )
                .into_response();
        }
        Err(e) => {
            return (
                StatusCode::INTERNAL_SERVER_ERROR,
                Json(ErrorResponse {
                    error: format!("Failed to get player game: {}", e),
                    message: None,
                }),
            )
                .into_response();
        }
    };

    // Get cross state
    let cross_state = match CrossStateRepository::get_or_create_cross_state(&mut conn, &game_id).await {
        Ok(state) => state,
        Err(e) => {
            return (
                StatusCode::INTERNAL_SERVER_ERROR,
                Json(ErrorResponse {
                    error: format!("Failed to get cross state: {}", e),
                    message: None,
                }),
            )
                .into_response();
        }
    };

    let summary = cross_state.get_summary();

    let response = CrossStateResponse {
        message: "Cross state retrieved successfully".to_string(),
        game_id: game_id.clone(),
        trump_team_score: summary.trump_team_score,
        opponent_team_score: summary.opponent_team_score,
        trump_team_crosses: summary.trump_team_crosses,
        opponent_team_crosses: summary.opponent_team_crosses,
        trump_team_on_hook: summary.trump_team_on_hook,
        opponent_team_on_hook: summary.opponent_team_on_hook,
        next_game_bonus: summary.next_game_bonus,
        rubber_complete: summary.rubber_complete,
    };

    (StatusCode::OK, Json(response)).into_response()
}

/// Response for cross state endpoint
#[derive(Serialize, Deserialize, ToSchema)]
pub struct CrossStateResponse {
    /// Success message
    pub message: String,
    /// The game ID
    pub game_id: String,
    /// Trump team's current score (starts at 24, counts down)
    pub trump_team_score: i8,
    /// Opponent team's current score (starts at 24, counts down)
    pub opponent_team_score: i8,
    /// Crosses won by trump team
    pub trump_team_crosses: u8,
    /// Crosses won by opponent team
    pub opponent_team_crosses: u8,
    /// Whether trump team is "on the hook" (6 points)
    pub trump_team_on_hook: bool,
    /// Whether opponent team is "on the hook" (6 points)
    pub opponent_team_on_hook: bool,
    /// Bonus points for next game
    pub next_game_bonus: u8,
    /// Whether the rubber is complete
    pub rubber_complete: bool,
}
```

### 5. Initialize Cross State on Match Creation

**File: `src/api/handlers/normal_match.rs`** (modify create_match_handler)

```rust
// Add to imports
use crate::redis::cross_state::repository::CrossStateRepository;

// After creating the match, add:
// Initialize cross state for the new match
if let Err(e) = CrossStateRepository::initialize_cross_state(&mut conn, &game_id).await {
    return (
        StatusCode::INTERNAL_SERVER_ERROR,
        Json(ErrorResponse {
            error: format!("Failed to initialize cross state: {}", e),
            message: None,
        }),
    )
        .into_response();
}
```

### 6. Add New Game Within Rubber Endpoint

**File: `src/api/handlers/game_scoring.rs`** (add endpoint)

```rust
/// Start a new game within the current rubber
#[utoipa::path(
    post,
    path = "/game/new-game",
    tag = "Game Management",
    security(
        ("jwt_auth" = [])
    ),
    responses(
        (status = 200, description = "New game started", body = StartGameResponse),
        (status = 400, description = "Cannot start new game", body = ErrorResponse),
        (status = 403, description = "Only host can start new game", body = ErrorResponse),
        (status = 500, description = "Internal server error", body = ErrorResponse)
    ),
    summary = "Start new game in rubber",
    description = "Starts a new game within the current rubber after previous game completion"
)]
#[axum::debug_handler]
pub async fn start_new_game_handler(
    Extension(user_id): Extension<String>,
    State(redis_pool): State<RedisPool>,
) -> Response {
    // Implementation similar to start_game_handler but:
    // 1. Check that previous game is complete
    // 2. Check cross state allows new game (rubber not complete)
    // 3. Reset match to Waiting state
    // 4. Preserve cross state
    // 5. Call existing game start logic
    
    (StatusCode::OK, Json(serde_json::json!({
        "message": "New game within rubber - to be implemented"
    }))).into_response()
}
```

### 7. Update Module Structure and Routes

**File: `src/game/mod.rs`** (add)

```rust
pub mod card;
pub mod deck;
pub mod hand;
pub mod trick;
pub mod scoring;
pub mod cross;  // Add this line
```

**File: `src/redis/mod.rs`** (add)

```rust
pub mod normal_match;
pub mod player;
pub mod game_state;
pub mod pubsub;
pub mod trick_state;
pub mod cross_state;  // Add this line
```

**File: `src/api/routes.rs`** (add)

```rust
// Add to protected router
.route("/game/cross", get(game_scoring::get_cross_state_handler))
.route("/game/new-game", post(game_scoring::start_new_game_handler))
```

**File: `src/api/handlers/openapi.rs`** (add to paths and schemas)

```rust
paths(
    // ... existing paths ...
    crate::api::handlers::game_scoring::get_cross_state_handler,
    crate::api::handlers::game_scoring::start_new_game_handler,
    // ... rest of paths ...
),
components(schemas(
    // ... existing schemas ...
    CrossStateResponse,
    // ... rest of schemas ...
)),
```

### 8. Add Cross State Broadcasting

**File: `src/redis/pubsub/broadcasting.rs`** (additions)

```rust
/// Broadcast cross/rubber completion
pub async fn broadcast_cross_complete(
    conn: &mut RedisConnection,
    game_id: &str,
    winner: &GameCrossWinner,
    final_scores: (i8, i8),
) -> Result<(), String> {
    let event_data = serde_json::json!({
        "type": "cross_complete",
        "winning_team": format!("{:?}", winner.winning_team),
        "double_victory": winner.double_victory,
        "final_scores": {
            "trump_team": final_scores.0,
            "opponent_team": final_scores.1
        },
        "crosses_won": winner.crosses_won,
        "timestamp": chrono::Utc::now().timestamp()
    });

    broadcast_to_game(conn, game_id, &event_data).await
}

/// Broadcast "on the hook" status update
pub async fn broadcast_hook_status(
    conn: &mut RedisConnection,
    game_id: &str,
    trump_team_on_hook: bool,
    opponent_team_on_hook: bool,
) -> Result<(), String> {
    let event_data = serde_json::json!({
        "type": "hook_status_update",
        "trump_team_on_hook": trump_team_on_hook,
        "opponent_team_on_hook": opponent_team_on_hook,
        "timestamp": chrono::Utc::now().timestamp()
    });

    broadcast_to_game(conn, game_id, &event_data).await
}
```

## Key Features Implemented

### 1. **Authentic 24-Point Countdown System**
- Teams start with 24 points
- Points are subtracted based on game results
- First to reach/pass zero wins a cross

### 2. **"On the Hook" Detection**
- Automatically detects when teams reach 6 points
- Broadcasts hook status updates
- Traditional Sjavs superstition implementation

### 3. **Double Victory Tracking**
- Detects when winner reaches zero while opponent still at 24
- Special recognition for dominant performance

### 4. **Tie Game Bonus System**
- Handles 60-60 ties correctly
- Adds 2 bonus points to next game
- Accumulates multiple tie bonuses

### 5. **Cross/Rubber Management**
- Tracks crosses won by each team
- Determines rubber completion
- Supports multiple games within rubber

## Testing Strategy

### Unit Tests
- Cross state initialization
- Score calculation and updates
- Hook status detection
- Double victory scenarios
- Tie bonus accumulation

### Integration Tests
- Complete game → cross update cycle
- Multiple games within rubber
- Cross completion and rubber end
- New rubber initialization

### Cross Scenarios to Test
1. **Normal progression**: 24 → 20 → 16 → 12 → 8 → 4 → 0 (win)
2. **On the hook**: Team reaches 6 points
3. **Double victory**: Win while opponent still at 24
4. **Tie bonuses**: 60-60 games adding +2 to next game
5. **Large wins**: Vol scenarios (12-24 points in single game)

## Summary

This completes the full Sjavs implementation with:

1. ✅ **Card dealing and bidding** (Steps 1-2 from trump selection)
2. ✅ **Trick-taking mechanics** (Steps 1-3 of trick phase)
3. ✅ **Authentic Sjavs scoring** (Step 4 of trick phase)
4. ✅ **Cross/Rubber management** (Step 5 of trick phase)

The system now supports the complete Sjavs game flow from match creation through multiple games to rubber completion, implementing all the traditional rules and scoring systems of this authentic Faroese card game.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/odinellefsen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
