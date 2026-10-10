  ## SignBridge // Accessible Interface & Real-Time Hybrid Gesture Recognition System

<p align="center">
  <a href="https://github.com">
  </a>
</p>

<p align="center">
   <img width="500" height="350" alt=" signbridge-dashboard" src="https://github.com/user-attachments/assets/954bc200-442f-45bc-b318-bf104a9a83b6" />
  <img width="1280" height="629" alt=" signbridge-gateway" src="https://github.com/user-attachments/assets/db3a4a70-64a6-4f61-8683-06a6cf829424" />
</p>

## Overview

**SignBridge** is a high-performance, universally accessible hybrid gesture recognition ecosystem engineered to translate real-time sign language inputs into text output and vocal synthesis instantly. Operating across low-latency duplex streaming pipelines, SignBridge accommodates **American Sign Language (ASL)**, **Indian Sign Language (ISL)**, and dynamically handles high-context **Tamil Sign Language (TSL)** structural matrices.

The system leverages localized browser-side GPU architectures to decode 3D hand spatial landmarks, offloading runtime execution loads via a concurrent, asynchronous FastAPI framework on the backend engine.

---

## System Architecture & Data Flow

SignBridge splits its execution loop across a high-frequency decoupled pipeline to achieve negligible end-to-end processing delays, keeping frames per second high and tracking latency minimal.

```text
                  ┌─────────────────────────────────────────┐
                  │         Client Web Camera Sensor        │
                  └────────────────────┬────────────────────┘
                                       │ (Live Video Frames)
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │    MediaPipe Frontend Graph (hands.js)  │
                  └────────────────────┬────────────────────┘
                                       │ (21 3D Joint Landmark Vectors)
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │     Persistent WebSockets Tunnel (/ws)   │
                  └────────────────────┬────────────────────┘
                                       │
            ┌──────────────────────────┴──────────────────────────┐
            ▼ (Pathway A: Boolean Rules Check)                     ▼ (Pathway B: Fallback Vector Matching)
┌───────────────────────────────────────┐             ┌───────────────────────────────────────┐
│     Finger-State Structural Rules     │             │       Wrist-Centered Vector Maker     │
│ (Instant relative joint state check)  │             │    (Origin translation to wrist)      │
└───────────────────┬───────────────────┘             └───────────────────┬───────────────────┘
                    │                                                     │
                    │ [Rule Match Validated]                              ▼
                    │                                 ┌───────────────────────────────────────┐
                    │                                 │     KNN Distance Template Engine      │
                    │                                 │  (Calculates Euclidean spatial dist)  │
                    └───────────────────┬─────────────┴───────────────────┘
                                        │
                                        ▼
                  ┌─────────────────────────────────────────┐
                  │       Rolling Queue Debounce Filter     │
                  │        (Temporal Frame Stabilizer)      │
                  └────────────────────┬────────────────────┘
                                       │ [Validated Token Stream]
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │     Tamil Aggregator & Compound Resolver│
                  └────────────────────┬────────────────────┘
                                       │ [Unified Linguistic Output]
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │    Dynamic UI Presentation Dashboard    │
                  └─────────────────────────────────────────┘
```

---

## Component Interactions

```text
 ┌────────────────┐               ┌─────────────────┐               ┌─────────────────┐
 │ Client Browser │               │  MediaPipe Core │               │ FastAPI Backend │
 └───────┬────────┘               └────────┬────────┘               └────────┬────────┘
         │                                 │                                 │
         │ 1. Push Raw Video Frame Buffer  │                                 │
         ├────────────────────────────────>│                                 │
         │                                 │ 2. Compute 3D Joint Arrays      │
         │                                 ├─────────────────┐               │
         │                                 │ (WebGL Shaders) │               │
         │ 3. Return 63 Landmark Array     │<────────────────┘               │
         │<────────────────────────────────┤                                 │
         │                                 │                                 │
         │ 4. Open Duplex Connection (WS /ws/{system})                       │
         ├──────────────────────────────────────────────────────────────────>│
         │                                                                   │
         │ 5. Stream High-Frequency Raw JSON Coordinate Packets              │
         ├──────────────────────────────────────────────────────────────────>│
         │                                                                   │ 6. Match Logic Engine
         │                                                                   ├─────────────────┐
         │                                                                   │ (Rules vs KNN)  │
         │                                                                   │                 │
         │ 7. Return Evaluated Sign Tokens & Confidence Scores               │<────────────────┘
         │<──────────────────────────────────────────────────────────────────┤
         │                                                                   │
```

---

## Frontend Architecture Flow

The client interface separates low-level frame telemetry extraction from high-level user interface elements to avoid main thread blocking:

* **Presentation Engine UI:** A modern web dashboard interface powered by CSS variables, providing custom configuration templates supporting four inclusive interaction profiles: **Mook (Gunga / Mute)**, **Badhir (Bahira / Deaf)**, **Drishtihin (Andha / Blind)**, and **Saamanya (Standard)**.
* **Computer Vision Kernel (`hands.js`):** Downloads and executes standard Google MediaPipe graphs inside localized WebGL textures, instantly passing normalized landmark data arrays to the network stream.
* **TSL Combination Resolver:** Evaluates dynamic structural input tokens. When a base Tamil consonant and a following vowel are parsed sequentially, the interface automatically aggregates the states to render the combined compound character correctly on screen.

---

## Backend Architecture Flow

The Python runtime infrastructure is optimized for maximum frame ingestion speed using concurrent, non-blocking components:

* **Asynchronous Routing Framework:** Manages low-latency real-time streams utilizing native Python asynchronous event loops via `FastAPI` and `Uvicorn WebSockets`.
* **Invariance Vector Transformer (`normalize()`):** Takes the raw 63 floating-point tracking variables, resets their coordinate origin directly to the wrist pivot point `(0, 0, 0)`, and scales all vector elements to unity. This guarantees layout, position, and webcam depth invariance across varying hardware devices.
* **Hybrid Recognition Router:** Implements parallel matching logic pathways:
  1. **Hardcoded State Check:** Directly calculates relative node distances to map geometric finger extension matrices, avoiding computational weight overhead.
  2. **Euclidean KNN Fallback:** Computes absolute geometric separation distances (`math.dist`) against customized template arrays saved inside `templates.json`.
* **Temporal Window Stabilizer:** Feeds evaluated tokens through sliding matrix blocks (`deque(maxlen=7)`), enforcing a strict **5-out-of-7 frame consensus threshold** to eliminate tracking jitters or accidental double triggers.

---

## Dataset Recording & Verification Protocol

To expand the template dataset arrays (`templates.json`) accurately for new gesture sets, adhere to the following setup parameters:

1. **Environmental Quality Rules:** Capture sequences under clean, multi-directional lighting setups with image tracking sensors delivering a baseline performance of **720p @ 30 FPS minimum**.
2. **Dynamic Range Sweeps:** Users must provide 30 continuous position variations per target sign token, systematically altering camera distance metrics (`Z-axis depth changes`) and hand tilting orientation.
3. **Naming Convention Standards:** Write all custom entries to the templates tree using explicit category syntax tags:
   * `TAMIL_VOWEL_[KEY]`
   * `TAMIL_CONSONANT_[KEY]`
   * `TAMIL_WORD_[KEY]`

---

## Project Directory Structure
<pre>
│
├── data/
│   └── templates.json          # Persistent JSON datastore for custom sign vectors
│
├── frontend/
│   ├── Frontend.html           # Universal multi-tenant responsive user panel
│   └── assets/
│       └── hands.js            # Compiled MediaPipe real-time tracking dependency
│
├── backend/
│   ├── main.py                 # Core FastAPI engine processing websockets & routes
│   └── tsl_rules.py            # Static structural token logic matrices for Tamil
│
└── README.md                   # Operational reference manuals and system documentation
</pre>
