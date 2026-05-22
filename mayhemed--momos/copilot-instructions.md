## momos

> > Read this at the start of every session. Update it after every session.

# MomOS — Project Memory
> Read this at the start of every session. Update it after every session.

## Project Identity
- **Repo:** https://github.com/mayhemed/momos
- **Goal:** Open-source AI care companion for elderly people living alone
- **Owner:** Mark Piesner (mayhemed)
- **AI PM:** Claude Sonnet 4.6
- **Status:** 🟡 Phase 1 Active — Foundation

## Product Philosophy
This is a PRODUCT, not a one-off build. It must:
- Work for ANY family, not just one specific person
- Have a full onboarding wizard (onboarding/setup_wizard.py) on first run
- Generate profile.json at /data/config/profile.json from onboarding
- Pull all personalization from profile.json at runtime (no hardcoded names)
- Be deployable with: curl setup_hub.sh | bash (one command)

## Companion Personality — TV Detective Theme
Companion options: Columbo, Matlock, Jessica (Fletcher), Magnum, Miss Marple, Perry (Mason)
Default: Columbo — warm, curious, "just one more thing" = perfect for daily interview feature
Personas defined in: hub/ollama/companion_styles.json
Style injected into system prompt at runtime from profile.json

## Architecture
- Hub: Beelink Mini S13 N150 16GB, Ubuntu 24.04, Docker Compose
- Voice: Wyoming Satellite (Pi Zero 2W pendant) → Wyoming STT/TTS on hub
- AI: Ollama (llama3.2:3b voice, mistral:7b writing, moondream vision)
- Memory: ChromaDB vector DB (all local)
- Vision: Frigate NVR + Google Coral TPU + Moondream vision LLM
- Sensors: Zigbee2MQTT → MQTT → Home Assistant
- Integration glue: momos-agent conversation proxy (agent/conversation_proxy.py)
- Onboarding: momos-setup service (onboarding/setup_wizard.py) on port 8200

## CRITICAL INTEGRATION INSIGHT
HA voice pipeline → momos-agent:8100/v1 (OpenAI-compatible proxy)
NOT directly to Ollama. This enables memory injection + transcript logging + interview mode.
See: agent/conversation_proxy.py

## Completed Files
- docker-compose.yml (all 12 services)
- .env.example
- hub/homeassistant/configuration.yaml
- hub/homeassistant/automations/ (hydration, emergency, interview, kitchen_appliances)
- hub/mosquitto/mosquitto.conf
- hub/frigate/config.yml
- hub/ollama/persona.md + companion_styles.json
- agent/main.py + conversation_proxy.py
- agent/interview/session_manager.py (40+ question bank)
- agent/journal/transcript_processor.py
- agent/biography/synthesizer.py
- agent/memory/chroma_store.py
- agent/email/weekly_summary.py
- agent/utils/ (config.py, ollama_client.py, mqtt_client.py)
- agent/Dockerfile + requirements.txt
- onboarding/setup_wizard.py + Dockerfile
- scripts/setup_hub.sh (one-command Ubuntu setup)
- wearable/install.sh (Pi Zero Wyoming Satellite setup)
- INTEGRATION_GAPS.md (20 issues documented and resolved)

## Remaining Work
- [ ] hub/frigate/config.yml needs real camera IPs (from .env)
- [ ] agent/conversation_proxy.py needs to load companion_styles.json at runtime
- [ ] onboarding/setup_wizard_ui.html (full HTML UI, large file)
- [ ] scripts/setup_hub.sh — pull Ollama models step
- [ ] wearable/README.md — hardware assembly guide
- [ ] hardware/bom.csv — bill of materials
- [ ] docs/ARCHITECTURE.md
- [ ] Integration tests
- [ ] GitHub Actions CI

## How to Run Sessions
1. "Read CLAUDE.md" → I have full context
2. "Work on [specific file/feature]" → I write real code
3. End of session: I update this file

## Open Decisions
- [ ] Physical hardware purchased? (user action)
- [ ] Ubuntu installed on Beelink?
- [ ] Mom's actual name/profile (comes from onboarding wizard)

---
> Source: [Mayhemed/momos](https://github.com/Mayhemed/momos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
