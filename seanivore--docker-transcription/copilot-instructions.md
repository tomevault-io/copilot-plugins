## docker-transcription

> A self-hosted speech-to-text transcription service using OpenAI's Whisper AI and Docker. This service monitors a directory for audio files, transcribes them automatically, and saves the results in both text and JSON formats.

# Docker Transcriptions Project Map

## Project Overview
A self-hosted speech-to-text transcription service using OpenAI's Whisper AI and Docker. This service monitors a directory for audio files, transcribes them automatically, and saves the results in both text and JSON formats.

## Directory Structure
```text
/docker-transcriptions
├── docker-compose.yml     # Main Docker configuration
├── app/                   # Application code
│   └── transcribe.py      # Python script for transcription
├── data/                  # Persistent storage for uploaded files and transcriptions
│   ├── uploads/           # Audio files to be transcribed
│   │   └── processed/     # Audio files that have been processed
│   └── transcriptions/    # Generated transcription files
└── docs/                  # Project documentation
    └── PROJECT_MAP.md     # This file
```

## Component Relationships
- Docker container running Python with Whisper
- Local filesystem for file monitoring and storage
- Automatic transcription of audio files

## Key Files
### Core Components
- `docker-compose.yml`: Main configuration file that defines the Docker service
- `app/transcribe.py`: Python script that handles transcription

### Data
- `data/uploads/`: Directory to place audio files for transcription
- `data/transcriptions/`: Directory where transcription results are saved

### Documentation
- `docs/PROJECT_MAP.md`: Overview of the project structure and workflow

## Integration Points
- Local filesystem for input and output
- Audio files as input (MP3, WAV, M4A, etc.)
- Text and JSON files as output

## Development Workflow
1. Modify docker-compose.yml or transcribe.py as needed
2. Run/restart the Docker container
3. Place audio files in the uploads directory
4. Check transcription results in the transcriptions directory

## Usage Workflow
1. Start the Docker container
2. Copy or move audio files to the data/uploads directory
3. Wait for automatic transcription
4. Find transcription results in data/transcriptions directory
5. Original audio files are moved to data/uploads/processed

## Deployment Architecture
- Runs locally via Docker
- File-based monitoring and processing
- Data persistence through Docker volumes
- No external API dependencies (fully self-contained)

## Technical Requirements
- Docker and Docker Compose installed
- Sufficient disk space for audio files and models
- Sufficient RAM for running Whisper AI models
- Network connectivity for initial Docker image download

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seanivore)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/seanivore)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
