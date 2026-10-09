# SignBridge // Accessible Interface & Real-time Gesture Recognition System

**SignBridge** ek high-performance, accessible real-time hybrid gesture recognition system hai. Yeh system human hands ko webcam feed ke through real-time me track karta hai aur perform kiye gaye gestures ko instantly **ASL (American Sign Language)** aur **ISL (Indian Sign Language)** me translate karta hai.

Isme video input processing ke liye client-side par frontend infrastructure aur asynchronous computation ke liye ek modular FastAPI backend engine functional hai.

---

## Application Preview & Interface

```text
 ┌────────────────────────────────────────────────────────────────────────┐
 │  SignBridge // Accessible Interface                        [●] [▬] [X] │
 ├────────────────────────────────────────────────────────────────────────┤
 │                                                                        │
 │   ┌──────────────────────────┐      ┌──────────────────────────────┐   │
 │   │                          │      │  ⚡ Live Sign Engine          │   │
 │   │                          │      ├──────────────────────────────┤   │
 │   │    [ Real-time Video ]   │      │  Hand 1: [ B ] (Conf: 90%)   │   │
 │   │    (MediaPipe Tracking)  │      │  Hand 2: [ -- ]              │   │
 │   │                          │      ├──────────────────────────────┤   │
 │   │                          │      │  System Health               │   │
 │   └──────────────────────────┘      │  Avg Latency: 12.4ms         │   │
 │   [Frontend: Next.js/Tailwind]      │  Processed Frames: 4892      │   │
 │                                     └──────────────────────────────┘   │
 └────────────────────────────────────────────────────────────────────────┘
```

---

## App Architecture & System Flow

SignBridge ka complete system logic aur data-flow down levels par distributed hai, jo system latency ko baseline minimize karta hai:

```text
                  ┌─────────────────────────────────────────┐
                  │          Client Camera / WebCam         │
                  └────────────────────┬────────────────────┘
                                       │ (Video Frames)
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │     Frontend Engine (hands.js / WebGL)  │
                  └────────────────────┬────────────────────┘
                                       │ (21 Landmark Coordinates Matrix)
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │      Live WebSocket Stream (/ws)        │
                  └────────────────────┬────────────────────┘
                                       │
            ┌──────────────────────────┴──────────────────────────┐
            ▼ (Pathway 1: Rules Match)                            ▼ (Pathway 2: Fallback Machine Learning)
┌───────────────────────────────────────┐             ┌───────────────────────────────────────┐
│     Finger-State Structural Rules     │             │       Wrist-Centered Vector Maker     │
│ (Instant structural angle mapping)    │             │    (Origin normalization at wrist)    │
└───────────────────┬───────────────────┘             └───────────────────┬───────────────────┘
                    │                                                     │
                    │ [Match Found]                                       ▼
                    │                                 ┌───────────────────────────────────────┐
                    │                                 │     KNN Distance Template Engine      │
                    │                                 │    (Calculates Euclidean math.dist)   │
                    └───────────────────┬─────────────┴───────────────────┘-------------------|
                                        │
                                        ▼
                  ┌─────────────────────────────────────────┐
                  │       Rolling Queue Debounce Filter     │
                  │        (Temporal Frame Checker)         │
                  └────────────────────┬────────────────────┘
                                       │ [Validated Sign Input]
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │        Dashboard Metrics UI & Core      │
                  └─────────────────────────────────────────┘
##  Features Implemented

* **Multi-Language Sign Support:** Dynamic support for both **ASL** and **ISL** language engines directly inside a single runtime environment.
* **Hybrid Classification Pipeline:** Dual-layered recognition protocol:
  1. **Rule-Based Check:** Directly processes structural states (open/closed fingers) for immediate mapping without training weight overhead.
  2. **Template Match:** Wrist-centered vector normalization scaling engine with Euclidean distance sorting algorithm (`math.dist`) for highly precise user-trained gestures.
* **Double-Hand Parallel Processing:** Independent concurrent analysis for both hands over a single JSON streaming payload.
* **Anti-Flicker Temporal Debounce:** Integrated sliding queue implementation (`deque(maxlen=7)`) to restrict accidental letter triggering until the frame counts reach stable structural thresholds (minimum 5 matches).
* **Integrated Emergency Lifeline Framework:** Priority lookup dictionary indexing essential citizen lifelines (112 Emergency, 108 Ambulance, 1091 Women Helpline).

---

## Tech Stack & Dependencies

### Frontend
* **Core Framework:** HTML5, Modern CSS Variables architecture (Responsive custom glass-morphic theme).
* **Tracking Library:** Google MediaPipe Hands (`hands.js`) running specialized 3D landmark structural tracking graphs.

### Backend Engine
* **Language/Framework:** Python 3.10+, FastAPI (Asynchronous high-concurrency request routing).
* **Real-time Pipeline:** Uvicorn Async WebSockets protocol management.
* **Data Layer:** File-based standard JSON persistence templates (`templates.json`).

---

## API Reference Map

### Live WebSocket Channel
* **Endpoint:** `WS /ws/{system}`
* **Parameters:** `system` (Must be either `ASL` or `ISL`)
* **Payload Format:**
```json
{
  "hands": [
    [x0, y0, z0, ... x20, y20, z20],
    [optional_hand_2_landmarks]
  ]
}
```

### HTTP Endpoints
* **`POST /api/train`**: Custom gestures ko append karne ke liye endpoint interface. Accepts a 63-floating point array mapped against custom token labels.
* **`GET /api/stats`**: Total processed frames, average latency (ms), trained samples count, aur recent recognition history track karne ke liye monitor engine interface.

---

## Quick Start Guide

### Backend Initialize Kaise Karein:
1. Sabse pehle required packages install karein:
```bash
pip install fastapi uvicorn pydantic
```
2. Engine trigger karne ke liye server start karein:
```bash
uvicorn main:app --reload --port 8000
```

### Frontend Initialize Kaise Karein:
1. Apne target folder me static HTML layout aur custom CSS files structure check karein.
2. `index.html` file ko local workspace browser me open karke live tracking ke liye webcam permission allow karein.
