## multi-timer-app

> A customizable multi-timer web application that allows users to create and run multiple timers simultaneously with flexible interval patterns.

# Timer App - Development Documentation

## Project Overview
A customizable multi-timer web application that allows users to create and run multiple timers simultaneously with flexible interval patterns.

## Key Features
- **Custom Time Intervals**: Set any duration for work/break periods (not limited to standard Pomodoro 25/5)
- **Multiple Concurrent Timers**: Run multiple independent timers simultaneously
- **Flexible Repetition Patterns**: Create custom sequences (e.g., 25/5, 25/5, 25/5, 25/15)
- **Infinite Repeat**: Timers can run infinitely (default) or for a set number of cycles
- **Custom Notifications**: Separate notification messages for work and break completions
- **Local Storage**: All timer configurations persist between sessions
- **Desktop Notifications**: System-level notifications with sound alerts

## Technical Stack
- **Frontend**: Vanilla JavaScript, HTML5, CSS3
- **Storage**: Browser LocalStorage API
- **Notifications**: Web Notifications API
- **Audio**: Web Audio API for notification sounds

## Project Structure
```
timer/
├── index.html          # Main HTML structure
├── timer.js           # Timer class with core logic
├── app.js            # Timer manager and UI interactions
├── styles.css        # Styling and responsive design
└── CLAUDE.md         # This documentation
```

## Core Components

### Timer Class (timer.js)
- Manages individual timer state and intervals
- Handles interval progression and cycle counting
- Supports infinite repeat mode
- Serializable for storage persistence

### TimerManager Class (app.js)
- Manages multiple timer instances
- Handles UI updates and user interactions
- Manages local storage operations
- Controls notifications and audio feedback

## Local Development
```bash
# Start local server
python3 -m http.server 8080

# Access at
http://localhost:8080
```

## Deployment
- **GitHub**: Repository for version control
- **Vercel**: Static site hosting with automatic deploys from GitHub

## Browser Compatibility
- Modern browsers supporting:
  - LocalStorage API
  - Notifications API
  - Web Audio API
  - CSS Grid and Flexbox

## Future Enhancements
- Timer templates/presets
- Statistics and analytics
- Import/export timer configurations
- Keyboard shortcuts
- Dark mode toggle
- Mobile app version

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evannaderi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/evannaderi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
