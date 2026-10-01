# Hand Gesture Control System

Touchless computer interaction with an ordinary webcam: MediaPipe finds the hand, rule-based geometry recognises the gesture, a state machine makes actions deliberate, and PyAutoGUI performs a fixed whitelist of actions.

## Overview
Show a hand to the camera, hold a pose, perform a small "tap" motion, and the matching action runs **once**. Holding a pose never repeats an action; you must release (change pose) first. Everything is configurable in YAML and testable without touching your real mouse (`--test-mode`).

## Features
- Live camera preview with 21 hand landmarks, finger count, gesture, confidence, action, FPS, latency and status
- Gesture state machine: stability -> activation -> cooldown -> release lock
- Mandatory gestures: 1/2/3/4 fingers -> left click / right click / Home / screenshot
- Optional: thumb volume, open-palm scroll and back/forward, index-finger mouse control
- PySide6 GUI with Start/Stop/Pause/Emergency stop, settings screen, log viewer; plain OpenCV mode too
- Safety: ESC emergency stop (global with `pynput`), pause, cooldown, confidence threshold, action whitelist, one-hand rule, PyAutoGUI fail-safe
- Dataset collection tool, optional ML classifier (Random Forest / MLP), evaluation report
- ~120 automated tests, test mode, rotating logs

## Gesture Mapping
| Gesture | Action |
|---|---|
| 1 finger + tap | Left click |
| 2 fingers + tap | Right click |
| 3 fingers + tap | Home |
| 4 fingers (thumb folded) + tap | Screenshot |
| Open palm | Neutral / pause |
| Open palm up / down | Scroll up / down |
| Open palm swipe left / right | Back / Forward |
| Thumb up / down | Volume up / down |
| Closed fist | Pause (no action) |
| Index finger (mouse mode) | Move cursor, pinch to click |

"Tap" = quick downward flick of the fingertips and back up (or a thumb-index pinch; configurable). Details: `docs/GESTURE_SPECIFICATION.md`.

## Architecture
`Webcam -> Camera Manager -> Hand Detector -> Finger Detection -> Gesture Recognizer -> Stability/Activation -> State Machine -> Action Executor -> OS`. See `docs/SYSTEM_ARCHITECTURE.md`.

## Technology Stack
Python 3.10-3.12 | OpenCV (capture, overlay) | MediaPipe 0.10.21 (hand landmarks) | NumPy | PyAutoGUI (OS actions) | Pillow (screenshots) | PyYAML (config) | PySide6 (GUI) | scikit-learn + joblib (optional ML) | pytest | pynput (optional global ESC)

## Installation
```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```
OS-specific notes (macOS permissions, Linux X11 + `scrot`): `docs/INSTALLATION.md`.

## Usage
```bash
python src/main.py                   # GUI
python src/main.py --mode gui
python src/main.py --mode camera     # OpenCV window (keys: ESC stop, R resume, P pause, Q quit)
python src/main.py --test-mode       # recognise + simulate, control nothing (start here!)
python tools/test_camera.py          # check the camera
```

## Dataset
```bash
python tools/collect_dataset.py --split train --session alice
```
See `docs/DATASET.md` (layout, keys, variety guidelines, why validation/test need separate sessions).

## Model Training
```bash
python ml/train_model.py --model rf
python ml/evaluate_model.py --split test
```
Then set `gesture.recognizer: ml`. The project ships no model and claims no accuracy; the report is generated from your data. See `docs/MODEL_TRAINING.md`.

## Testing
```bash
pytest
```

## Configuration
`config/config.yaml` (camera, gesture thresholds, activation, movement, mouse, screenshots, safety, logging, UI) and `config/gesture_mapping.yaml` (gesture -> whitelisted action). Key defaults: `confidence_threshold 0.75`, `stability_frames 5`, `cooldown_seconds 1.0`, `max_hands 1`. Optional `.env` (see `.env.example`).

## Screenshots
Captured screenshots are saved to `screenshots/screenshot_YYYY-MM-DD_HH-MM-SS.png`. (No UI images are bundled; run the app to see the interface.)

## Troubleshooting
`docs/TROUBLESHOOTING.md`. Most common: wrong MediaPipe version (`pip install "mediapipe==0.10.21" "numpy<2"`), camera in use/blocked, and forgetting the tap motion.

## Project Structure
```text
hand-gesture-control/
|-- README.md  LICENSE  requirements.txt  pytest.ini  .gitignore  .env.example
|-- config/      config.yaml, gesture_mapping.yaml
|-- src/
|   |-- main.py
|   |-- camera/     camera_manager.py
|   |-- detection/  hand_detector.py
|   |-- gesture/    finger_detector.py gesture_recognizer.py gesture_state.py
|   |               gesture_smoother.py activation.py gesture_types.py
|   |-- actions/    action_executor.py screenshot_manager.py mouse_controller.py action_types.py
|   |-- ui/         main_window.py camera_widget.py settings_window.py overlay.py
|   |-- utils/      logger.py config_loader.py helpers.py hotkeys.py
|   `-- services/   gesture_service.py
|-- ml/          train_model.py evaluate_model.py predict.py feature_extractor.py augmentation.py
|-- tools/       collect_dataset.py test_camera.py test_gestures.py
|-- dataset/     raw/ processed/ landmarks/
|-- models/  screenshots/  logs/  reports/
|-- tests/       (10 test modules + helpers)
`-- docs/        (13 documents)
```

## Future Enhancements
Two-hand gestures, custom gesture recording, per-app profiles, presentation/media/browser control, virtual keyboard, voice + gesture, learned dynamic gestures. See `docs/FUTURE_SCOPE.md`.

## License
MIT - see `LICENSE`.
