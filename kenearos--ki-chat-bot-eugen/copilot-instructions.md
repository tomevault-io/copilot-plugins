## ki-chat-bot-eugen

> 1. [Überblick](#überblick)

# Eugen – Intelligenter Gaming & 3D-Druck Twitch-Bot
## Komplettes Setup & Implementierungs-Guide für Windows

---

## 📋 Inhaltsverzeichnis

1. [Überblick](#überblick)
2. [Systemanforderungen](#systemanforderungen)
3. [Installation & Setup](#installation--setup)
4. [Konfigurationsassistent](#konfigurationsassistent)
5. [GUI Dashboard](#gui-dashboard)
6. [Architektur & Implementierung](#architektur--implementierung)
7. [API-Integration & Debugging](#api-integration--debugging)
8. [Fehlerbehandlung](#fehlerbehandlung)
9. [Projektstruktur](#projektstruktur)

---

## Überblick

**Eugen** ist ein intelligenter Twitch-Chat-Agent für:
- **Gaming**: World of Warcraft, Elden Ring, Gamedev
- **3D-Druck**: Prusa i3, Bambu, Creality
- **Tech**: Python, Linux, Home Automation

### Kernfeatures

| Feature | Beschreibung |
|---------|-------------|
| **Name Recognition** | Erkennt automatisch wenn angesprochen (@Eugen, Eugen:, etc.) |
| **Persistent Memory** | Speichert Chat-History pro User (max 25 Nachrichten) |
| **Context-Aware** | Antwortet basierend auf vorherigem Gesprächsverlauf |
| **Perplexity Integration** | Nutzt Perplexity Sonar für Echtzeit-Web-Suche |
| **Live Monitoring** | GUI zeigt alle API-Calls, Responses, Fehler in Echtzeit |
| **Windows-Native** | Vollständige Windows-Unterstützung, keine Linux-Tools nötig |

---

## Systemanforderungen

### Minimal
- Windows 10/11
- Python 3.9+
- 100 MB Festplatte
- Internetverbindung

### Empfohlen
- Python 3.11+
- 500 MB Festplatte (für Chat-History & Logs)
- Stabiles Netzwerk (für IRC & API)

---

## Installation & Setup

### Schritt 1: Python installieren

1. Gehe zu [python.org](https://www.python.org/downloads/)
2. Lade **Python 3.11+** herunter
3. **WICHTIG**: Häkchen setzen bei "Add Python to PATH"
4. Installieren

**Verifizieren:**
```powershell
python --version
pip --version
```

### Schritt 2: Projekt initialisieren

```powershell
# Neuer Ordner für Eugen
mkdir C:\Users\YourUsername\eugen
cd C:\Users\YourUsername\eugen

# Virtual Environment erstellen
python -m venv venv

# Aktivieren (Windows PowerShell)
.\venv\Scripts\Activate.ps1

# Falls Fehler bei Ausführungsrichtlinie:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
# Dann nochmal: .\venv\Scripts\Activate.ps1
```

### Schritt 3: Dependencies installieren

```powershell
pip install -r requirements.txt
```

**requirements.txt:**
```
perplexity-python-client==1.0.0
irc==20.1.0
python-dotenv==1.0.0
PySimpleGUI==4.60.0
requests==2.31.0
```

---

## Konfigurationsassistent

### Setup-Flow (Automatisch beim ersten Start)

Beim ersten Start wird ein **interaktives Setup-Fenster** angezeigt:

```
╔════════════════════════════════════════════════════════════════╗
║               EUGEN KONFIGURATIONSASSISTENT                    ║
╠════════════════════════════════════════════════════════════════╣
║                                                                ║
║  1️⃣  TWITCH KONFIGURATION                                     ║
║  ───────────────────────────────────────────────────────────  ║
║  Bot-Name:           [___________________________]             ║
║  OAuth Token:        [***hidden***] [🔑 Anleitung]            ║
║  Channel:            [___________________________]             ║
║                                                                ║
║  2️⃣  PERPLEXITY KONFIGURATION                                ║
║  ───────────────────────────────────────────────────────────  ║
║  API Key:            [***hidden***] [🔑 Anleitung]            ║
║  Model:              [sonar-pro ▼]                            ║
║  Max Tokens:         [450]                                     ║
║                                                                ║
║  3️⃣  BOT-VERHALTEN                                            ║
║  ───────────────────────────────────────────────────────────  ║
║  ☑ Context Memory aktivieren                                 ║
║  ☑ Name Recognition aktivieren                               ║
║  ☑ Debug-Mode (zeigt API-Calls)                             ║
║                                                                ║
║  [Speichern & Starten]  [Abbrechen]                           ║
║                                                                ║
╚════════════════════════════════════════════════════════════════╝
```

### API-Keys beschaffen

#### Twitch OAuth Token

1. Gehe zu [twitchtokengenerator.com](https://twitchtokengenerator.com/) **ODER** manuell:
   - [Twitch Developer Console](https://dev.twitch.tv/console/apps)
   - Neue Anwendung erstellen
   - OAuth-Authentifizierung
   - Scopes: `chat:read`, `chat:edit`
   - Token generieren

2. **Token Format:** `oauth:abcd1234efgh5678...`
3. **Speichern in Setup-Fenster**

**⚠️ WICHTIG:** Token niemals ins Repository committen!

#### Perplexity API Key

1. Gehe zu [perplexity.ai/api](https://www.perplexity.ai/api)
2. Registrieren / Anmelden
3. API Keys → Neuen Key generieren
4. **Format:** `pplx-abcd1234efgh5678...`
5. **Speichern in Setup-Fenster**

---

## GUI Dashboard

### Live Monitoring Interface

Während der Bot läuft, zeigt das Dashboard in Echtzeit:

```
╔════════════════════════════════════════════════════════════════════╗
║                    EUGEN BOT - LIVE DASHBOARD                     ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  STATUS: 🟢 VERBUNDEN (Kanal: #dein_kanal)                        ║
║  Uptime: 00:45:23 | Messages: 12 | Errors: 0                    ║
║                                                                    ║
║  ┌────────────────────────────────────────────────────────────┐  ║
║  │ LIVE CHAT AKTIVITÄT                                        │  ║
║  ├────────────────────────────────────────────────────────────┤  ║
║  │ 12:02:15 | User123: Eugen, was ist dein Lieblings-Game?   │  ║
║  │ 12:02:16 | [API] → Perplexity (sonar-pro)                 │  ║
║  │ 12:02:18 | [RESPONSE] Mein Lieblings-Game ist WoW...      │  ║
║  │ 12:02:19 | Eugen: @User123 Mein Lieblings-Game ist WoW... │  ║
║  │                                                            │  ║
║  │ 12:03:05 | User456: Eugen, wie levelt man schnell?        │  ║
║  │ 12:03:06 | [CONTEXT] Gefundener History (3 msgs)          │  ║
║  │ 12:03:06 | [API] → Perplexity (sonar-pro)                 │  ║
║  │ 12:03:08 | [RESPONSE] Die besten Leveling-Spots sind...   │  ║
║  │ 12:03:09 | Eugen: @User456 Die besten Leveling-Spots...   │  ║
║  └────────────────────────────────────────────────────────────┘  ║
║                                                                    ║
║  ┌─────────────────────┬─────────────────────────────────────┐  ║
║  │ API STATISTIKEN     │ FEHLER LOG                          │  ║
║  ├─────────────────────┼─────────────────────────────────────┤  ║
║  │ Requests: 12        │ (Leer - alles OK!)                  │  ║
║  │ Avg Response: 620ms │                                     │  ║
║  │ Costs: $0.0036      │                                     │  ║
║  │ Erfolgsrate: 100%   │                                     │  ║
║  └─────────────────────┴─────────────────────────────────────┘  ║
║                                                                    ║
║  [Settings] [Clear Logs] [Export Data] [Stop Bot]                 ║
║                                                                    ║
╚════════════════════════════════════════════════════════════════════╝
```

### Dashboard-Tabs

#### 1. **Live Feed**
Zeigt Echtzeit-Aktivität:
- Chat-Messages
- API-Calls (Prompt → Perplexity)
- Responses von der KI
- Fehler & Warnungen

#### 2. **API Debug**
Detaillierte API-Logs mit:
```
[12:02:16] REQUEST SENT
├─ Endpoint: chat.completions
├─ Model: sonar-pro
├─ Messages: 4
├─ Prompt: "was ist dein Lieblings-Game?"
├─ Max Tokens: 450
├─ Temperature: 0.7
└─ Timestamp: 2026-01-02 12:02:16

[12:02:18] RESPONSE RECEIVED
├─ Status: 200 OK
├─ Tokens Used: 127
├─ Response: "Mein Lieblings-Game ist WoW wegen..."
├─ Processing Time: 1.834s
└─ Cost: $0.0003
```

#### 3. **User Statistics**
- Nutzer mit den meisten Interaktionen
- Top-Topics (Gaming, 3D-Druck, Tech)
- Chat-History pro User anschauen
- Context-Memory Status

#### 4. **Settings**
- API-Keys ändern
- Model auswählen
- Max Tokens anpassen
- Debug-Mode togglen
- Auto-Reconnect konfigurieren

---

## Architektur & Implementierung

### Dateisystem

```
C:\Users\YourUsername\eugen\
├── chatbot.py              # Hauptprogramm
├── config.py               # Config-Management
├── gui.py                  # Dashboard GUI
├── ai_provider.py          # Perplexity Integration
├── memory.py               # Conversation Memory
├── utils.py                # Hilfsfunktionen
├── requirements.txt        # Dependencies
├── .env                    # Secrets (NICHT committen!)
│
├── data/
│   ├── conversations/      # User Chat-Histories
│   │   ├── user1.json
│   │   ├── user2.json
│   │   └── ...
│   └── config.json         # Gespeicherte Konfiguration
│
└── logs/
    ├── eugen.log           # Hauptlogfile
    └── api_debug.log       # API Debug-Logs
```

### Kernklassen

#### `MentionDetector`
```python
class MentionDetector:
    """Erkennt ob Bot angesprochen wurde"""
    
    def is_mentioned(self, message: str) -> bool:
        """True wenn @Eugen, Eugen:, eugen, etc. erwähnt"""
    
    def extract_content(self, message: str) -> str:
        """Extrahiert Nachricht ohne Mention"""
```

#### `ConversationMemory`
```python
class ConversationMemory:
    """Speichert & lädt Chat-History pro User"""
    
    def get_user_history(self, username: str) -> list:
        """Letzte 5 Messages für Context"""
    
    def add_message(self, username: str, role: str, content: str):
        """Speichert Message (user/assistant)"""
    
    def format_for_prompt(self, history: list) -> str:
        """Konvertiert zu Prompt-Format"""
```

#### `PerplexityProvider`
```python
class PerplexityProvider:
    """Kommunikiert mit Perplexity API"""
    
    async def get_response(self, prompt: str, history: list) -> str:
        """Sendet Prompt + History, empfängt Response"""
    
    def validate_api_key(self, key: str) -> bool:
        """Prüft ob API-Key gültig ist"""
```

#### `EugenBot` (Main)
```python
class EugenBot:
    """Orchestriert alles: IRC, Memory, AI, GUI"""
    
    async def on_chat_message(self, username: str, message: str):
        """Wird aufgerufen wenn Chat-Message ankommt"""
    
    def send_response(self, channel: str, username: str, response: str):
        """Sendet Antwort im Chat"""
    
    def log_event(self, event_type: str, data: dict):
        """Loggt alles für Debug-Dashboard"""
```

---

## API-Integration & Debugging

### Was passiert beim Chat?

```
1. Chat-Message empfangen
   ↓
2. [MentionDetector] → Wurde Bot angesprochen?
   ↓ Ja
3. [ConversationMemory] → Lade letzte 5 Messages dieses Users
   ↓
4. [Prompt Builder]
   ├─ System Prompt: "Du bist Eugen, Gamer & 3D-Druck Experte"
   ├─ Context History: Letzte Konversation
   └─ Current Message: Die neue Frage
   ↓
5. [API Call] → Perplexity Sonar
   ├─ Method: POST /chat/completions
   ├─ Model: sonar-pro
   ├─ Messages: 4 (system, history, history, user)
   └─ Max Tokens: 450
   ↓
6. [Perplexity Processing] ~600-1000ms
   ↓
7. [Response] → KI-Antwort (z.B. "Mein Lieblings-Game ist WoW...")
   ↓
8. [ConversationMemory] → Speichere in user.json
   ↓
9. [IRC Send] → Sende im Chat: "@Username KI-Antwort"
   ↓
10. [Dashboard Update] → Zeige alles in Live-Feed
```

### Debug-Mode aktivieren

In der GUI oder `.env`:
```
DEBUG_MODE=true
```

Dann wird in der CLI ausgegeben:
```
[DEBUG] 12:02:16 - Message received from User123
[DEBUG] 12:02:16 - Mention detected: @Eugen
[DEBUG] 12:02:16 - Content extracted: "was ist dein Lieblings-Game?"
[DEBUG] 12:02:16 - History loaded: 3 messages
[DEBUG] 12:02:16 - Building prompt...
[DEBUG] 12:02:16 - SYSTEM: "Du bist Eugen..."
[DEBUG] 12:02:16 - HISTORY[0]: User: "Wie geht es dir?"
[DEBUG] 12:02:16 - HISTORY[1]: Assistant: "Mir geht's gut!"
[DEBUG] 12:02:16 - HISTORY[2]: User: "Spielst du WoW?"
[DEBUG] 12:02:16 - CURRENT: "was ist dein Lieblings-Game?"
[DEBUG] 12:02:16 - Sending to Perplexity API...
[DEBUG] 12:02:16 - REQUEST: {"model": "sonar-pro", "messages": [...], ...}
[DEBUG] 12:02:18 - RESPONSE: 200 OK
[DEBUG] 12:02:18 - Content: "Mein Lieblings-Game ist WoW..."
[DEBUG] 12:02:18 - Tokens used: 127 (cost: $0.0003)
[DEBUG] 12:02:18 - Saving to memory...
[DEBUG] 12:02:18 - Sending IRC: "@User123 Mein Lieblings-Game ist WoW..."
[DEBUG] 12:02:19 - Message sent successfully
```

### API-Keys validieren

Setup-Fenster prüft automatisch:

```python
# Twitch Token
def validate_twitch_token(token: str) -> bool:
    """Test mit Twitch IRC Connection"""
    try:
        irc.client.IRC().connect(
            "irc.chat.twitch.tv", 6667,
            nickname, token=token
        )
        return True
    except:
        return False

# Perplexity API Key
def validate_perplexity_key(key: str) -> bool:
    """Test mit einfachem API Call"""
    try:
        response = requests.post(
            "https://api.perplexity.ai/chat/completions",
            headers={"Authorization": f"Bearer {key}"},
            json={
                "model": "sonar-pro",
                "messages": [{"role": "user", "content": "test"}],
                "max_tokens": 10
            },
            timeout=5
        )
        return response.status_code == 200
    except:
        return False
```

---

## Fehlerbehandlung

### Häufige Fehler & Lösungen

#### 1. **"Invalid Twitch Token"**
```
Fehler: Setup schlägt fehl
Grund: Token abgelaufen oder falsch

Lösung:
1. Gehe zu twitchtokengenerator.com
2. Generiere neuen Token
3. Kopiere: oauth:... (mit oauth: Prefix!)
4. Paste in Setup-Fenster
5. Validierung sollte grün werden
```

#### 2. **"Perplexity API Error 401"**
```
Fehler: Bot startet, aber keine Responses
Grund: Ungültiger API-Key oder Account-Issue

Lösung:
1. Prüfe API-Key auf perplexity.ai
2. Stelle sicher Account hat Credits (kostenlos gibt es $5/Monat)
3. Copy-paste genau (ohne Spaces)
4. Teste mit Setup-Validierung
```

#### 3. **"Cannot connect to Twitch IRC"**
```
Fehler: Bot startet nicht
Grund: Netzwerk oder Twitch offline

Lösung:
1. Prüfe Internet-Verbindung
2. Teste: ping irc.chat.twitch.tv
3. Firewall-Regel für Python hinzufügen
4. Auto-Reconnect wartet 10 Sekunden, versucht erneut
5. Dashboard zeigt Reconnect-Versuche
```

#### 4. **"Memory File Corrupted"**
```
Fehler: Chat-History wird nicht geladen
Grund: JSON-Datei beschädigt

Lösung:
1. Dashboard → Settings → "Reset Memory"
2. Oder manuell: Lösche data/conversations/username.json
3. Nächster Chat-Start erstellt neue Datei
```

#### 5. **"Response Rate Limited"**
```
Fehler: Lange Wartezeiten, Timeout
Grund: Zu viele API-Calls zu schnell

Lösung:
1. Perplexity hat Rate-Limit pro Account
2. Warte 1-2 Sekunden zwischen Messages
3. Dashboard zeigt "Rate Limit - Waiting..."
4. Nach 60 Sekunden Auto-Retry
```

### Error Dashboard

Alle Fehler werden im Dashboard gesammelt:

```
┌─────────────────────────────────────────┐
│ FEHLER (LETZTE 24H)                     │
├─────────────────────────────────────────┤
│                                         │
│ ❌ 12:05:22 | Perplexity Timeout       │
│    Message von User123                  │
│    → Auto-Retry in 10 Sekunden         │
│                                         │
│ ❌ 11:45:15 | API Rate Limited         │
│    10 Requests in 5 Sekunden           │
│    → Auto-Wait aktiviert               │
│                                         │
│ [Clear Errors] [Export Report]          │
│                                         │
└─────────────────────────────────────────┘
```

---

## Projektstruktur

### Wichtigste Dateien für Programmiere r

#### `chatbot.py` (Hauptprogramm)
```python
import asyncio
from config import Config
from memory import ConversationMemory
from ai_provider import PerplexityProvider
from gui import Dashboard
from utils import MentionDetector, Logger

class EugenBot:
    def __init__(self):
        self.config = Config()
        self.memory = ConversationMemory()
        self.ai = PerplexityProvider(self.config.perplexity_key)
        self.detector = MentionDetector()
        self.logger = Logger()
        self.dashboard = Dashboard(self)
    
    async def run(self):
        # IRC Connection
        # Listen for messages
        # Process & respond
        # Update dashboard
```

#### `config.py` (Konfiguration)
```python
import os
import json
from pathlib import Path
from dotenv import load_dotenv

class Config:
    def __init__(self):
        load_dotenv()
        
        self.twitch_token = os.getenv("TWITCH_OAUTH_TOKEN")
        self.twitch_channel = os.getenv("TWITCH_CHANNEL")
        self.bot_name = os.getenv("TWITCH_BOT_NICKNAME", "Eugen")
        
        self.perplexity_key = os.getenv("PERPLEXITY_API_KEY")
        self.model = os.getenv("PERPLEXITY_MODEL", "sonar-pro")
        self.max_tokens = int(os.getenv("MAX_TOKENS", "450"))
        
        self.debug_mode = os.getenv("DEBUG_MODE", "false").lower() == "true"
```

#### `ai_provider.py` (Perplexity API)
```python
import httpx
import json
from typing import List, Dict

class PerplexityProvider:
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = "https://api.perplexity.ai"
        self.model = "sonar-pro"
    
    async def get_response(self, messages: List[Dict]) -> str:
        """
        Sendet Messages zu Perplexity, erhält Response
        
        Args:
            messages: [
                {"role": "system", "content": "..."},
                {"role": "user", "content": "..."},
                ...
            ]
        
        Returns:
            Response-Text von der KI
        """
        headers = {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json"
        }
        
        payload = {
            "model": self.model,
            "messages": messages,
            "max_tokens": 450,
            "temperature": 0.7
        }
        
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{self.base_url}/chat/completions",
                json=payload,
                headers=headers,
                timeout=30
            )
        
        if response.status_code == 200:
            data = response.json()
            return data['choices'][0]['message']['content']
        else:
            raise Exception(f"API Error {response.status_code}: {response.text}")
```

#### `memory.py` (Chat-Speicher)
```python
import json
from pathlib import Path
from datetime import datetime, timedelta
from typing import List, Dict

class ConversationMemory:
    def __init__(self, max_messages=25, data_dir="data/conversations"):
        self.data_dir = Path(data_dir)
        self.data_dir.mkdir(parents=True, exist_ok=True)
        self.max_messages = max_messages
    
    def get_user_history(self, username: str, limit: int = 5) -> List[Dict]:
        """Lädt letzte Chat-Messages für User"""
        file_path = self.data_dir / f"{username.lower()}.json"
        
        if not file_path.exists():
            return []
        
        with open(file_path, 'r', encoding='utf-8') as f:
            history = json.load(f)
        
        # Nur Messages der letzten 1 Stunde
        cutoff_time = datetime.now() - timedelta(hours=1)
        recent = [
            msg for msg in history[-limit:]
            if datetime.fromisoformat(msg['timestamp']) > cutoff_time
        ]
        
        return recent
    
    def add_message(self, username: str, role: str, content: str):
        """Speichert eine Message (role: 'user' oder 'assistant')"""
        file_path = self.data_dir / f"{username.lower()}.json"
        
        history = []
        if file_path.exists():
            with open(file_path, 'r', encoding='utf-8') as f:
                history = json.load(f)
        
        history.append({
            "role": role,
            "content": content,
            "timestamp": datetime.now().isoformat()
        })
        
        # Max Nachrichten-Limit einhalten
        history = history[-self.max_messages:]
        
        with open(file_path, 'w', encoding='utf-8') as f:
            json.dump(history, f, ensure_ascii=False, indent=2)
    
    def format_for_prompt(self, history: List[Dict]) -> List[Dict]:
        """Konvertiert History zu Message-Format für AI-API"""
        return [
            {
                "role": msg['role'],
                "content": msg['content']
            }
            for msg in history
        ]
```

#### `gui.py` (Dashboard)
```python
import PySimpleGUI as sg
import threading
from datetime import datetime

class Dashboard:
    def __init__(self, bot):
        self.bot = bot
        sg.theme('DarkBlue3')
        
        # Event-Queue für Thread-Safe Updates
        self.event_queue = []
    
    def log_event(self, event_type: str, data: Dict):
        """Protokolliert Event für Dashboard"""
        timestamp = datetime.now().strftime("%H:%M:%S")
        
        if event_type == "chat_message":
            msg = f"{timestamp} | {data['username']}: {data['content']}"
        elif event_type == "api_call":
            msg = f"{timestamp} | [API] → {data['model']}"
        elif event_type == "api_response":
            msg = f"{timestamp} | [RESPONSE] {data['content'][:50]}..."
        elif event_type == "error":
            msg = f"{timestamp} | ❌ {data['error']}"
        
        self.event_queue.append(msg)
    
    def render(self):
        """Rendet GUI-Fenster"""
        layout = [
            [sg.Text("EUGEN BOT - LIVE DASHBOARD", font=("Arial", 16, "bold"))],
            [sg.Text(f"Status: 🟢 VERBUNDEN", key="-STATUS-")],
            [sg.Multiline(
                size=(120, 30),
                key="-LOG-",
                disabled=True,
                autoscroll=True
            )],
            [sg.Button("Settings"), sg.Button("Clear"), sg.Button("Stop")]
        ]
        
        window = sg.Window("Eugen Bot", layout)
        
        while True:
            event, values = window.read(timeout=100)
            
            # Update Log
            if self.event_queue:
                for msg in self.event_queue:
                    window["-LOG-"].print(msg)
                self.event_queue = []
            
            if event == sg.WINDOW_CLOSED or event == "Stop":
                break
        
        window.close()
```

#### `.env` (Secrets)
```
# Twitch Configuration
TWITCH_OAUTH_TOKEN=oauth:xxxxxxxxxxxxx
TWITCH_BOT_NICKNAME=Eugen
TWITCH_CHANNEL=#dein_kanal

# Perplexity Configuration
PERPLEXITY_API_KEY=pplx-xxxxxxxxxxxxx
PERPLEXITY_MODEL=sonar-pro
MAX_TOKENS=450

# Bot Configuration
DEBUG_MODE=true
AUTO_RECONNECT=true
RECONNECT_DELAY=10

# Data Configuration
DATA_DIR=data/conversations
LOG_DIR=logs
CONTEXT_RETENTION_HOURS=1
```

---

## Quick Start für Windows

### Installation (5 Minuten)

```powershell
# 1. Python installiert? Wenn nicht: python.org
python --version

# 2. Projekt clonen/downlaoden
cd C:\Users\YourUsername\eugen

# 3. Virtual Environment
python -m venv venv
.\venv\Scripts\Activate.ps1

# 4. Dependencies
pip install -r requirements.txt

# 5. Konfigurieren
python chatbot.py
# → Setup-Fenster öffnet sich
# → API-Keys eingeben
# → Validierung läuft
# → Speichern & Starten

# 6. Dashboard öffnet sich mit Live-Feed
```

### Fehlersuche

**Problem: Module nicht gefunden?**
```powershell
# Sicherstellen dass venv aktiviert ist:
.\venv\Scripts\Activate.ps1

# Dann reinstallieren:
pip install --upgrade pip
pip install -r requirements.txt
```

**Problem: Port 6667 blocked?**
```powershell
# Firewall-Exception für Python:
# Windows Defender → "Firewall & Netzwerkschutz"
# → "App durch Firewall zulassen"
# → Python hinzufügen
```

---

## Verbesserungen & Erweiterungen

### Einfache Adds (für später)

- [ ] Multi-Channel Support (mehrere Kanäle gleichzeitig)
- [ ] Custom Commands (!gaming, !3dprint)
- [ ] User-Blacklist (bestimmte User ignorieren)
- [ ] Response-Timeout (wenn API zu langsam)
- [ ] Discord Webhook (errori an Discord schicken)
- [ ] Analytics Export (CSV mit Stats)

### Für Programmiere r: Custom Features

Neue Commands hinzufügen:
```python
class CommandHandler:
    def handle_command(self, cmd: str, args: str) -> str:
        if cmd == "gaming":
            return "Tipps für WoW, Elden Ring..."
        elif cmd == "3dprint":
            return "3D-Druck Hilfe..."
```

---

## Support & Debugging

Bei Fehlern:

1. **Prüfe Dashboard** → "Fehler" Tab
2. **Debug-Mode aktivieren** → `.env` → `DEBUG_MODE=true`
3. **Logs anschauen** → `logs/eugen.log`
4. **API Key validieren** → Setup-Fenster erneut öffnen
5. **Internet prüfen** → ping google.com

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kenearos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
