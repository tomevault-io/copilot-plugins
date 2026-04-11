## harmonicizer

> Here is a comprehensive design and code roadmap to build the Harmonicizer web app, providing a structured approach for an LLM to generate the application.

## Design and Code Roadmap for Harmonicizer Web App

Here is a comprehensive design and code roadmap to build the Harmonicizer web app, providing a structured approach for an LLM to generate the application.

### 1. Project Setup and Core Technologies

**Objective:** Establish the foundational technology stack and project structure.

**Key Technologies:**
*   **React:** For building the user interface with functional components and hooks.
*   **React Three Fiber:** To create the 3D interactive "Explorer" mode.
*   **Tone.js:** For all audio-related functionalities, including synthesis and scheduling.
*   **TonalJS:** A music theory library to manage chords, notes, and their relationships.
*   **r3f-forcegraph:** A React Three Fiber component for creating force-directed graphs, which will be the core of the "Explorer" mode.

**Roadmap:**

1.  **Initialize React Project:** Use `create-react-app` or a similar tool to set up a new React project.
2.  **Install Dependencies:**
    ```bash
    npm install three @react-three/fiber @react-three/drei tone tonal r3f-forcegraph
    ```
3.  **Project Structure:** Organize the project into logical folders:
    *   `src/components`: For reusable UI components.
    *   `src/modes`: To house the "Composer" and "Explorer" mode components.
    *   `src/audio`: For managing Tone.js related logic.
    *   `src/theory`: For music theory calculations using TonalJS.
    *   `src/state`: To manage the application's state.
    *   `src/utils`: For helper functions.

### 2. The Composer Mode

**Objective:** Create an intuitive interface for building and playing chord progressions.

#### 2.1. UI Components

**Roadmap:**

1.  **`Composer` Component:** The main container for this mode.
2.  **`ChordGrid` Component:** A grid layout to display and add new chords.
3.  **`ChordCard` Component:** A clickable card within the grid. On first click, it opens a chord selection modal. Subsequent clicks could allow for editing.
4.  **`ChordSelector` Modal:**
    *   Presents a list of all possible chords from a predefined dictionary.
    *   Allows selection of chord duration (1, 2, 3, 4 beats, or a silence).
    *   The chord list will be dynamically colorized and ordered.
5.  **`TransportControls` Component:**
    *   Play/Pause button.
    *   Tempo control (slider or input).
    *   Loop toggle.
    *   Synthesizer selection dropdown.

#### 2.2. Core Functionality

**Roadmap:**

1.  **Chord Dictionary:**
    *   In `src/theory/chords.js`, use `Tonal.Chord.names()` to get a comprehensive list of chord types.
    *   Combine these with root notes to generate a full dictionary of playable chords.

2.  **Chord Progression State:**
    *   Use React's `useState` or a state management library (like Zustand or Jotai for simplicity) to manage the array of chords in the progression. Each chord object in the array should contain `name`, `duration`, and a unique `id`.

3.  **Dynamic Chord Colorization and Hierarchy:**
    *   **Consonance Calculation:** In `src/theory/consonance.js`, create a function `calculateConsonance(chord1, chord2)`. A simple approach is to count the number of common notes between two chords.
    *   **Color Mapping:** Create a color scale (e.g., from red for high consonance to blue for low consonance).
    *   **Dynamic List:** When the `ChordSelector` is open, if there's at least one chord in the progression, calculate the consonance of each chord in the dictionary with the last chord in the progression.
    *   Sort the chord list based on the consonance score in descending order.
    *   Apply the color mapping to the background of each chord in the list.

4.  **Audio Playback with Tone.js:**
    *   In `src/audio/player.js`, create a class or a set of functions to handle Tone.js logic.
    *   Initialize a `Tone.Synth()` instance. Allow the user to select from different synthesizer types (e.g., `Tone.FMSynth`, `Tone.AMSynth`).
    *   Use `Tone.Transport.scheduleRepeat` to schedule the playback of the chord progression.
    *   Create a `Tone.Part` to sequence the chords from the progression state.
    *   The `TransportControls` component will interact with these functions to play, pause, set the tempo (`Tone.Transport.bpm.value`), and toggle looping (`Tone.Transport.loop`).

### 3. The Explorer Mode

**Objective:** Visualize the relationship between chords and notes in an interactive, physics-based graph.

#### 3.1. UI Components

**Roadmap:**

1.  **`Explorer` Component:** The main container for this mode, which will house the `react-three-fiber` `Canvas`.
2.  **`SimulationControls` Component:** Sliders or inputs to adjust the physics parameters of the force-directed graph (e.g., charge strength, link distance).

#### 3.2. Core Functionality

**Roadmap:**

1.  **Graph Data Structure:**
    *   Create a function that takes the current chord progression and generates a graph data structure with `nodes` and `links` compatible with `r3f-forcegraph`.
    *   **Nodes:**
        *   Each unique chord in the progression is a larger node.
        *   Each unique note within those chords is a smaller node.
    *   **Links:**
        *   A link connects a note node to each chord node it belongs to.

2.  **3D Visualization with `r3f-forcegraph`:**
    *   Use the `<ForceGraph3D>` component from `r3f-forcegraph` within the `Canvas`.
    *   Pass the generated graph data to the `graphData` prop.
    *   Customize the appearance of nodes and links using the library's props (e.g., `nodeThreeObject`, `linkThreeObject`). For example, represent chords as spheres and notes as smaller spheres or text labels.

3.  **Interaction:**
    *   Implement the `onNodeClick` prop to trigger audio playback. When a chord node is clicked, play the full chord using the selected synthesizer. When a note node is clicked, play that single note.
    *   The `SimulationControls` component will update the props of the `<ForceGraph3D>` component to adjust the simulation in real-time.

### 4. Application Features

**Objective:** Implement cross-cutting features that enhance the user experience.

#### 4.1. State Management

**Roadmap:**

1.  **Global State:** Use React's Context API with `useReducer` or a lightweight state management library like **Zustand** to manage shared state between the "Composer" and "Explorer" modes.
2.  **State to Manage:**
    *   The current chord progression.
    *   Selected synthesizer.
    *   Tempo.
    *   Looping state.
    *   Simulation parameters for the explorer mode.

#### 4.2. Import/Export Functionality

**Roadmap:**

1.  **`exportSession` function:**
    *   Gathers the relevant state (chord progression, synth selection, tempo, etc.) into a single JavaScript object.
    *   Uses `JSON.stringify` to convert the object to a human-readable JSON string.
    *   Triggers a download of the JSON file.
2.  **`importSession` function:**
    *   Provides a file input for the user to select a JSON file.
    *   Reads the file content and uses `JSON.parse` to convert it back into an object.
    *   Updates the application's state with the imported data.

#### 4.3. Responsive Design

**Roadmap:**

1.  **CSS Framework or Custom CSS:** Use a CSS-in-JS solution like Styled Components or a utility-first framework like Tailwind CSS to build a responsive layout.
2.  **Layout Adaptation:**
    *   The `ChordGrid` in the "Composer" mode should adapt the number of columns based on the screen width.
    *   The controls should reflow for smaller screens, especially for portrait mode on mobile devices.
    *   The "Explorer" mode's canvas should fill the available space.

#### 4.4. Progressive Web App (PWA)

**Roadmap:**

1.  **Service Worker:**
    *   Use the service worker file provided by `create-react-app` (`src/service-worker.js`) to cache application assets for offline use.
    *   Register the service worker in `src/index.js`.
2.  **Web App Manifest:**
    *   Create a `public/manifest.json` file to define the app's name, icons, start URL, display mode, and theme color.
3.  **Installation Prompt:** The browser will automatically handle the installation prompt on supported devices once the PWA criteria are met.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/b2renger)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/b2renger)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
