## arena-domain

> WYD Royal Arena domain rules


# Royal Arena Domain Rules

# Arena Types

There are two arenas:

## Champions Hall
- unrestricted arena
- celestial level >= 300
- no PR restriction

## Aspirants Field
- celestial level >= 300
- has Power Rating restrictions
- intended for balanced progression
- players above level 400 gain no EXP

# Badge Rules

There are two badges:
- Badge Aspirant
- Badge Champion

A player can only participate in ONE arena per season.

Rules:
- player chooses one badge
- changing from Aspirant to Champion is allowed
- changing from Champion to Aspirant is NOT allowed
- switching to Champion resets all Aspirant CS
- new badge selection only next season

# Combat Score Rules

Each arena registration:
- consumes 1 CS

Each victory:
- grants 7 CS

Defeat:
- grants 0 CS

CS can NEVER become negative.

Example:
- player has 0 CS
- first victory grants 7 CS
- registration consumes 1 CS
- resulting visible CS = 6

# Ranking Tiers

Exiled:
- 0 to 4 CS

Warrior:
- 5 to 25 CS

Knight:
- 26 to 51 CS

Master:
- 52 to 77 CS

Master of War:
- 78 to 120 CS

GrandMaster:
- 121 to 250 CS

# Arena Match Structure

Each arena match contains:
- 4 teams
- each team can contain up to 13 players
- maximum 52 players per arena match

Rules:
- only ONE team wins the arena
- all winning team members receive rewards
- only players alive for at least 1 minute qualify for rewards

Reward eligibility:
- receive Combat Score (CS)
- receive Royal Arena Coupons

Players that die before surviving 1 minute:
- do NOT receive rewards
- do NOT receive CS
- do NOT receive coupons

# Winning Team Tracking

The system must identify the winning team for every arena match.

The ranking system must:
- highlight winning team members
- detect new victories using:
  currentWinCount - previousWinCount = 1

Maximum highlighted winners per match:
- 13 players

# Winner Feed

The system must support:
- listing winning players
- congratulating winning teams
- generating winner announcements
- historical winner tracking

# Seasons

- seasons are monthly
- CS resets every season
- rankings reset every season
- badge state resets every season

# Reward Rules

Victory rewards Royal Arena Coupons based on ranking.

Defeat always gives:
- 5 coupons

# Arena Schedule Rules

There are exactly 4 arena windows per day.

Arena times (Brasília timezone):
- 13:00
- 19:00
- 20:30
- 23:00

Timezone:
- America/Sao_Paulo

Thursday Rule:
- Thursday 13:00 arena NEVER exists
- Thursday 13:31 collection NEVER executes

# Snapshot Collection Rules

The system must ONLY collect arena snapshots:
- exactly 31 minutes after each arena starts

Collection schedule:
- 13:31
- 19:31
- 21:01
- 23:31

The system must NEVER continuously poll the API.

Reason:
- avoid unnecessary database growth
- avoid redundant snapshots
- reduce processing overhead
- improve event reconstruction accuracy

# Snapshot Integrity Rules

Only one official snapshot collection is allowed per arena window.

Duplicate collections for the same arena window must be prevented.

Each snapshot must be associated with:
- arena date
- arena scheduled time
- arena type
- collection timestamp

# Retry Rules

If snapshot collection fails during an official arena window,
the system may retry for a limited grace period.

Retries must:
- remain associated with the same arena window
- never create duplicate official snapshots

# Timezone Rules

All system operations must use:
- Brasília timezone
- America/Sao_Paulo

Never use:
- UTC directly in business logic
- server local timezone

All arena scheduling calculations must be timezone-aware.

# Snapshot-Based Data Source

All arena data comes from:

GET https://mochila.tapiocahut.com/royal-arena

The API returns snapshot-based data.

The system does NOT receive:
- direct match events
- team information
- timestamps for matches
- winner events

The system must infer arena events by comparing snapshots over time.

# Snapshot Structure

{
  "champion": [],
  "aspirant": []
}

Each player object:

{
  "charName": string,
  "class": number,
  "subClass": number,
  "wins": number,
  "kills": number,
  "deaths": number,
  "points": number
}

# Event Reconstruction Rules

The system uses heuristic-based event reconstruction.

Arena teams are inferred from synchronized player progression.

Winning team detection is probabilistic and based on:
- simultaneous win increases
- synchronized point changes
- synchronized kill/death updates

# Historical Integrity

Arena results are append-only.

Never overwrite finalized match results.

Corrections must generate adjustment records instead of mutating history.

# System Priorities

The ranking system must:
- preserve historical snapshots
- support leaderboard generation
- support ranking history
- support future analytics
- avoid inconsistent CS calculations
- support replay analysis

---
> Source: [davidzaque-leal/royal-arena](https://github.com/davidzaque-leal/royal-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
