## qualityleap

> Database schema design and model definitions for QualityLeap platform

# QualityLeap - Database Models & Schema

## Technology Choice
- **Primary Database**: PostgreSQL 15+
- **Reason**: ACID compliance critical for transaction integrity (XP, currency, rewards)
- **Cache Layer**: Redis for leaderboards, sessions, pub/sub

## Core Entities

### User Model
Represents call center employees and their progression state.

**Key Fields**:
- `id` (UUID, primary key)
- `email` (unique, indexed)
- `first_name`, `last_name`
- `avatar_url`
- `team_id` (foreign key → Team)
- `role` (enum: 'agent', 'supervisor', 'manager', 'admin')
- `current_level` (integer, default: 1)
- `current_xp` (integer, default: 0)
- `total_xp` (integer, default: 0)
- `currency_balance` (integer, default: 0) // Quality Coins
- `created_at`, `updated_at`
- `last_login_at`
- `is_active` (boolean)

**Indexes**: `email`, `team_id`, `current_level`

### Action Model
Records all business actions (calls, cases, QA submissions) that trigger gamification.

**Key Fields**:
- `id` (UUID, primary key)
- `user_id` (foreign key → User)
- `action_type` (enum: 'CALL_COMPLETED', 'QA_SUBMITTED', 'CSAT_RECEIVED', etc.)
- `metadata` (JSONB) // Flexible storage for action-specific data
- `xp_awarded` (integer)
- `timestamp` (timestamptz)
- `source_system` (string) // e.g., 'ACD', 'QA_SOFTWARE'
- `external_id` (string) // Reference to source system record

**Indexes**: `user_id`, `action_type`, `timestamp`, `external_id`

### Achievement Model (Badge Definitions)
Defines available badges and their unlock criteria.

**Key Fields**:
- `id` (UUID, primary key)
- `name` (string, unique)
- `description` (text)
- `category` (enum: 'performance', 'quality', 'consistency', 'development', 'collaboration')
- `tier` (enum: 'bronze', 'silver', 'gold', 'platinum', null)
- `image_url` (string)
- `criteria` (JSONB) // Structured rules for earning badge
- `xp_reward` (integer)
- `currency_reward` (integer)
- `is_active` (boolean)
- `display_order` (integer)

**Indexes**: `category`, `is_active`

### UserAchievement Model (Bridge Table)
Tracks which users have earned which badges.

**Key Fields**:
- `id` (UUID, primary key)
- `user_id` (foreign key → User)
- `achievement_id` (foreign key → Achievement)
- `earned_at` (timestamptz)
- `progress` (JSONB, nullable) // For progressive badges (e.g., streak count)
- `is_pinned` (boolean, default: false)

**Indexes**: `user_id`, `achievement_id`, composite unique index on `(user_id, achievement_id)`

### Reward Model (Catalog Items)
Defines available rewards that can be purchased with Quality Coins.

**Key Fields**:
- `id` (UUID, primary key)
- `name` (string)
- `description` (text)
- `category` (enum: 'tangible', 'experiential', 'social')
- `cost` (integer) // Quality Coins required
- `image_url` (string)
- `stock_quantity` (integer, nullable) // null = unlimited
- `is_active` (boolean)
- `requires_approval` (boolean) // Manager approval needed?
- `display_order` (integer)

**Indexes**: `category`, `is_active`

### Transaction Model (Audit Trail)
Critical for financial integrity - records all XP and currency movements.

**Key Fields**:
- `id` (UUID, primary key)
- `user_id` (foreign key → User)
- `transaction_type` (enum: 'XP_EARNED', 'CURRENCY_EARNED', 'CURRENCY_SPENT', 'BADGE_EARNED', 'REWARD_REDEEMED')
- `amount` (integer) // Can be negative for deductions
- `balance_after` (integer) // Snapshot of balance after transaction
- `reference_type` (string) // e.g., 'Action', 'Reward', 'Achievement'
- `reference_id` (UUID) // ID of related record
- `description` (text)
- `metadata` (JSONB)
- `created_at` (timestamptz)

**Indexes**: `user_id`, `transaction_type`, `created_at`
**Note**: This table is append-only; never delete records

### Quest Model (P1)
Defines daily quests and weekly missions.

**Key Fields**:
- `id` (UUID, primary key)
- `name` (string)
- `description` (text)
- `quest_type` (enum: 'daily', 'weekly', 'special')
- `criteria` (JSONB) // Completion requirements
- `xp_reward` (integer)
- `currency_reward` (integer)
- `starts_at`, `ends_at` (timestamptz)
- `is_active` (boolean)

### UserQuest Model (P1)
Tracks user progress on quests.

**Key Fields**:
- `id` (UUID, primary key)
- `user_id` (foreign key → User)
- `quest_id` (foreign key → Quest)
- `progress` (JSONB) // Current progress state
- `status` (enum: 'active', 'completed', 'expired')
- `started_at`, `completed_at` (timestamptz)

### Challenge Model (P1)
Manager-created custom competitions.

**Key Fields**:
- `id` (UUID, primary key)
- `name` (string)
- `description` (text)
- `created_by` (foreign key → User)
- `target_kpi` (enum: 'FCR', 'CSAT', 'AHT', 'QA_SCORE', etc.)
- `participant_type` (enum: 'individual', 'team', 'all')
- `participants` (JSONB or relation table)
- `start_date`, `end_date` (timestamptz)
- `success_criteria` (JSONB)
- `rewards` (JSONB)
- `is_active` (boolean)

### Team Model
Represents call center teams for team-based leaderboards.

**Key Fields**:
- `id` (UUID, primary key)
- `name` (string)
- `description` (text)
- `manager_id` (foreign key → User)
- `is_active` (boolean)

### Kudos Model (P1)
Peer-to-peer recognition system.

**Key Fields**:
- `id` (UUID, primary key)
- `from_user_id` (foreign key → User)
- `to_user_id` (foreign key → User)
- `message` (text)
- `value_type` (enum: 'helpful', 'empathy', 'innovation', 'teamwork')
- `points_given` (integer, default: 1)
- `created_at` (timestamptz)

## Database Best Practices

### Transaction Management
- Use database transactions for multi-step operations
- Example: XP award + level up + badge unlock = single transaction
- Ensure ACID compliance for all currency operations
- Implement optimistic locking for concurrent updates

### Audit Trail
- Never delete records from Transaction table
- Use soft deletes (is_active/deleted_at) where appropriate
- Log all sensitive operations
- Timestamp all records (created_at, updated_at)

### Performance Optimization
- Create appropriate indexes (shown above)
- Use JSONB indexes for frequently queried JSON fields
- Partition large tables if needed (e.g., Action table by date)
- Use materialized views for complex leaderboard queries
- Cache frequently accessed data in Redis

### Data Integrity
- Use foreign key constraints
- Add CHECK constraints for valid ranges (e.g., level between 1 and 100)
- Use NOT NULL constraints where appropriate
- Validate data at application layer AND database layer
- Use database triggers sparingly (prefer application logic)

### Migrations
- Use migration tool (e.g., Knex.js, TypeORM migrations, Sequelize)
- Always write reversible migrations (up/down)
- Test migrations on copy of production data
- Never modify existing migrations after deployment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emirluffy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/emirluffy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
