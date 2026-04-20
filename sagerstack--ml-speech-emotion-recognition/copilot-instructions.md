## ml-speech-emotion-recognition

> - Use the artifact-generator skill when asked to generate specific artifacts, like user story, implementation plan

# Behavioral Rules for CLAUDE

### Thought-Process
- Use the artifact-generator skill when asked to generate specific artifacts, like user story, implementation plan
- Spend sufficient tokens to plan a task with the user before jumping to solution. Ask clarifying questions ONE AT A TIME

### Decision-Making Preferences
- Prioritize accuracy over speed
- Always suggest security improvements
- Prefer existing patterns over new approaches

### Coding Instructions
 - 1. When writing Python code, always use poetry for dependency and package management
 - 2. When writing Python code, always use the python-developer subagent

#### Coding Behaviors
 - 1. Always write incremental code and test. Identify a logical stop when sufficient code has been written that should be tested before further development. There is little value in writing a significant amount of code without testing. 

### Deployment Strategy
1) make sure the app is able to run locally without docker. for frontend and backend services, they should be able to communicate if both services are launched\
2) for docker- each service should have its own docker file and then a docker-compose file for full-stack\. make sure docker-compose works and the app is able to execute successfully and services can connect to each other
3) the docker images should be pushed to docker hub
4) minikube should pull these images, deploy and run. again, make sure the entire app is running 

## Important
- Ensure strict adherence to the instructions provided. Non-compliance will result in failure 


# Machine Learning Speech to Emotion Recognition Project

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a machine learning project for speech emotion recognition using the **CREMA-D (Crowd-sourced Emotional Multimodal Actors Dataset)**. CREMA-D is a comprehensive dataset designed for training and evaluating emotion recognition systems.

## Dataset Details

### CREMA-D Overview
- **Total audio clips**: 7,442 original clips
- **Number of actors**: 91 actors (48 male, 43 female)
- **Age range**: 20-74 years with diverse ethnic backgrounds
- **Languages**: Multiple languages represented
- **Data collection**: Crowdsourced with 2,443 participants rating clips
- **Rating modalities**: Audio-only, video-only, and audio-visual
- **Reliability**: 95% of clips have more than 7 ratings

### Dataset Structure
The dataset contains:
- **12 sentences** spoken per actor
- **6 basic emotions**: Anger, Disgust, Fear, Happy, Neutral, Sad
- **4 emotion levels**: Low, Medium, High, Unspecified (where applicable)
- **12 recording series** representing different methodologies

## Project Structure

```
ml-speech-emotion-recognition/
├── data/
│   └── AudioWAV/           # CREMA-D audio files (7,442 WAV files)
│       ├── {ID}_DFA_{EMOTION}_XX.wav  # Dialogue-based emotion recordings
│       ├── {ID}_IEO_{EMOTION}_{LEVEL}.wav  # Interactive emotion with intensity
│       ├── {ID}_ITS_{EMOTION}_XX.wav  # Emotional statements
│       ├── {ID}_IOM_{EMOTION}_XX.wav  # Interactive method
│       ├── {ID}_IWL_{EMOTION}_XX.wav  # Whispered loudness
│       ├── {ID}_TAI_{EMOTION}_XX.wav  # Targeted affect induction
│       ├── {ID}_TSI_{EMOTION}_XX.wav  # Targeted emotional statements
│       ├── {ID}_IWW_{EMOTION}_XX.wav  # Whispered words
│       ├── {ID}_TIE_{EMOTION}_XX.wav  # Targeted intensity expression
│       ├── {ID}_MTI_{EMOTION}_XX.wav  # Method training induction
│       ├── {ID}_WSI_{EMOTION}_XX.wav  # Whispered speech intensity
│       └── {ID}_ITH_{EMOTION}_XX.wav  # Intensity training high
├── src/
│   └── notebooks/         # Jupyter notebooks for development
│       └── feature-ext.ipynb  # Feature extraction and dataset loading
├── project-files/
│   └── Machine Learning Project Proposal.pdf
└── .gitignore              # Standard Python gitignore
```

## Audio Data Format

### File Naming Convention
All audio files follow the pattern: `{ActorID}_{SeriesCode}_{EmotionCode}_{LevelCode}.wav`

**Series Codes:**
- **DFA**: Dialogue-based emotional acting
- **IEO**: Interactive emotion (only series with explicit intensity levels)
- **ITS**: Emotional statements
- **IOM**: Interactive method
- **IWL**: Whispered loudness
- **TAI**: Targeted affect induction
- **TSI**: Targeted emotional statements
- **IWW**: Whispered words
- **TIE**: Targeted intensity expression
- **MTI**: Method training induction
- **WSI**: Whispered speech intensity
- **ITH**: Intensity training high

**Emotion Codes:**
- **ANG**: Angry
- **DIS**: Disgusted
- **FEA**: Fearful
- **HAP**: Happy
- **NEU**: Neutral
- **SAD**: Sad

**Intensity/Level Codes:**
- **HI**: High intensity (IEO series only)
- **MD**: Medium intensity (IEO series only)
- **LO**: Low intensity (IEO series only)
- **XX**: Unspecified intensity (all other series)

### Dataset Statistics
- **Balanced emotions**: ~1,271 samples each for angry, disgusted, fearful, happy, sad
- **Neutral emotion**: 1,087 samples
- **Intensity distribution**: 6,532 medium, 455 high, 455 low
- **Speaker diversity**: Each emotion expressed by multiple actors

## Development Environment

- Use Poetry for Python dependency management and virtual environments
- Always prefer `poetry run` for executing Python commands
- The project follows standard Python development patterns based on .gitignore

## Common Development Tasks

Since this is a new ML project, typical tasks would include:

1. **Setting up the environment**:
   ```bash
   poetry init
   poetry install
   ```

2. **Data exploration and preprocessing**:
   - Audio files are in `data/AudioWAV/`
   - Consider using librosa, pydub, or similar for audio processing
   - Extract features like MFCCs, spectrograms, etc.

3. **Model development**:
   - Consider frameworks: TensorFlow/Keras, PyTorch, or scikit-learn
   - Implement audio preprocessing pipelines
   - Train emotion classification models

4. **Experimentation**:
   - Use Jupyter notebooks for iterative development
   - Track experiments with MLflow or similar tools

## Git Workflow

- Work on feature branches, never directly on main
- Main branch is protected - create pull requests for changes
- Always merge latest main changes into your feature branch before creating PRs to avoid conflicts

## CREMA-D Research & Licensing

### Research Papers
The dataset methodology and validation are detailed in:
- **IEEE Transactions on Affective Computing** (2014)
- **Behavior Research Methods** (2015)

### License
- **Open Database License (ODbL)**: For the database structure
- **Database Contents License (DbCL)**: For the actual data content
- **Repository**: https://github.com/CheyneyComputerScience/CREMA-D

## Next Steps for Project Setup

The project appears to need:
- Poetry configuration (pyproject.toml)
- Requirements definition for audio processing and ML libraries (librosa, pydub, tensorflow/pytorch)
- Source code structure for models, data processing, and utilities
- Experiment tracking and model versioning setup
- Audio feature extraction pipelines (MFCCs, spectrograms, chroma features)
- Data splitting strategies (speaker-independent evaluation recommended)
- for all spec-kit commands and interactions, use ENGLISH as the communication language ONLY

## Active Technologies
- Python 3.11+, TypeScript 5.x + FastAPI, Uvicorn, WebSockets, Librosa, Boto3, SageMaker SDK, Pydantic, Streamlit, Reac (001-model-deployment-api)
- S3 for temporary audio files, Redis for rate limiting (optional), in-memory for session state (001-model-deployment-api)

## Recent Changes
- 001-model-deployment-api: Added Python 3.11+, TypeScript 5.x + FastAPI, Uvicorn, WebSockets, Librosa, Boto3, SageMaker SDK, Pydantic, Streamlit, Reac
- use github cli (gh cli) for git commands
- always use python-developer subagent when writing python code and tests
- for all streamlit python code changes, use subagent streamlit-developer
- for streamlit app code changes, always run the streamlit app before you respond back to me, check the frontend for any runtime errors and fix them
- for streamlit app, always launch on port 8510
- always check streamlit official documentation for referencing HOW to implement a capability or feature in streamlit- https://docs.streamlit.io/develop/api-reference
- after you launch the app, wait for 15 seconds to monitor for any errors in the logs
- always sleep for 15 seconds after launching app. not 5
- never create documentation files unless I ask you to do so
- never create guides unless i ask you to do so
- always deploy the local k8s stack using local-deploy.sh and include monitoring
- always deploy the local k8s stack using local-deploy.sh and include monitoring
- never create test python/shell scripts for temporary usage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sagerstack) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
