#AI Vision Risk Assistant

«An offline AI-powered visual assistance and collision-risk detection system for visually impaired people.»

AI Vision Risk Assistant is a real-time computer vision system designed to help visually impaired users understand nearby objects and potential collision risks.

The system uses a webcam, object detection, object tracking, monocular depth estimation, distance estimation, motion analysis, TTC (Time to Collision), and a multi-level risk engine to generate voice-based safety alerts.

The entire system is designed to run locally on Ubuntu Linux without requiring an Internet connection or cloud API.

---

🎯 Project Goal

The main goal is to provide a local AI assistant that can answer:

- What object is in front of the user?
- Where is the object?
- Approximately how far away is it?
- Is the object getting closer?
- How quickly is it approaching?
- Is the user's path blocked?
- How dangerous is the situation?
- When should the user receive an audio warning?

Example:

«🎧 "Warning! Person from left, 1.4 meters."»

Or in a critical situation:

«🔊 "DANGER! Bicycle very close! Stop now!"»

---

🧠 System Architecture

                    Webcam
                       │
                       ▼
                OpenCV Capture
                       │
                       ▼
              Image Pre-processing
             CLAHE + Gamma Correction
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
          YOLOv8n          Depth Anything V2
             │                   │
             ▼                   ▼
        Object Detection    Depth Map
             │                   │
             ▼                   │
       ByteTrack Tracking        │
             │                   │
             └─────────┬─────────┘
                       ▼
              Distance Estimation
                       │
                       ▼
                Motion Analysis
                       │
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
              Alert Decision Engine
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
           Warning   Voice    Sound
                       │
                       ▼
                 User Feedback

---

✨ Main Features

1. Real-Time Object Detection

Uses YOLOv8n to detect common objects such as:

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
- Other supported COCO objects

Each detection contains:

Object Class
Confidence
Bounding Box
Object ID

---

2. Object Tracking

The system uses ByteTrack to maintain object identities across video frames.

Example:

Person ID: 04

Frame 1 → 2.8 m
Frame 2 → 2.5 m
Frame 3 → 2.2 m
Frame 4 → 1.9 m

This allows the system to estimate whether an object is approaching the user.

---

📏 3. Distance Estimation

The system uses Depth Anything V2 Small for monocular depth estimation.

For each detected object:

YOLO Bounding Box
        ↓
Depth Map Crop
        ↓
Center Region Selection
        ↓
Median Depth
        ↓
Metric Calibration
        ↓
Estimated Distance

Example:

Person     → 1.8 m
Bicycle    → 2.1 m
Car        → 4.3 m

Important

Depth Anything V2 produces relative depth, not guaranteed metric distance.

Therefore, the system uses a calibration value:

distance_m = DEPTH_AT_1_METER / median_depth

The calibration value must be adjusted for the actual camera.

---

🧭 4. Direction Detection

The position of the object inside the camera frame is classified as:

LEFT
CENTER
RIGHT

Example:

Person → LEFT
Car → CENTER
Bicycle → RIGHT

Voice output:

«"Person on your left."»

---

⚠️ 5. Time To Collision (TTC)

The system estimates whether an object is getting closer.

Formula:

TTC = distance / closing_speed

Example:

Distance = 1.5 m
Closing Speed = 1.0 m/s

TTC = 1.5 seconds

A smaller TTC indicates a potentially more dangerous situation.

---

🧮 6. 8-Factor Risk Engine

The system calculates a risk score from 0 to 100.

The following factors are considered:

Factor 1 — Object Type

Different objects have different risk weights.

Car / Truck / Bus  → 1.00
Motorcycle         → 0.95
Bicycle            → 0.75
Person             → 0.65
Dog                → 0.50
Cat                → 0.40
Chair / Bench      → 0.25
Default            → 0.20

Factor 2 — Distance

Closer objects produce higher risk.

Factor 3 — Direction

Objects near the center of the camera view receive higher risk.

Factor 4 — Front / Side Position

Large objects occupying more of the camera frame are treated as potentially more relevant.

Factor 5 — Velocity / TTC

Approaching objects increase the risk.

Factor 6 — User / Ego Motion

The system estimates whether the user is:

FORWARD
BACKWARD
STATIONARY

Factor 7 — Path Blocked

The bottom-center region of the scene is analyzed to determine whether the user's path may be blocked.

Factor 8 — Detection Confidence

Low-confidence detections contribute less to the final risk score.

---

🚨 7. Four-Level Alert System

The final risk score determines the alert level.

Risk| Level| Action
0–25| 🟢 NO_EVENT| No alert
25–50| 🟡 WARNING| Sound warning
50–75| 🟠 USER_PROMPT| Voice warning
75–100| 🔴 IMMEDIATE| Emergency voice alert

---

🔊 Voice Alerts

WARNING

"Caution, bicycle nearby."

USER PROMPT

"Warning! Bicycle from left, 1.4 meters."

IMMEDIATE

"DANGER! Bicycle very close! Stop now!"

The system uses:

pyttsx3

for local text-to-speech.

No cloud TTS API is required.

---

🔁 8. Alert Debounce & Cooldown

To avoid repeating the same warning continuously, the system uses:

Debounce

An event must remain active for a certain number of frames.

WARNING       → 5 frames
USER_PROMPT   → 3 frames
IMMEDIATE     → 1 frame

Cooldown

The same alert cannot immediately repeat.

WARNING       → 4 seconds
USER_PROMPT   → 2 seconds
IMMEDIATE     → 0.5 seconds

Higher-priority alerts override lower-priority alerts.

---

🛡️ Fault Tolerance

The system contains multiple safety layers.

Layer 1 — Camera Recovery

If the webcam disconnects:

Retry → 5 times
Delay → 2 seconds

Layer 2 — Image Pre-processing

Lighting conditions:

Low Light   → Gamma Correction
Very Low     → CLAHE
Normal       → Minimal processing

Layer 3 — Detection Validation

Invalid detections are rejected using:

Confidence > 0.35
Box size > 5 pixels
Valid aspect ratio

Layer 4 — Depth Fallback

If depth estimation fails:

Use last valid depth

instead of crashing the entire system.

Layer 5 — Performance Adaptation

Depth estimation is computationally expensive.

The system adapts according to FPS:

FPS > 12
→ Depth every 3 frames

FPS 6–12
→ Depth every 5 frames

FPS < 6
→ Depth every 10 frames

---

💻 Hardware Requirements

Minimum recommended:

- Ubuntu Linux laptop
- Built-in webcam or USB webcam
- 4 GB RAM or more
- Modern CPU
- 5 GB+ free disk space

Recommended:

- 8 GB+ RAM
- Intel Core i5 / AMD Ryzen 5 or better
- NVIDIA GPU if available

The system is designed to work on CPU-only machines, although performance may be lower.

---

🐧 Supported Operating Systems

Tested target platforms:

Ubuntu 20.04 LTS
Ubuntu 22.04 LTS
Ubuntu 24.04 LTS

---

🛠️ Installation

1. Install System Dependencies

sudo apt update

sudo apt install -y \
python3 \
python3-pip \
python3-venv \
libgl1 \
libglib2.0-0 \
libsm6 \
libxext6 \
libxrender-dev \
git \
espeak

---

2. Create Project Directory

mkdir -p ~/ai_vision_system
cd ~/ai_vision_system

---

3. Create Virtual Environment

python3 -m venv venv

Activate it:

source venv/bin/activate

---

4. Install Python Dependencies

pip install --upgrade pip

pip install \
ultralytics \
opencv-python \
numpy \
lapx \
torch \
torchvision \
transformers \
huggingface_hub \
pillow \
accelerate \
pyttsx3 \
pygame

---

📁 Project Structure

ai_vision_system/
│
├── venv/
│
├── main.py
│
├── risk_alert_system.py
│
├── depth_estimator.py
│
├── preprocessor.py
│
├── config.py
│
├── requirements.txt
│
├── alert_sounds/
│   ├── warning.wav
│   └── immediate.wav
│
└── README.md

---

⚙️ Configuration

All major thresholds should be stored in:

config.py

Example:

CAMERA_INDEX = 0

FRAME_WIDTH = 640
FRAME_HEIGHT = 480

CONFIDENCE_THRESHOLD = 0.35

DEPTH_INTERVAL = 3

DEPTH_AT_1_METER = 180.0

MIN_DISTANCE = 0.1
MAX_DISTANCE = 30.0

WARNING_RISK = 25
USER_PROMPT_RISK = 50
IMMEDIATE_RISK = 75

---

▶️ Running the System

Activate the environment:

cd ~/ai_vision_system
source venv/bin/activate

Run:

python3 main.py

The webcam should open and start processing frames.

---

📷 Camera

The default camera is:

/dev/video0

OpenCV:

cap = cv2.VideoCapture(0)

If multiple cameras exist:

ls /dev/video*

You may need to change:

CAMERA_INDEX = 1

---

🧪 Distance Calibration

Place a known object approximately 1 meter from the camera.

For example:

Actual Distance = 1.0 meter

Suppose the depth system produces:

Median Depth = 200

The calibration constant can initially be adjusted so that:

200 → approximately 1 meter

The calibration must be tested with the actual webcam because monocular depth estimation is affected by:

- Camera position
- Camera field of view
- Lighting
- Object type
- Scene geometry
- Camera resolution

---

📊 Example Output

--------------------------------------------------
AI Vision Risk Assistant
--------------------------------------------------

FPS: 14.2

Object: Person
ID: 04
Distance: 1.42 m
Direction: CENTER
Closing Speed: 0.82 m/s
TTC: 1.73 s

Risk Score: 67
Alert: USER_PROMPT

Voice:
"Warning! Person from center, 1.4 meters."
--------------------------------------------------

---

🔒 Offline Architecture

The project is designed to operate locally:

Camera
   ↓
Local Ubuntu Laptop
   ↓
YOLOv8
   ↓
Depth Anything V2
   ↓
Risk Engine
   ↓
Local TTS
   ↓
User

No:

❌ Cloud API
❌ Online object detection
❌ Cloud speech recognition
❌ Cloud translation
❌ Remote server

is required for the core vision pipeline.

---

📚 Main Technologies

Object Detection

Ultralytics YOLO

https://github.com/ultralytics/ultralytics

Object Tracking

ByteTrack

https://github.com/ifzhang/ByteTrack

Depth Estimation

Depth Anything V2

https://github.com/DepthAnything/Depth-Anything-V2

Depth Anything V2 Hugging Face

https://huggingface.co/depth-anything/Depth-Anything-V2-Small-hf

Computer Vision

OpenCV

https://github.com/opencv/opencv-python

Alert Logic Reference

openpilot

https://github.com/commaai/openpilot

---

🧩 Future Improvements

Possible future features:

- Spatial audio
- Bengali voice alerts
- Multiple language voice output
- Better metric depth calibration
- Stereo camera support
- GPS integration
- Indoor navigation
- Outdoor navigation
- Better pedestrian trajectory prediction
- Personalized risk thresholds
- Wearable camera support
- Vibration feedback
- Lightweight models for low-end CPUs

---

⚠️ Limitations

This is an assistive AI prototype, not a certified mobility or safety device.

Monocular depth estimation can produce inaccurate distance estimates.

Risk prediction may fail because of:

- Poor lighting
- Occlusion
- Camera movement
- Fast-moving objects
- Incorrect object detection
- Depth estimation errors
- Unusual environments

Users should not rely on this system as their only means of navigation or safety.

---

🎓 Research Contribution

The project combines several computer vision components into a single risk-aware assistive pipeline:

Object Detection
        +
Object Tracking
        +
Monocular Depth
        +
Distance Estimation
        +
Motion Analysis
        +
TTC
        +
8-Factor Risk Model
        +
Adaptive Alert Engine
        +
Local Voice Assistance

The primary research focus is not simply detecting objects, but determining which detected objects represent an immediate collision risk and generating an appropriate audio alert.

---

👨‍💻 Project Status

Project Type: Research / Prototype
Platform: Ubuntu Linux
Processing: Local / Offline
Input: Webcam
Output: Voice + Visual Debugging
Target Users: Visually Impaired People

---

📜 License

This project is intended for educational and research purposes.

Individual third-party models and libraries remain subject to their respective licenses.
