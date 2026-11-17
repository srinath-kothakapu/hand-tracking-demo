# Hand Tracking Demo (first step towards an AI virtual mouse)

![CI](https://github.com/srinath-kothakapu/hand-tracking-demo/actions/workflows/ci.yml/badge.svg)

This is the first step of an "AI virtual mouse" idea. I renamed the repo from `ai-virtual-mouse` because the mouse part isn't built yet. Right now it opens the webcam, runs MediaPipe Hands on every frame and draws the hand landmarks and their connections on the video with OpenCV.

**It does not control the mouse.** It doesn't move the pointer, doesn't recognise click gestures and doesn't send any mouse or keyboard events. It just shows the hand landmarks, as a base for the actual gesture-to-pointer part later.

## What it does now

1. Opens the default webcam (`VideoCapture(0)`).
2. Reads frames until the stream ends or you quit.
3. Converts each frame from OpenCV's BGR to RGB for MediaPipe.
4. Finds hand landmarks with MediaPipe Hands when a hand is visible.
5. Draws the landmarks and connections on the original frame.
6. Shows the result in a window called **Hand Tracking**.
7. Releases the camera and closes the windows on exit, also when there's an error.

There's also `camera_test.py`, a plain webcam preview to check that OpenCV can open the camera at all, and tests in `tests/` that run the real MediaPipe model on generated frames, so they don't need a camera.

```text
webcam frame (BGR) -> convert to RGB -> MediaPipe Hands -> draw landmarks -> preview window
```

The scripts only show the frames on screen. They don't save frames or send them anywhere. The program still needs camera permission from the operating system.

## Requirements

- Python 3.9 to 3.12 (MediaPipe 0.10.21 has no wheels for newer versions)
- a webcam OpenCV can open
- a desktop session that can open an OpenCV window

Pinned dependencies:

- `mediapipe==0.10.21` for the hand landmarks
- `opencv-python==4.11.0.86` for camera, colour conversion, drawing and the window
- `pytest` for the tests

I pinned the versions on purpose. The code uses MediaPipe's `mp.solutions` API, which was removed later (it's gone from 0.10.30 on), and newer OpenCV versions need a NumPy version that MediaPipe 0.10.21 doesn't support. Moving to a newer MediaPipe would mean porting the code to the MediaPipe Tasks `HandLandmarker` API.

## Install

```bash
git clone https://github.com/srinath-kothakapu/hand-tracking-demo.git
cd hand-tracking-demo
python -m venv .venv

# Windows PowerShell
.\.venv\Scripts\Activate.ps1
# macOS / Linux
source .venv/bin/activate

python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

If the install fails, check that your Python version, OS and CPU have a matching MediaPipe wheel. Without a desktop the libraries may install, but the window can't open.

## Run it

```bash
python hand_tracking.py
```

Hold one or two hands in front of the camera and the landmarks and skeleton lines get drawn on the live image. Press **q** in the window to quit.

It uses camera index `0`. If your camera has another index, change `cv2.VideoCapture(0)` in `hand_tracking.py`. There's no command-line option for this yet.

## Check the camera on its own

```bash
python camera_test.py
```

This opens a **Camera Test** window with the raw image. Press **q** to close it. If the camera can't be opened or a frame can't be read, it raises a clear error.

## Tests

```bash
python -m pytest
```

The tests send blank and random-noise frames through the real MediaPipe Hands model. They check that the pipeline runs, that nothing gets drawn when there's no hand, and that a missing camera gives a clear error. GitHub Actions runs them on every push. They don't measure how well hands are detected; there's no accuracy evaluation in this repo yet.

## Files

```text
hand_tracking.py    webcam, MediaPipe Hands, drawing the landmarks
camera_test.py      plain webcam preview without hand detection
tests/              tests that run without a camera
requirements.txt    pinned MediaPipe, OpenCV and pytest
.github/workflows/  CI on every push
```

## Troubleshooting

- **Can't open the camera:** give your terminal or editor camera permission, close other apps using the webcam, and try another camera index.
- **Window opens but frames fail:** check the camera connection and the privacy settings. The script raises a `RuntimeError` when a frame can't be read.
- **No landmarks:** use better light and keep the whole hand in the frame. The scripts don't expose confidence or threshold settings yet.
- **No window on a headless machine:** you need a desktop session; there's no headless mode.
- **`module 'mediapipe' has no attribute 'solutions'`:** your MediaPipe is newer than 0.10.21. Reinstall from `requirements.txt` in a fresh environment.
- **Install fails:** check Python version (3.9 to 3.12), OS, CPU and wheel availability for both packages.

## What's missing for a real virtual mouse

Right now it only shows landmarks. For an actual virtual mouse I would still need:

- mapping a filtered landmark to screen coordinates, with a calibrated area
- smoothing against jitter, and handling a lost hand, camera movement and changing distance
- defined gestures, click debouncing and protection against accidental clicks
- an opt-in backend for real pointer and click events
- unit tests for the coordinate mapping and gesture logic, and a manual usability test (the current tests only check that the pipeline runs)
- clear permission and privacy controls before anything controls the computer

## Privacy

The scripts only show the webcam image. They don't record or upload anything. Camera input is still sensitive: only use a camera and a place you're allowed to, don't show other people in screenshots or recordings, and close the program when you're done. Nothing in here controls the cursor or triggers system actions.
