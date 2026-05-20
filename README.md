# Rehably — Personal AI Rehab Support

Rehably is a browser-based rehabilitation platform designed to help seniors and patients recover at home. It uses real-time AI-powered motion tracking to guide users through therapeutic exercises, give live form feedback, and generate downloadable session summaries.

---

## Features

- **Condition-specific exercise programs** — Users select their condition (arthritis, stroke recovery, osteoporosis, dementia, hypertension, sarcopenia) and are guided to the appropriate exercise module.
- **Real-time hand tracking** — Uses MediaPipe Hands via WebAssembly to detect hand landmarks at 30+ fps directly in the browser. No server required for motion tracking.
- **Live form feedback** — Voice alerts (Web Speech API) correct posture in real time:
  - Wrist tilt and internal bend detection
  - Finger spacing monitoring
  - Wrong-finger-touch detection during thumb opposition sequences
- **Rep counting & hold timers** — Tracks reps, hold duration, and session time. Plays an audio ding on successful holds and completed reps.
- **AI session summaries** — After each session, calls the OpenAI API (`gpt-4o-mini`) to generate a personalized summary and coaching tips. Falls back to a local summary if no API key is set.
- **Downloadable reports** — Session results (reps, hold times, AI feedback) can be saved as a `.txt` file for sharing with a healthcare provider.

---

## Project Structure

```
rehably/
├── index.html                  # Landing page
├── conditions.html             # Condition selection screen
├── arthritis.html              # Arthritis exercise info page
├── sit-to-stand-exercise.html  # Sit-to-stand exercise (camera + tracking)
├── script.js                   # Shared frontend logic
├── condition-script.js         # Condition page logic
├── styles.css                  # Global styles
├── vision_bundle.mjs           # MediaPipe Tasks Vision bundle (local copy)
├── wasm/                       # MediaPipe WebAssembly assets
└── sit-to-stand-ai/
    ├── index.html              # Hand osteoarthritis exercise UI
    ├── script.js               # Hand tracking, rep counting, feedback logic
    └── config.js               # Loads OPENAI_API_KEY from .env.local
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI | HTML, CSS, vanilla JavaScript |
| Motion tracking | [MediaPipe Hands](https://developers.google.com/mediapipe/solutions/vision/hand_landmarker) (WebAssembly) |
| Voice feedback | Web Speech API (`SpeechSynthesisUtterance`) |
| Audio cues | Web Audio API |
| AI summaries | OpenAI API (`gpt-4o-mini`) |
| Icons | Font Awesome 6 |
| Dependencies | `@mediapipe/tasks-vision` |

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/tiiffanyho/rehably.git
cd rehably
```

### 2. Install dependencies

```bash
npm install
```

> The only dependency is `@mediapipe/tasks-vision`, used for the local WASM bundle.

### 3. Set up your OpenAI API key (optional)

AI-powered session summaries require an OpenAI API key. Without one, a local fallback summary is shown.

Create a `.env.local` file inside `sit-to-stand-ai/`:

```
VITE_OPENAI_API_KEY=sk-proj-YOUR-KEY-HERE
```

Alternatively, set it at runtime in the browser console:

```js
localStorage.setItem("OPENAI_API_KEY", "sk-proj-YOUR-KEY-HERE")
```

Then refresh the page.

### 4. Serve the project

Because the app uses webcam access and ES modules, it must be served over HTTP (not opened as a local file). Use any static server:

```bash
# Using Python
python3 -m http.server 8080

# Using Node
npx serve .

# Using VS Code Live Server extension
# Right-click index.html → Open with Live Server
```

Open `http://localhost:8080` in your browser.

---

## How It Works

### User flow

1. **Landing page** (`index.html`) — Overview of Rehably and how it works.
2. **Condition selection** (`conditions.html`) — User picks their condition from six options.
3. **Exercise page** — The relevant AI exercise module loads (currently fully implemented for **arthritis** hand exercises).
4. **Session** — The webcam activates, MediaPipe tracks hand landmarks at 30fps, and the app:
   - Guides the user through a thumb-opposition sequence (thumb touches index → middle → ring → pinky, repeating for the set rep goal).
   - Speaks real-time corrections if form breaks down.
   - Plays audio cues on successful reps.
5. **Session end** — The app fetches an AI-generated summary from OpenAI and offers a downloadable `.txt` report.

### Hand tracking pipeline

```
Webcam frame → MediaPipe Hands (WASM) → 21 hand landmarks
  → analyzeHand() → distance / wrist angle / finger spacing metrics
  → detectPosition() → "open" or "closed"
  → trackRep() → rep events (finger_done, rep_complete)
  → generateFeedback() + checkWristBend() + checkFingerSpacing()
  → UI update + Speech synthesis
```

---

## Supported Conditions

| Condition | Exercise Type | Status |
|---|---|---|
| Arthritis | Thumb opposition (finger touch sequence) | Implemented |
| Stroke Recovery | Facial & coordination exercises | Planned |
| Osteoporosis | Weight-bearing & balance exercises | Planned |
| Dementia | Cognitive games & gentle movements | Planned |
| Hypertension | Cardio & breathing exercises | Planned |
| Sarcopenia | Resistance & functional training | Planned |

---

## Browser Requirements

- Chrome or Edge (recommended — best WebAssembly and Speech API support)
- Webcam access must be granted
- Internet connection required on first load to fetch MediaPipe model files from CDN

---

## Privacy

All motion tracking runs entirely client-side in the browser via WebAssembly. No video or landmark data is sent to any server. The only outbound network call is to the OpenAI API at session end, which sends anonymized exercise statistics (rep count, duration, hold times) to generate the summary.

---

## Healthcare Partners

Rehably is designed in collaboration with:

- London Health Sciences Centre
- St. Joseph's Hospital
- Victoria Hospital
- Parkwood Institute

---

## License

&copy; 2025 Rehably. All rights reserved.
