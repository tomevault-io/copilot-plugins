## apex-trading-bot

> **BEIM SESSION-START:**

# APEX Trading Bot – Claude Code Kontext

## Automatische Anweisungen für Claude Code

**BEIM SESSION-START:**
Der Hook läuft automatisch und zeigt den APEX-Status. Lese ihn vollständig.
Dann prüfe ob offene Positionen oder ausstehende Punkte aus dem letzten Session-Log existieren und weise Andre darauf hin.

**WENN ANDRE SCHREIBT "apex session end":**
1. Fasse die Erkenntnisse dieser Session zusammen (was analysiert, was entschieden, was implementiert)
2. Aktualisiere den Abschnitt "Architektur-Entscheidungen & Session-Log" in dieser Datei
3. Aktualisiere die Memory-Dateien unter `/root/.claude/projects/-root/memory/` wenn sich etwas am Projektstand geändert hat
4. Bestätige kurz was gespeichert wurde

---

## Wer bin ich / Wer ist der User

- **User:** Andre (nicht Christian – das ist der Kollege dessen Repo als Basis diente)
- **Ziel:** Den Bot für Andres eigenes Setup adaptieren und produktiv auf einem Server laufen lassen
- **Sprache:** Deutsch bevorzugt
- **Stil:** Direkt, kein Overhead, kurze Antworten

---

## Was ist APEX

Vollautomatischer Krypto-Trading-Bot mit zwei Strategien:
1. **ORB (Opening Range Breakout)** – Hauptstrategie, Mo–Fr, 3 Sessions/Tag
2. **WeekendMomo** – Wochenend-Momentum für AVAX

Keine KI für Trading-Entscheidungen – rein algorithmische Technische Analyse.

---

## Andres Setup (abweichend vom Kollegen)

| Parameter | Kollege (Original) | Andre |
|---|---|---|
| Exchange | Hyperliquid (DEX) | **Bitget (CEX)** |
| Kapital | ~$2.300 | **50 USDT** |
| Orchestrierung | OpenClaw + 26 Cron-Jobs | **Linux crontab** |
| Auto-Deploy | n8n | **entfällt** |
| Assets | BTC, ETH, SOL, AVAX | **ETH, SOL, AVAX** (BTC zu groß bei 50 USDT) |
| Hebel | variabel | **5x** |
| KI | Gemini via OpenClaw | **nicht nötig** |
| Modus | Live | **DRY_RUN=True** (zum Start) |

**Wichtig:** 2% Risiko pro Trade = $1 bei 50 USDT. Hebel nur um Mindestordergrößen zu erreichen, nicht zum Risiko-Amplify.

---

## Architektur nach Migration

```
apex-trading-bot/
├── CLAUDE.md                  ← Diese Datei
├── config/
│   ├── bot_config.py          ← ZENTRALE KONFIGURATION (DRY_RUN, CAPITAL, LEVERAGE etc.)
│   └── .env.bitget            ← API-Keys (gitignored, muss manuell erstellt werden)
├── scripts/
│   ├── bitget_client.py       ← Haupt-Exchange-Client (ersetzt hyperliquid_client.py)
│   ├── autonomous_trade.py    ← Breakout-Erkennung & Order-Ausführung (Cron-Trigger)
│   ├── weekend_momo.py        ← Weekend-Momentum-Strategie
│   ├── save_opening_range.py  ← Opening Range Box speichern
│   ├── position_monitor.py    ← Offene Positionen überwachen
│   ├── pre_market.py          ← Session-Start Health Check
│   ├── daily_closeout.py      ← Tages-Abschluss Report
│   ├── session_summary.py     ← Session-Ende Summary
│   └── telegram_sender.py     ← Telegram Notifications (unverändert)
├── data/                      ← Laufzeit-Daten (gitignored)
│   ├── opening_range_boxes.json
│   ├── trades.json
│   └── ...
├── logs/                      ← Log-Dateien (gitignored)
└── setup_server.sh            ← Einmaliges Server-Setup-Skript
```

---

## Architektur-Entscheidungen & Session-Log

### Session 2026-04-08 – Code Review: 8 Bugs behoben (Trailing, Atomars, MIN_SIZE)

**Externe Code-Review** identifizierte 8 versteckte Bugs:

**KRITISCH – Trailing-Stop-Kill bei Break-Even:**
- `check_and_apply_break_even()` cancelte mit `cancel_tpsl_orders(pos.coin)` **ALLE** Plan-Orders.
- Das heißt: TP1, TP2 (Trailing), UND SL wurden gelöscht; nur neuer BE-SL kam rein.
- Effekt: Nach 1R Gewinn war dein kompletter Trailing-Stop (Upside-Maschine) tot.
- **Fix:** `cancel_tpsl_orders()` nimmt jetzt optionalen `plan_types`-Filter → nur `["loss_plan"]` canceln.
- Failsafe: Falls Trailing trotzdem weg → aus `trailing_activation` + `trail_pct` aus `trades.json` neu setzen.

**Log-Rotation (truncate vs tail):**
- Alte Variante: `truncate -s 1M` behielt die **ersten** 1MB (älteste Daten).
- Neue Variante: `tail -c 1048576 | mv` behält die **letzten** 1MB (neueste Daten).

**Atomare Writes:**
- `save_opening_range.py`, `update_pnl_tracker()`, `save_state()` schreiben jetzt mit tmp+rename.
- Verhindert JSON-Korruption bei mid-write Crashes.

**MIN_TRADE_SIZE Enforcement:**
- TP1-Split kann bei tiny Sizes zu `0.0` runden (banker's rounding).
- Jetzt: if `size_tp1 < MIN_TRADE_SIZE` → skip TP1, alles ins Trailing.

**API-Call Optimierung:**
- `scan_for_breakouts()` gibt jetzt `(breakout, positions)` Tupel → kein 2. `get_positions()` nötig.

**Commit:** `5a26115` — alle 8 Fixes live, Crontab aktualisiert.

---

### Session 2026-04-07 – Exit-Mechanismus Überarbeitung

**Analyse:** State Pattern vs. Hybrid-Architektur für ORB-Exit-Management

Wir haben eine vorgeschlagene State-Pattern-Architektur (Active → BreakEven → Trailing) analysiert
und gegen die bestehende cron-basierte Architektur abgewogen (Chain of Truth + Devil's Advocate).

**Kernerkenntnisse:**

1. **State Pattern ist konzeptuell richtig**, aber für cron-Betrieb nicht direkt umsetzbar –
   Trailing bei 30-Minuten-Granularität ist de facto ein statischer Stop.

2. **Bitget hat nativen Trailing Stop** (`planType: "moving_plan"`), den wir nicht nutzten.
   Exchange-side Trailing überlebt Server-Abstürze ohne Cancel-Replace-Lücken.

3. **Split-TP implementiert Break-Even bereits implizit** (TP1 bei 1:1 auf halbe Size),
   aber ein expliziter BE-SL-Verschiebung macht den Schutz sauber und Exchange-side.

4. **Short-Handling Bug:** `peak_price` wurde mit `max()` für beide Richtungen getrackt.
   In der BE-Logik fehlte die Short-Richtung vollständig.

5. **Cancel-Replace erzeugt SL-Lücken-Fenster** (~2-5 Sekunden ohne Schutz).
   Jede SL-Aktualisierung = ein Fenster. Native Orders umgehen das.

**Implementierte Verbesserungen (2026-04-07):**

| # | Was | Datei | Warum |
|---|-----|-------|-------|
| 1 | `place_trailing_stop()` (moving_plan) | `bitget_client.py` | Exchange-side Trailing, kein Daemon nötig |
| 2 | TP2 → Trailing Stop bei 2R Aktivierung | `autonomous_trade.py` | Dynamischer Exit statt statischem 3:1 |
| 3 | Break-Even SL-Verschiebung bei 1R | `position_monitor.py` | SL auf Entry+Buffer wenn Trade risikolos |
| 4 | Monitor-Intervall: 30 Min → 5 Min | `crontab_template.txt` | BE-Check braucht feinere Granularität |
| 5 | Short-Direction korrekt in BE-Logik | `position_monitor.py` | SL-Bewegung ist richtungsabhängig |
| 6 | Volume-Logging beim Entry | `autonomous_trade.py` | Breakout-Volumen + 20er-Avg für spätere Analyse |
| 7 | Exit-Logging in trades.json | `position_monitor.py` | PnL, R-Multiple, Exit-Grund pro Trade nachvollziehbar |
| 8 | `apex_status.py` – Session Context Script | `scripts/apex_status.py` | Einbefehl-Kontext: Balance, Trades, P&L, Session-Log |
| 9 | Auto-Hook Session-Start | `~/.claude/settings.json` | Kontext läuft automatisch beim ersten Prompt |
| 10 | Memory-Dateien erstellt | `/root/.claude/projects/-root/memory/` | Projektkontext über Sessions hinweg persistent |

**Entschiedene Nicht-Implementierungen (mit Begründung):**
- **Volume-Filter:** Zu wenig Daten (13 Trades). Erst loggen, dann mit ~30+ Trades evaluieren.
- **State Pattern als Daemon:** Architektursprung zu groß. Bitget native Trailing ist gleichwertig.
- **Häufigeres Polling:** Kein Mehrwert – Trailing läuft exchange-side, BE bei 5 Min ok.

**Qualitycheck-Findings (2026-04-07):**

3 verpasste Trades in 48h – alle durch Bugs verursacht, alle behoben:

| # | Bug | Schwere | Behoben |
|---|-----|---------|---------|
| 1 | `KeyError: 'breakout_size'` (umbenannt zu `breakout_distance`) | Kritisch | ✅ |
| 2 | Direktes JSON-Schreiben ohne tmp+rename (Korruption bei Absturz) | Hoch | ✅ |
| 3 | Kein Telegram-Alert bei position_monitor.py Exception | Hoch | ✅ |
| 4 | BE-Preis Fallback: `current_price=0` wenn API fehlschlägt | Mittel | ✅ |
| 5 | Doppelter `get_positions()` API-Call pro Cron-Run | Mittel | ✅ |

Übersprungen (begründet): Timezone (Server=Berlin ✓), SL-Buffer (by design), Race Condition (Lock schützt), trades.json Format-Mix (legacy, kein Einfluss auf neue Trades)

**Offene Verbesserungsliste (priorisiert):**
1. Volume-Filter evaluieren sobald ~30 Trades mit Volume-Daten vorliegen
2. P&L-Analyse: Winrate, Avg R-Win vs R-Loss nach ausreichend Trade-History
3. `45115` Bitget Preis-Format Fehler beobachten – einmalig aufgetreten, kein Fix nötig
4. Daemon-Architektur wenn Kapital > $200 und Edge bewiesen
5. Funding-Rate Logging (nach Volume-Analyse)

---

## Was bereits vollständig erledigt ist

- [x] `bitget_client.py` geschrieben (HMAC-Auth, Market/SL/TP Orders, DRY_RUN)
- [x] `config/bot_config.py` mit allen Parametern
- [x] Alle 7 Skripte auf Bitget umgestellt
- [x] OpenClaw-Pfade (`/data/.openclaw/...`) entfernt → relative Pfade
- [x] `setup_server.sh` geschrieben (Python venv, Dependencies, Crontab)
- [x] `.gitignore` aktualisiert (`.env.bitget`, `.claude/`, `data/` geschützt)
- [x] Lokal getestet: Marktdaten, Candles, Orderbook ✅
- [x] GitHub Repo: https://github.com/AS90555/apex-trading-bot

---

## Was auf dem Server noch zu tun ist

1. **Repo klonen**
   ```bash
   git clone https://github.com/AS90555/apex-trading-bot.git
   cd apex-trading-bot
   ```

2. **Setup ausführen**
   ```bash
   chmod +x setup_server.sh && ./setup_server.sh
   ```

3. **Timezone prüfen** (muss Europe/Berlin sein!)
   ```bash
   timedatectl
   sudo timedatectl set-timezone Europe/Berlin
   ```

4. **API-Keys eintragen**
   ```bash
   nano config/.env.bitget
   # BITGET_API_KEY=...
   # BITGET_SECRET_KEY=...
   # BITGET_PASSPHRASE=...

   nano .env.telegram
   # TELEGRAM_BOT_TOKEN=...
   # TELEGRAM_CHAT_ID=...
   ```

5. **Testen mit echten Credentials**
   ```bash
   source venv/bin/activate
   python scripts/bitget_client.py     # Marktdaten + Balance
   python scripts/pre_market.py eu     # Vollständiger Check
   ```

6. **Live schalten**
   ```bash
   # In config/bot_config.py:
   # DRY_RUN = False
   ```

---

## Server-Details

- **OS:** Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-90 generic aarch64)
- **User:** root
- **Python:** system python3 + venv unter `./venv/`
- **Crontab:** Eingerichtet via `setup_server.sh`
- **Logs:** `./logs/*.log` (rotation bei >5MB)

---

## Bitget API

- **Dokumentation:** https://www.bitget.com/api-doc/contract/intro
- **Endpoint:** `https://api.bitget.com`
- **Product Type:** `USDT-FUTURES`
- **Auth:** HMAC-SHA256 (API Key + Secret + Passphrase)
- **Benötigte Rechte:** Lesen + Futures-Trading (kein Auszahlen nötig)

---

## Wichtige Konfigurationsparameter (`config/bot_config.py`)

```python
DRY_RUN = True        # ← AUF FALSE SETZEN FÜR LIVE
CAPITAL = 50.0        # USDT
LEVERAGE = 5          # 5x Hebel
MAX_RISK_PCT = 0.02   # 2% = $1 pro Trade
ASSETS = ["ETH", "SOL", "AVAX"]
```

---

## Kritische Hinweise

- **BTC nicht handelbar** bei 50 USDT (Mindestorder > max. Positionsgröße)
- **Telegram** benötigt `.env.telegram` im Projektroot (nicht in `config/`)
- **DRY_RUN immer zuerst** – erst wenn Telegram-Nachrichten korrekt ankommen live schalten
- **Timezone muss Berlin sein** – alle Cron-Zeiten sind Europe/Berlin
- Die Datei `config/.env.bitget` muss manuell auf dem Server erstellt werden (nie in Git!)

---

## Session-Befehle für Claude Code

### Befehl 1: Session START – Kontext wiederherstellen
```bash
! python /root/apex-trading-bot/scripts/apex_status.py
```
Gibt aus: Balance, offene Positionen, aktive Orders, letzte 8 Trades mit Volume-Ratio und P&L, Winrate, Systemstatus.
**Immer als erstes in einer neuen Session ausführen.**

### Befehl 2: Session ENDE – Erkenntnisse sichern
Am Ende einer Session einfach schreiben:
```
apex session end
```
Claude aktualisiert dann `CLAUDE.md` (Architektur-Log) und die Memory-Dateien unter `/root/.claude/projects/-root/memory/` mit den Erkenntnissen der Session.

---

## Häufige Befehle auf dem Server

```bash
# Logs live beobachten
tail -f logs/eu.log
tail -f logs/us.log

# Bot manuell testen
source venv/bin/activate
python scripts/pre_market.py eu
python scripts/save_opening_range.py
python scripts/autonomous_trade.py

# Crontab anzeigen
crontab -l

# Weekend Momo Status
python scripts/weekend_momo.py --status

# Bot-Status (Balance + Positionen)
python -c "from scripts.bitget_client import BitgetClient; c = BitgetClient(dry_run=False); print(c.format_status())"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AS90555)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AS90555)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
