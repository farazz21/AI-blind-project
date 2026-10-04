#AI Vision Risk Assistant for Android

«An on-device AI vision assistant for visually impaired people that detects nearby objects, estimates risk, and provides real-time voice alerts.»

AI Vision Risk Assistant is an Android-based assistive vision prototype designed to help visually impaired users understand potentially dangerous objects around them.

The application uses the smartphone camera to detect objects, track their movement, estimate approximate distance, calculate collision risk, and provide real-time voice alerts.

The main goal is to provide useful safety information without requiring a cloud server or continuous Internet connection.

---

🎯 Project Goal

The application answers four important questions:

1. What is around the user?
2. Where is it?
3. How close or dangerous is it?
4. Should the user receive an alert?

Example:

«🎧 "Person on your left, 2 meters."»

If the risk becomes high:

«🔊 "Warning! Bicycle approaching from right."»

For critical situations:

«🔊 "Danger! Object very close. Stop."»

---

🧠 System Architecture

                 📱 Android Phone
                       │
                       ▼
                CameraX Camera
                       │
                       ▼
              Frame Pre-processing
                       │
                       ▼
              On-device AI Model
                 (Object Detection)
                       │
                       ▼
                 Object Tracking
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
       Distance/Depth      Motion Analysis
        Estimation              │
              │                 │
              └────────┬────────┘
                       ▼
                 TTC Calculation
                       │
                       ▼
               8-Factor Risk Engine
                       │
                       ▼
                 Risk Score 0–100
                       │
                       ▼
                Alert Decision
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
       Android TTS           Alert Sound
             │                   │
             └─────────┬─────────┘
                       ▼
                  🎧 User

---

✨ Main Features

1. Real-Time Object Detection

The phone camera continuously captures the user's surroundings.

The AI model detects objects such as:

- Person
- Car
- Bicycle
- Motorcycle
- Bus
- Truck
- Dog
- Cat
- Chair
- Bench
- Other supported objects

Each detection contains:

Object Class
Confidence
Bounding Box
Object ID

---

🔄 2. Object Tracking

The application tracks detected objects across consecutive frames.

Example:

Person ID: 04

Frame 1 → 3.0 m
Frame 2 → 2.6 m
Frame 3 → 2.2 m
Frame 4 → 1.8 m

This helps determine whether an object is approaching the user.

---

📏 3. Distance Estimation

The application attempts to estimate the approximate distance of detected objects.

Possible approaches:

Option A — Monocular Depth

Use an on-device monocular depth model.

Camera Image
     ↓
Depth Model
     ↓
Relative Depth
     ↓
Calibration
     ↓
Approximate Distance

Option B — Device Depth Sensor

If the Android device provides a compatible depth sensor, its depth information can be used where available.

The application should not assume every Android phone has a depth sensor.

---

🧭 4. Direction Detection

The camera frame is divided into three primary regions:

┌──────────┬──────────┬──────────┐
│          │          │          │
│   LEFT   │  CENTER  │  RIGHT   │
│          │          │          │
└──────────┴──────────┴──────────┘

The application determines whether an object is:

LEFT
CENTER
RIGHT

Example:

«"Person on your left."»

---

⚠️ 5. Time To Collision

When enough motion information is available, the application estimates closing speed.

Formula:

TTC = Distance / Closing Speed

Example:

Distance = 2.0 m
Closing Speed = 1.0 m/s

TTC = 2 seconds

Lower TTC indicates greater potential risk.

---

🧮 6. 8-Factor Risk Engine

The application calculates a risk score between 0 and 100.

The following factors are considered:

1. Object Type

Different objects receive different risk weights.

Car / Truck / Bus → High
Motorcycle        → High
Bicycle            → Medium-High
Person             → Medium
Dog                → Medium
Chair / Bench      → Low

2. Distance

Closer objects increase risk.

3. Direction

Objects near the center of the user's path receive higher risk.

4. Object Position

Large objects occupying more of the frame may represent a greater obstacle.

5. Motion / TTC

Objects approaching the user increase risk.

6. User Motion

The system can attempt to determine whether the user is:

FORWARD
BACKWARD
STATIONARY

using available camera/depth/motion information.

7. Path Blocking

The system checks whether an object appears to block the user's forward path.

8. Detection Confidence

Low-confidence detections have reduced influence on the final risk.

---

🚨 7. Four-Level Alert System

┌──────────────┬───────────────┐
│ Risk         │ Alert         │
├──────────────┼───────────────┤
│ 0 – 25       │ 🟢 No Alert   │
│ 25 – 50      │ 🟡 Warning    │
│ 50 – 75      │ 🟠 Prompt     │
│ 75 – 100     │ 🔴 Immediate  │
└──────────────┴───────────────┘

---

🎧 8. Voice Assistance

Android's built-in Text-to-Speech system is used for voice alerts.

Warning

"Caution, bicycle nearby."

User Prompt

"Warning! Bicycle from left, 1.5 meters."

Immediate

"Danger! Bicycle very close. Stop."

The app should keep voice messages short because the user needs to understand them quickly.

---

🔁 9. Debounce and Cooldown

The app should avoid repeatedly speaking the same warning.

Debounce

An event must remain active for several frames before triggering an alert.

Example:

WARNING       → 5 frames
USER_PROMPT   → 3 frames
IMMEDIATE     → 1 frame

Cooldown

The same alert cannot repeat immediately.

Example:

WARNING       → 4 seconds
USER_PROMPT   → 2 seconds
IMMEDIATE     → 0.5 seconds

Higher-risk alerts override lower-risk alerts.

---

📱 Android Technology Stack

Recommended stack:

Android Studio
Kotlin
CameraX
Android Text-to-Speech
Jetpack
On-device AI inference

For the AI model, use a model/runtime that is actually supported by the target Android deployment rather than assuming a desktop Python YOLO package can run unchanged on Android.

---

📁 Recommended Project Structure

AI-Vision-Risk-Assistant/
│
├── app/
│   └── src/
│       └── main/
│           ├── java/
│           │   └── com/example/aivision/
│           │       ├── MainActivity.kt
│           │       ├── camera/
│           │       │   └── CameraManager.kt
│           │       ├── detection/
│           │       │   ├── ObjectDetector.kt
│           │       │   └── DetectionResult.kt
│           │       ├── tracking/
│           │       │   └── ObjectTracker.kt
│           │       ├── depth/
│           │       │   └── DepthEstimator.kt
│           │       ├── risk/
│           │       │   ├── RiskEngine.kt
│           │       │   └── TTCCalculator.kt
│           │       ├── alert/
│           │       │   ├── AlertManager.kt
│           │       │   └── VoiceManager.kt
│           │       └── settings/
│           │           └── AppSettings.kt
│           │
│           ├── res/
│           │   ├── layout/
│           │   ├── drawable/
│           │   ├── mipmap/
│           │   └── values/
│           │
│           └── AndroidManifest.xml
│
├── models/
│   └── detection_model
│
├── README.md
└── LICENSE

---

🔐 Privacy

The core vision pipeline is intended to process camera frames on the device.

The application should not upload camera frames to a remote server unless a future feature explicitly requires it and the user is informed.

No account should be required for the basic offline functionality.

---

🌐 Offline Operation

The target architecture is:

📱 Camera
   ↓
On-device AI
   ↓
Object Detection
   ↓
Tracking
   ↓
Distance / Depth
   ↓
Risk Engine
   ↓
Android TTS
   ↓
🎧 User

Internet should not be required for the core detection and alert pipeline.

However, model files and application dependencies need to be installed/downloaded beforehand.

---

⚡ Performance Strategy

Real-time computer vision can be demanding on mobile hardware.

The application should use:

- Reduced camera resolution when necessary
- Efficient on-device models
- Frame skipping
- Detection interval control
- Depth estimation at a lower frequency
- Tracking between detection frames
- Lightweight image preprocessing
- Background processing
- Avoiding unnecessary UI rendering

Example:

Detection → every frame or selected interval
Depth     → every 3–5 frames
Tracking  → continuously
Risk      → continuously using latest valid data
Voice     → only when alert state changes

---

🧪 Distance Calibration

Monocular depth estimation does not automatically guarantee accurate real-world meters.

The app should provide a calibration process.

Example:

Place an object 1 meter away.

Actual distance:
1.0 meter

Model output:
relative depth value

Calibration:
adjust model-to-distance mapping

Calibration should be tested using the actual phone camera.

Results can change depending on:

- Camera
- Lens
- Lighting
- Field of view
- Camera position
- Object size
- Environment

---

📊 Example Runtime Output

For development/debugging:

AI Vision Risk Assistant

FPS: 18

Object: Person
ID: 04

Distance: 1.42 m
Direction: CENTER

Closing Speed: 0.82 m/s
TTC: 1.73 s

Risk Score: 67
Alert: USER_PROMPT

Voice:

«"Warning! Person ahead, 1.4 meters."»

---

🛠️ Development Phases

Phase 1 — Camera

Implement:

CameraX
↓
Live camera frames

---

Phase 2 — Object Detection

Add:

Camera
↓
On-device object detector
↓
Bounding boxes
↓
Object labels

---

Phase 3 — Voice

Add Android Text-to-Speech.

Example:

Detected:
Person

Voice:
"Person ahead."

---

Phase 4 — Direction

Add:

LEFT
CENTER
RIGHT

Voice:

«"Person on your right."»

---

Phase 5 — Tracking

Add object IDs and motion tracking.

Person ID 01
↓
Distance changes
↓
Closing / moving away

---

Phase 6 — Distance

Add depth/distance estimation.

Example:

Person → 1.8 m
Car → 3.4 m

---

Phase 7 — TTC

Add closing-speed estimation and TTC.

---

Phase 8 — Risk Engine

Implement the 8 risk factors.

Output:

Risk = 0–100

---

Phase 9 — Alert Engine

Implement:

NO_EVENT
WARNING
USER_PROMPT
IMMEDIATE

with debounce and cooldown.

---

Phase 10 — Optimization

Optimize for:

Battery
FPS
CPU/GPU/NPU usage
Memory
Latency

---

⚠️ Limitations

This is a research/prototype assistive system.

It should not be treated as a certified replacement for a white cane, guide dog, or other established mobility aid.

Potential sources of error include:

- Incorrect object detection
- Poor lighting
- Camera obstruction
- Depth estimation errors
- Fast-moving objects
- Occlusion
- Camera motion
- Incorrect distance estimation
- Phone hardware limitations

Safety-critical decisions should therefore not rely exclusively on the app.

---

🎓 Research Contribution

The main research focus is not simply:

«"Detect an object."»

Instead, the project combines:

Object Detection
        +
Object Tracking
        +
Distance / Depth Estimation
        +
Direction Estimation
        +
Motion Analysis
        +
TTC
        +
8-Factor Risk Calculation
        +
Adaptive Alert Engine
        +
On-Device Voice Assistance

The goal is to develop a risk-aware AI vision assistant that provides concise audio warnings to visually impaired users.

---

🚀 Future Features

Possible future improvements:

- Bengali voice alerts
- English/Bengali language selection
- Spatial audio
- Vibration alerts
- Headphone support
- Personalized risk thresholds
- Better depth estimation
- Device sensor fusion
- Indoor navigation
- Outdoor navigation
- GPS integration
- Crosswalk detection
- Traffic-light detection
- Road/sidewalk detection
- Person trajectory prediction
- Emergency mode
- Battery-saving mode

---

📌 Project Summary

Project:
AI Vision Risk Assistant

Platform:
Android

Input:
Smartphone Camera

Processing:
On-device AI

Output:
Voice + Sound + Optional Vibration

Target Users:
Visually Impaired People

Main Functions:
Object Detection
Object Tracking
Direction Detection
Distance Estimation
TTC
Risk Calculation
Voice Alert

Internet:
Not required for core operation

---

⚠️ Important Development Note

The first working Android prototype should not attempt every feature at once.

Recommended order:

Camera
  ↓
Object Detection
  ↓
Voice
  ↓
Direction
  ↓
Tracking
  ↓
Distance
  ↓
TTC
  ↓
Risk Engine
  ↓
Final Alert System

This makes debugging and research evaluation much easier.
