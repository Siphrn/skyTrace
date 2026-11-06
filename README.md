<div align="center">

# SkyTrace

**Turn every paper airplane throw into a flight experiment.**

Track a paper airplane with a webcam, visualize its trajectory, score its smoothness, compare designs, and hear your results, all from one laptop.

[![HackPHS 2026](https://img.shields.io/badge/HackPHS%202026-Winner-gold)](https://hackphs2026.devpost.com/)
[![MLH](https://img.shields.io/badge/MLH-Best%20Use%20of%20ElevenLabs-blue)](https://devpost.com/software/skytrace-6dfgk3)
![Python](https://img.shields.io/badge/Python-3-3776AB?logo=python&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-computer%20vision-5C3EE8?logo=opencv&logoColor=white)
![Arduino](https://img.shields.io/badge/Arduino-optional%20scoreboard-00979D?logo=arduino&logoColor=white)

[Demo Video](https://www.youtube.com/watch?v=YG3gNSOa1BQ) · [Devpost](https://devpost.com/software/skytrace-6dfgk3) · [Quick Start](#quick-start) · [Troubleshooting](#testing--troubleshooting)

</div>

---

## Awards

Built at **HackPHS 2026** for the **"Into the Skies"** theme.

- 🏆 **[MLH] Best Use of ElevenLabs**
- 🏆 **General Best Use of ElevenLabs**

## Overview

Paper airplanes are easy to build but hard to compare by eye. After a throw, it is difficult to say why one flight felt smoother than another, or whether a design tweak actually helped.

SkyTrace makes each throw reviewable. It follows a yellow-marked airplane through a webcam and produces a camera-visible trajectory, a 0-100 smoothness score, a short-term motion forecast, and an annotated video clip. Build, fly, compare, improve.

## Screenshots

| Dashboard | Trajectory graph |
|---|---|
| ![Dashboard UI](https://d112y698adiu2z.cloudfront.net/photos/production/software_photos/005/555/643/datas/original.png) | ![Trajectory graph](https://d112y698adiu2z.cloudfront.net/photos/production/software_photos/005/557/255/datas/original.png) |

| OpenCV tracking | Flight log |
|---|---|
| ![OpenCV frame tracking the yellow plane](https://d112y698adiu2z.cloudfront.net/photos/production/software_photos/005/557/251/datas/original.png) | ![Flight log](https://d112y698adiu2z.cloudfront.net/photos/production/software_photos/005/557/266/datas/original.png) |

## Features

| | Feature | Description |
|---|---|---|
| 🎯 | **Live tracking** | OpenCV follows yellow markings on the airplane through the webcam, with smoothing to reduce detection jitter. |
| 📈 | **Smoothness scoring** | A 0-100 score based on direction and speed consistency during the visible flight. |
| 🛰️ | **Trajectory dashboard** | Observed path, visible airtime, pixel-based speed, and a short-term forecast. |
| ✈️ | **Plane profiles** | Organize designs, reorder profiles, and drag saved flights into categories. |
| 🔀 | **Flight comparisons** | Overlay up to five previous trajectories for side-by-side analysis. |
| 🎬 | **Flight videos** | Short local clips with tracking annotations and the final score. |
| 🔢 | **Arduino scoreboard** | Send completed scores over USB serial to a 1602 LCD. |
| 🗣️ | **Voice coach** | ElevenLabs text-to-speech delivers spoken, score-based feedback. |

## Architecture

```
 Webcam ──► OpenCV tracker (Python) ──► Local HTTP server ──► Browser dashboard
                  │                            │                (HTML/CSS/JS/SVG)
                  │                            └──► ElevenLabs TTS (optional)
                  ├──► USB serial ──► Arduino + LCD1602 (optional)
                  └──► H.264 clip encoder ──► .skytrace/ (local storage)
```

Everything runs on a single machine. No second laptop and no separate frontend server are required.

## Requirements

- A laptop with a webcam and **Python 3** (developed on macOS)
- A paper airplane with clearly visible **yellow markings**
- *Optional, scoreboard:* Arduino, USB cable, LCD1602, breadboard, and wiring that matches the sketch
- *Optional, voice coach:* an ElevenLabs API key, internet access, and available API credits

Tracking and the dashboard work without an Arduino or an internet connection. Only the voice coach needs internet access.

## Quick Start

### 1. Install dependencies

From the repository root:

```bash
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
python -m pip install -r requirements.txt
```

### 2. Set up the scoreboard (optional)

1. Open [`arduino/arduinoWriting.ino`](arduino/arduinoWriting.ino) in the Arduino IDE and upload it to your board.
2. Wire the LCD to match the sketch's parallel connection: `LiquidCrystal lcd(12, 11, 5, 4, 3, 2)`.
3. Connect the Arduino to the same laptop by USB and **close the Arduino IDE Serial Monitor** before starting Python. Communication runs at **9600 baud**.

The tracker checks common macOS Arduino ports automatically. If needed, set `SKYTRACE_ARDUINO_PORT` to your board's serial port. Without a board, tracking and the dashboard continue without the LCD.

### 3. Start SkyTrace

```bash
python code/main.py
```

Allow camera access if prompted, then open **[localhost:8000](http://localhost:8000)** in a browser on the same laptop. If the port is occupied, use the dashboard URL printed in the terminal. Keep the OpenCV window and browser side by side, and leave Python running while you use the dashboard.

### 4. Make your first throw

1. Keep the camera stationary and choose a well-lit background with no other yellow objects.
2. Optionally add a design under **Planes**, then select it under **Next throw**.
3. Focus the **OpenCV camera window** and press **Space** to arm the tracker.
4. Throw the airplane through the camera view.
5. Review the score, trajectory, and video in the browser.

When the airplane is no longer tracked, the flight ends and its graph freezes. Press **Space** to begin the next throw.

**Keyboard controls (OpenCV window)**

| Key | Action |
|---|---|
| `Space` | Arm a new throw |
| `C` | Clear the current path and cancel an unfinished throw |
| `Q` | Quit the tracker |

## Using the Dashboard

### Flights

The log keeps the **50 most recent completed flights**, including very short throws.

- Filter by plane with **All planes**.
- Click a flight name to rename it (Enter or click away to save, Escape to cancel).
- Open **⋯** for *Rename* and *Move to plane…*, or drag a flight's grip onto a plane or **Unassigned**.
- Choose **Compare** to overlay up to five flights as translucent paths. **Done** hides the controls without removing overlays, and **Clear comparisons** removes overlays without deleting flights.
- **Video** opens the recorded camera view once encoding finishes. Each clip includes up to 1 second before detection and 1 second after the flight.
- Trash buttons permanently delete a flight (and its video) or a plane profile. **Deleting a profile also deletes all flights and videos assigned to it.**

Replaying a video never changes the current score or triggers spoken commentary.

### Planes

Use **+ Add plane** to create a profile. Each row shows its saved flight count and best score, and can be reordered with the drag grip. Up to **30 profiles** can be saved locally.

The shared **Next throw** selector decides which plane is recorded when you arm a throw. Changing it mid-flight affects the *next* throw, and moving a saved flight to another category does not change the selection.

## ElevenLabs Flight Coach (optional)

The coach speaks short, scripted feedback based on each completed flight's score.

**Set up credentials**

On macOS, run the one-time secure setup:

```bash
python code/configure_voice.py
```

The key is stored in **macOS Keychain**, never in source files or the browser. Alternatively, provide `ELEVENLABS_API_KEY` as an environment variable. Enable **Text to Speech** access on the key.

> Never commit API keys, and rotate any key that has been exposed.

**Use it**

Start the tracker normally, then enable **Flight coach** in the dashboard's top bar. The click also enables browser audio. While enabled, the coach speaks once per completed flight. Disabling it stops playback and prevents new requests from that browser, though requests already in progress may still consume credits.

| Setting | Default | Override |
|---|---|---|
| Voice | George (`JBFqnCBsd6RMkjVDRZzb`) | `ELEVENLABS_VOICE_ID` |

**How AI is used:** ElevenLabs converts a short, scripted message into speech. It does not compute the score or predict the flight. Only that text is sent to ElevenLabs, never camera footage. The feedback is general coaching, not a diagnosis of the airplane's design.

## Configuration

| Variable | Purpose |
|---|---|
| `SKYTRACE_ARDUINO_PORT` | Serial port of the Arduino, if auto-detection fails |
| `ELEVENLABS_API_KEY` | ElevenLabs API key (alternative to Keychain) |
| `ELEVENLABS_VOICE_ID` | Voice used by the flight coach |

## Understanding the Measurements

- The **solid yellow line** is observed motion; the **dashed blue line** is a forecast. Comparison paths appear as translucent colored lines.
- Speed and distance are in **camera pixels**, not meters. Angle is derived from recent tracked positions.
- Prediction uses a simplified 2D motion model, not a full aerodynamic simulation.
- The graph distinguishes launch, last observed position, estimated frame exit, and estimated landing at the model's ground reference. **Leaving the camera view is not a measured landing.**

Real-world measurements would require camera calibration, and meaningful comparisons require a consistent camera position and view.

## Local Recordings & Privacy

SkyTrace keeps a rolling one-second camera buffer while it waits for a throw. A completed clip contains up to one second before detection, the tracked throw, and one second after the final detection. Clearing a throw discards an unfinished recording.

| | |
|---|---|
| **Format** | H.264 MP4, no audio, up to 17 s including pre/post-roll, max 640 px wide, 20 fps |
| **Storage cap** | 100 MiB of completed clips, 10 MiB per clip. Oldest clips expire first; scores and paths remain |
| **Capture buffer** | An additional bounded 8 MiB JPEG buffer and one background encoding job. Limited clips are marked as shortened; if encoding is unavailable or busy, tracking continues without a clip |
| **Persistence** | Profiles, the 50-flight log, and completed clips are restored after restarts from the gitignored `.skytrace/` folder |
| **Encoding** | OpenCV uses a temporary local MP4 that is removed immediately afterward. **Footage is never uploaded to cloud services** |

Clips also expire when their flight leaves the 50-flight log, and older throws cannot gain recordings retroactively. Deleting a flight deletes its stored MP4.

> **Security note:** although only one laptop is needed, the development server is reachable on the local network. Anyone who can access it can view that session's dashboard and clips. **Do not expose it to the public internet.**

## Tech Stack

| Component | Technology |
|---|---|
| Tracking, scoring & forecasting | Python, OpenCV, NumPy |
| Dashboard server | Python standard library |
| Dashboard & trajectory graphics | HTML, CSS, JavaScript, SVG |
| Physical scoreboard | Arduino, LiquidCrystal, USB serial via pyserial |
| Spoken feedback | ElevenLabs Text to Speech API |
| Video recording | OpenCV, H.264 MP4 |

## Project Structure

```
skyTrace/
├── arduino/            # Arduino sketch for the LCD scoreboard
├── code/               # Python tracker, scoring, server, voice setup, and tests
├── web/                # Browser dashboard (HTML, CSS, JavaScript, SVG)
├── requirements.txt    # Python dependencies
└── README.md
```

## Testing & Troubleshooting

Run checks from the repository root with the virtual environment active:

```bash
python -m unittest discover -s code -p 'test_*.py'
node code/test_coach.cjs        # Node.js is only needed for these JS checks
```

| Problem | What to try |
|---|---|
| Camera won't open | Close other apps using it and check camera permissions |
| Plane isn't detected | Check yellow markings, lighting, background, and the detection-mask window |
| Dashboard isn't updating | Confirm Python is running and use the URL it printed |
| LCD isn't updating | Check wiring, serial port, matching sketch, and that Serial Monitor is closed |
| Voice isn't playing | Enable Flight coach; check credentials, internet access, and API credits |
| Video unavailable | The clip may still be encoding, may have expired, or the encoder may be unsupported or busy |

After installing code updates, restart the tracker and refresh the page.

## Roadmap

- Tracking-confidence indicators
- Camera calibration for real-world distances and speeds
- Airplane detection that doesn't depend on colored markings
- An improved prediction model
- Richer design-comparison tools

## Team

Built at HackPHS 2026 by:

- [Lucas Kordzakhia](https://github.com/Siphrn): website (backend and frontend) and trajectory logic
- [paishrisha](https://github.com/paishrisha)
- [Lazi Barbakadze](https://devpost.com/lazibarbakadze1)
- [Ishaan Mohapatra](https://devpost.com/ishaan-mohapatra2024)

---

<div align="center">

**SkyTrace · HackPHS 2026 · Into the Skies**

</div>
