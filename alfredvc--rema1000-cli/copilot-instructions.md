## rema1000-cli

> All commands output JSON to stdout. Errors go to stderr as `{"error": "..."}`.

# REMA 1000 CLI — Agent Reference

## Quick Start

```bash
# Authenticate (one-time, requires browser)
rema auth login

# Verify auth
rema auth status
```

## Commands

All commands output JSON to stdout. Errors go to stderr as `{"error": "..."}`.

### Global Flags
- `--json` — compact JSON (overrides TTY pretty-printing)
- `--fields f1,f2` — return only these fields (reduces tokens)

### Authentication
| Command | Description |
|---------|-------------|
| `rema auth login` | Browser-based phone auth |
| `rema auth status` | Auth state and token expiry |
| `rema auth logout` | Delete tokens |

### Read Commands (GET)
| Command | Auth | Description |
|---------|------|-------------|
| `rema version` | No | App version and maintenance status |
| `rema customer get` | Yes | Full customer profile |
| `rema customer minimal` | Yes | Bonus balance + primary store |
| `rema offers list` | Yes | Offers, missions, price cuts |
| `rema campaigns list [--type SP]` | Yes | Sales campaigns with products |
| `rema articles search <query> [--rows N]` | Yes | Product search |
| `rema articles image <gtin> [--output file]` | Yes | Product image (binary) |
| `rema stores get <workplace-id>` | Yes | Store details |
| `rema stores search <query>` | Yes | Search stores |
| `rema stores hours <store-number>` | Yes | Weekly opening hours |
| `rema transactions list` | Yes | Transaction history headers |
| `rema transactions get <transaction-id>` | Yes | Line items + payments for one transaction |
| `rema lists list` | Yes | All shopping lists |
| `rema lists get <list-id>` | Yes | List with items |
| `rema lists image <list-id> [--output file]` | Yes | List custom image (binary) |

### Write Commands (POST/PUT)
| Command | Description |
|---------|-------------|
| `rema customer update --home-store <id>` | Set home store |
| `rema lists create <name> [--emoji X] [--theme COLOR]` | Create list |
| `rema lists delete <list-id>` | Delete list |
| `rema lists share <list-id> --name <name>` | Generate share link |
| `rema items add <list-id> <name> [--gtin X] [--description X] [--quantity N] [--store-area N]` | Add item |
| `rema items update <list-id> <item-id> [--state Active\|Inactive] [--name X] [--quantity N]` | Update item |
| `rema items delete <list-id> <item-id>` | Delete item |
| `rema items clear <list-id>` | Remove checked-off items |
| `rema items activate <list-id>` | Uncheck all items |

### Typical Agent Workflow

```bash
# Search for a product, then add to list
rema articles search "melk" --rows 5 --json
# Use gtin from result to add to list
rema items add LIST-UUID "Helmelk 1L" --gtin 7038010009457 --description "1 L"
```

---
> Source: [Alfredvc/rema1000-cli](https://github.com/Alfredvc/rema1000-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
