## soccer-telegram-bot

> - Python 3, `python-telegram-bot[webhooks]==21.10`, `pytz==2024.1`

# Soccer Telegram Bot — Claude Context

## Stack
- Python 3, `python-telegram-bot[webhooks]==21.10`, `pytz==2024.1`
- SQLite persistence at `/data/soccer_bot_v2.db` (Fly.io) or `./soccer_bot_v2.db` (local)
- Deployed on Fly.io (port 8080, `/data` volume, 256MB RAM)
- Webhook mode (not polling)

## Key Files
| File | Purpose |
|---|---|
| `bot.py` | Only production file (~2200 lines). Class: `SoccerBotV2` |
| `fly.toml` | Fly.io config |
| `Dockerfile` | Docker build |
| `.env.example` | Only env var: `TELEGRAM_BOT_TOKEN` |

## Deploy (Windows PowerShell)
```powershell
cd soccer-telegram-bot
git pull origin master
flyctl deploy
```

## Database Schema
| Table | Key Columns |
|---|---|
| `settings` | `key`, `value` — legacy global `chat_id` |
| `season` | `id`, `location_name`, `location_link`, `game_day`, `start_time`, `end_time`, `start_date`, `duration_weeks`, `max_players`, `current_week`, `active` |
| `members` | `username` PK NOCASE, `first_name`, `added_at`, `member_type` (`member`/`regular`) |
| `polls` | `id`, `season_id`, `week_number`, `game_date`, `message_id`, `chat_id`, `deadline`, `closed` |
| `votes` | `poll_id`, `user_id`, `username`, `vote_type` (`member_in`/`member_out`/`guest`) |
| `skills` | `username` PK NOCASE, `skill_rating` (1–10), `last_activity` |
| `quickpolls` | `id`, `location_name`, `max_players`, `deadline_time`, `num_teams`, `chat_id`, `admin_id`, `allow_guests`, `telegram_poll_id`, `poll_message_id` |
| `quickpoll_votes` | `poll_id`, `user_id`, `username`, `vote_type` (`in`/`out`/`guest`) |
| `scheduled_events` | `id`, `event_type`, `fire_time`, `payload` JSON, `executed` |
| `chat_admins` | `chat_id`, `user_id`, `username` |
| `chat_groups` | `chat_id` PK, `group_name` |

## Commands (all admin-only, private DM except /setchat)
| Command | Description |
|---|---|
| `/newseason` | 8-step wizard: location, Maps link, day, start/end time, first date (YYYY-MM-DD), weeks, max players |
| `/status` | Show active season info |
| `/testpoll` | Send test poll for week 1 immediately |
| `/cancelgame [week]` | Cancel a season game, post notice to group |
| `/quickpoll` | 10-step wizard: type (Standard/Simple), location, Maps link, date, start/end time, max players, deadline hours, auto-teams, num teams |
| `/closepoll` | Manually close latest quickpoll → admin approval queue |
| `/cancelquickpoll` | Cancel latest quickpoll |
| `/addmember Name` | Add season member |
| `/removemember Name` | Remove season member |
| `/addregular Name` | Add drop-in regular |
| `/removeregular Name` | Remove drop-in regular |
| `/members` | List all members + regulars |
| `/setskill Name Rating` | Set 1–10 skill rating |
| `/skills` | List all rated players |
| `/deleteskill Name` | Delete skill rating |
| `/maketeams [all] [N]` | Make N balanced teams from quickpoll voters (or all rated players with `all`) |
| `/setchat [GroupName]` | Register group (run in group) or link group by ID (run in private) |
| `/addadmin @user or ID` | Add admin for managed group |
| `/removeadmin @user or ID` | Remove admin |
| `/listadmins` | List admins |
| `/listchats` | List all groups you manage |

## Key Internal Functions
| Function | Location | Description |
|---|---|---|
| `is_admin(user_id, chat_id)` | ~line 220 | DB lookup; falls back to username match if user_id is NULL |
| `check_admin(update)` | ~line 230 | Gets chat_id from settings, checks is_admin |
| `get_target_group(update, group_name_arg)` | ~line 250 | Smart group selection for multi-group admins |
| `resolve_chat_context(update, context)` | ~line 270 | Determines target chat: args → current group → most recent → legacy |
| `schedule_event(type, fire_time, payload)` | ~line 310 | Insert into scheduled_events |
| `process_pending_events()` | ~line 1845 | Fires overdue DB events. Called on startup + every 5 min |
| `request_approval(admin_id, text, callback_data)` | ~line 1561 | DM admin with Post/Discard buttons. Returns True/False. Falls back to direct group post on failure |
| `post_roster(poll_id, chat_id, force_send)` | ~line 1596 | Posts player list. Simple poll = plain numbered list. Standard poll = Members + Guests sections |
| `balance_teams(players, num_teams)` | ~line 1728 | Greedy sort-by-skill team balancer |
| `send_quickpoll(...)` | ~line 1486 | Sends info message + native Telegram poll + auto-pins |

## Scheduled Event Types
| Event | Action |
|---|---|
| `send_poll` | Send season poll to group; schedule countdown/reminder/close events |
| `update_countdown` | Edit poll with hours remaining (24h, 12h, 6h, 2h, 1h) |
| `send_reminder` | At -12h: tag non-voters by username |
| `close_poll` | At deadline: post final season roster |
| `close_quickpoll` | At deadline: stop native poll → post_roster → admin approval |
| `finalize_teams` | At deadline: balance teams → admin approval |

## Callback Data Formats
| Pattern | Handler |
|---|---|
| `vote_{season_id}_{week}_member_in/out/guest` | `handle_callback` → `process_vote` |
| `status_{season_id}_{week}` | `handle_callback` → `show_status` |
| `season_renew/modify/stop` | `handle_callback` → `handle_season_action` |
| `qvote_{poll_id}_{type}` | `handle_callback` → `process_quickpoll_vote` |
| `qstatus_{poll_id}` | `handle_callback` → `show_quickpoll_status` |
| `approve/discard:action:poll_id:chat_id` | `handle_approval_callback` |
| `qp_type:standard/simple` | `qp_type_response` (in ConversationHandler) |

## Bugs Fixed (all merged to master)
1. **CallbackQueryHandler wrong pattern** — was `'^qp:'` (matched nothing), now catch-all → season poll voting works
2. **Missing `admin_id` column in quickpolls** — added to schema + INSERT + migration
3. **Duplicate function definitions** — removed second `delete_group_commands` and `delete_message_safely` that overrode the correct ones
4. **`send_nonvoter_reminder` queried removed `user_id` column** — rewrote to match by username
5. **`/addadmin` by @username stored `user_id=0`** — now stores NULL; `is_admin()` falls back to username match + back-fills real ID on first interaction
6. **`finalize_teams` approval posted roster instead of teams** — added `teams` action + `_pending_teams` dict
7. **`/cancelgame` posted to legacy `settings.chat_id`** — fixed to use resolved `target_chat_id`
8. **Timezone-naive datetime comparison** — changed `datetime.now()` to `datetime.now(TZ)` throughout
9. **`/closepoll` said "roster posted!"** — corrected to say "sent for approval"
10. **`/maketeams` backtick formatting** — added `parse_mode='Markdown'` to error messages
11. **`request_approval` silently lost roster if admin hadn't DM'd bot** — now returns True/False; `post_roster` falls back to direct group post on failure
12. **Quickpoll roster display** — Simple polls show plain numbered list; Standard polls show Members + Guests sections (backed by `allow_guests` column)
13. **Roster banner text** — changed to "Damn, you made it 🔥"

## Multi-Group Notes
- Register a group: run `/setchat GroupName` inside the group
- One season is global (no per-group seasons yet)
- `/addadmin` by @username works now (bug #5 fixed)

## Files to Focus On
- `bot.py:2082–2179` — `run()`: all handler registrations
- `bot.py:532–553` — `handle_callback()`: main callback router
- `bot.py:1596–1690` — `post_roster()`: roster rendering
- `bot.py:1728–1831` — `finalize_teams()`: team building
- `bot.py:1845–1896` — `process_pending_events()`: scheduler
- `bot.py:341–366` — `schedule_season_polls()`: season scheduling

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/465300) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
