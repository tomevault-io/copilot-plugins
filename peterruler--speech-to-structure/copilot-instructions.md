## speech-to-structure

> - [x] **Verify that the copilot-instructions.md file in the .github directory is created** ✓ Erstellt

# Speech-to-Structure - Copilot Instructions

## Projekt Status

- [x] **Verify that the copilot-instructions.md file in the .github directory is created** ✓ Erstellt
- [x] **Clarify Project Requirements** ✓ Node.js App mit Whisper-Tiny, GPT-OSS:20B, Web-GUI für Audio-Aufnahme
- [x] **Scaffold the Project** ✓ Server.js, HTML-GUI, package.json erstellt
- [x] **Customize the Project** ✓ Audio-Aufnahme, Whisper-Integration, GPT-OSS:20B-Integration implementiert
- [x] **Install Required Extensions** ✓ Alle Dependencies installiert (Express, Multer, FFmpeg, etc.)
- [x] **Compile the Project** ✓ Pakete installiert und konfiguriert
- [x] **Create and Run Task** ✓ npm start Skript verfügbar
- [x] **Launch the Project** ✓ Server läuft auf Port 3000
- [x] **Ensure Documentation is Complete** ✓ Vollständige README.md mit Screenshots

## Technologie Stack

### Backend
- **Node.js/Express.js**: Main server application
- **Python faster-whisper**: Lokale Speech-to-Text (244MB small model)
- **FFmpeg**: Audio-Konvertierung zu 16kHz mono WAV
- **Ollama GPT-OSS:20B**: Lokale Text-zu-JSON Strukturierung (13GB)

### Frontend
- **Vanilla HTML/CSS/JavaScript**: Web-Interface
- **Web Audio API**: Audio-Aufnahme im Browser
- **MediaRecorder**: Audio-Recording

### Integration
- **Multer**: File-Upload Handling
- **Axios**: HTTP-Client für Ollama API
- **child_process**: Python-Integration
- **fluent-ffmpeg**: Audio-Processing

## Projekt Features

- ✅ **Web-GUI Audio-Aufnahme**: Direkte Aufnahme über Browser
- ✅ **Datei-Upload**: Upload bestehender Audio-Dateien (WAV, MP3, M4A, etc.)
- ✅ **Lokale Speech-to-Text**: Python faster-whisper Integration
- ✅ **Audio-Konvertierung**: Automatische FFmpeg-Konvertierung 
- ✅ **Intelligente KI-Strukturierung**: GPT-OSS:20B mit optimierten Prompts
- ✅ **Medizinische Datenextraktion**: Vorname, Nachname, Alter, Geschlecht, Blutdruck, etc.
- ✅ **Automatische Bereinigung**: Audio-Dateien werden nach Verarbeitung gelöscht
- ✅ **Fallback-Mechanismen**: Intelligentes Text-Parsing bei JSON-Fehlern
- ✅ **Vollständige Dokumentation**: README mit Installation und Troubleshooting

## Installation & Setup

1. **Node.js Dependencies**: `npm install`
2. **Python Environment**: Python venv mit faster-whisper
3. **FFmpeg Installation**: Audio-Processing
4. **Ollama Setup**: gpt-oss:20b Model (13GB)
5. **Environment Configuration**: .env Setup

## Aktueller Status

**🟢 VOLLSTÄNDIG IMPLEMENTIERT**
- Alle Hauptfunktionen sind implementiert und getestet
- Server läuft stabil auf Port 3000
- Lokale Modelle sind konfiguriert und funktional
- Dokumentation ist vollständig
- MIT-Lizenz hinzugefügt

## Nächste Schritte (Optional)

- [ ] **Performance Optimierung**: Caching für häufige Anfragen
- [ ] **Erweiterte Medizinische Kategorien**: Zusätzliche Gesundheitsdaten
- [ ] **Multi-Language Support**: Weitere Sprachen für Whisper
- [ ] **Database Integration**: Persistente Speicherung von Ergebnissen
- [ ] **User Authentication**: Benutzer-Management System

## Troubleshooting Guide

### Python Environment Issues
```bash
source .venv/bin/activate
pip install faster-whisper
```

### Ollama Connection Issues
```bash
ollama serve
ollama list
curl http://localhost:11434/api/tags
```

### Audio Processing Issues
```bash
ffmpeg -version
which ffmpeg
```

## Development Notes

- **Virtual Environment**: `.venv` für Python Dependencies
- **Environment Variables**: `.env` für Konfiguration
- **Audio Cleanup**: Automatische Bereinigung implementiert
- **Error Handling**: Comprehensive error handling mit Fallbacks
- **Medical Data**: Fokus auf deutsche medizinische Terminologie

---
> Source: [peterruler/speech-to-structure](https://github.com/peterruler/speech-to-structure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
