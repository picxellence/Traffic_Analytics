# 🚦 Near-Miss Traffic Analytics — Smart City Road Safety System

A real-time computer vision system that monitors traffic footage and automatically detects **near-miss events** — dangerous moments where a collision between road users nearly occurred. Built using pretrained YOLO models with no custom training.

---

## 📽️ Demo

> Annotated output video with live risk scoring, near-miss detection, and dashboard overlay.

- 🟢 Green box → Normal object  
- 🟡 Yellow box → Interacting pair  
- 🔴 Red flashing box → Near-miss event detected  

---

## 🧠 How It Works

The system runs a **6-stage pipeline** on every frame of a traffic video:

```
Input Video
    │
    ▼
Stage 1 — Object Detection       (detect road users: car, truck, bus, motorcycle, bicycle, person)
    │
    ▼
Stage 2 — Object Tracking        (assign persistent IDs using ByteTrack)
    │
    ▼
Stage 3 — Proximity Detection    (find interacting pairs using nearest-edge distance)
    │
    ▼
Stage 4 — Risk Scoring           (score 0–100 based on distance + object type + relative velocity)
    │
    ▼
Stage 5 — Near-Miss Detection    (sustained HIGH risk for N frames = near-miss event)
    │
    ▼
Stage 6 — Alert & Dashboard      (visual overlays, flashing alerts, live risk monitor)
```

---

## 🔍 Detection Pipeline Details

### Stage 1 — Object Detection
- Model: `yolo26n.pt` (CPU) / `yolo26m.pt` (GPU)
- Detects only road users: `person, bicycle, car, motorcycle, bus, truck`
- Confidence threshold: `0.25`

### Stage 2 — Object Tracking
- Algorithm: **ByteTrack** (`bytetrack.yaml`)
- Each object gets a persistent track ID across frames
- Stores last 30 center positions per track for velocity estimation

### Stage 3 — Proximity Detection
- Method: **Nearest-edge distance** between bounding boxes
- Threshold scales with object size to handle camera depth
- Only vehicle-involving pairs are checked

### Stage 4 — Risk Scoring (0–100)
| Factor | Weight | Description |
|---|---|---|
| Distance | 30% | Closer = higher risk, scaled to object size |
| Object Type | 20% | Person + vehicle = highest risk |
| Relative Velocity | 50% | Moving toward each other = dominant risk signal |

Risk Levels:
- `LOW` → score < 45
- `MEDIUM` → score 45–69
- `HIGH` → score ≥ 70

### Stage 5 — Near-Miss Event Definition
- A near-miss requires **12 consecutive HIGH-risk frames** (~0.4 seconds)
- Only triggers when objects are actively moving toward each other
- **120-frame cooldown** per pair after each event to avoid duplicates
- Stationary or slow-moving objects are filtered out via velocity check

### Stage 6 — Alert System
- Red flashing bounding box on near-miss objects
- Risk score badge between interacting pairs
- Countdown bar showing how long alert remains active
- Bottom banner: `⚠ NEAR-MISS DETECTED`
- Top-right dashboard: near-miss count + current risk level

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| [Ultralytics YOLO](https://github.com/ultralytics/ultralytics) | Object detection + tracking |
| OpenCV | Frame processing, annotation, video I/O |
| NumPy | Velocity calculations, distance math |
| ByteTrack | Multi-object tracking algorithm |
| Jupyter Notebook | Development environment |

---

## 📁 Project Structure

```
traffic_analytics/
│
├── traffic.mp4                  # Input traffic video
├── yolo26n.pt                   # YOLO nano model (CPU)
├── yolo26m.pt                   # YOLO medium model (GPU)
├── near_miss_analytics.ipynb    # Main Jupyter notebook
└── output/
    └── full_pipeline_output.mp4 # Annotated output video
```

---

## 🚀 Getting Started

### 1. Clone the repository
```bash
git clone https://github.com/yourusername/near-miss-traffic-analytics.git
cd near-miss-traffic-analytics
```

### 2. Create environment and install dependencies
```bash
conda create -n yolo python=3.10
conda activate yolo
pip install ultralytics opencv-python numpy
```

### 3. Add your files
- Place your traffic video as `traffic.mp4`
- Place `yolo26n.pt` or `yolo26m.pt` in the root folder

### 4. Run the notebook
```bash
jupyter notebook near_miss_analytics.ipynb
```
Run cells in order — each stage builds on the previous one.

---

## ⚙️ Configuration

You can tune these values at the top of the pipeline cell:

```python
HIGH_RISK_FRAMES_NEEDED = 12    # consecutive HIGH frames to trigger near-miss
COOLDOWN_FRAMES         = 120   # frames before same pair can trigger again
PROXIMITY_SCALE         = 1.0   # how close boxes must be (lower = stricter)
DISPLAY_DURATION        = 45    # frames to keep alert box visible
```

---

## 📊 Output

The system produces:
- ✅ Annotated output video with color-coded bounding boxes
- ✅ Real-time risk score between interacting pairs
- ✅ Near-miss event counter on screen
- ✅ Terminal log of every near-miss event with timestamp

Example terminal output:
```
✅ Done! Saved: output/full_pipeline_output.mp4
📊 Total near-miss events: 3

📋 Near-Miss Log:
   Event #1 | Pair (12, 47) | Frame 340  | 11.3s
   Event #2 | Pair (23, 61) | Frame 892  | 29.7s
   Event #3 | Pair (8,  34) | Frame 1450 | 48.3s
```

---

## 🎨 Designer's Choices

### Proximity Method
Used **nearest-edge distance** (Option C from the brief) scaled by average object size. This automatically adjusts for camera depth — objects far from the camera appear smaller so their proximity threshold shrinks proportionally.

### Risk Formula
Velocity is the **dominant factor (50%)** in the risk score. This prevents false positives from stationary vehicles that happen to be parked close together — a common scenario at busy intersections. Two objects only score HIGH if they are both close AND actively moving toward each other.

### Near-Miss Definition
A near-miss requires 12 consecutive HIGH-risk frames. A single spike is not an event — it must be a sustained dangerous condition. The 120-frame cooldown prevents the same pair from flooding the counter.

---

## ⚠️ Limitations

- Works best with **overhead or elevated angle** footage
- Minimum recommended resolution: **720p**
- Dense scenes (many overlapping vehicles) may cause tracking ID flickers
- Tuk-tuks and non-standard vehicles may be misclassified as cars

---

## 📜 Rules Followed

- ✅ Only pretrained YOLO models used — no training or fine-tuning
- ✅ Risk formula and scoring logic designed independently
- ✅ LLMs used only for debugging and syntax help
- ✅ All Designer's Choice sections documented

---

## 🙏 Acknowledgements

- [Ultralytics](https://ultralytics.com) for the YOLO framework
- [Pexels](https://www.pexels.com) for free stock traffic footage
- Project brief by YOLO Session AI Project series
