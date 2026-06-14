# 🚗 DriveIQ

DriveIQ is an AI-powered driving safety and fleet intelligence system that analyzes dashcam videos or live camera feeds to detect unsafe driving behavior and road hazards in real time. It automatically identifies red-light violations, tailgating, and potholes, updates driver safety scores, maintains fleet rankings, and provides live monitoring through a web dashboard.

## ✨ Features

- 🚦 Detects **red-light violations** using traffic light and vehicle analysis.
- 🚗 Detects **tailgating** by monitoring following distance between vehicles.
- 🕳️ Detects **potholes** and determines whether the vehicle drives over them.
- 🎥 Supports both **recorded dashcam videos** and **live camera feeds**.
- 🤖 Uses **YOLOv8** and computer vision for real-time object detection.
- 📊 Maintains a **daily driver safety score** (starting at 100).
- 🏆 Maintains a **fleet leaderboard score** (starting at 8800).
- ⚠️ Automatically applies penalties for unsafe driving events.
- 🛡️ Includes a **13-second Blast Shield** mechanism to prevent duplicate event counting.
- 🌐 Provides a live **Command Center dashboard** for monitoring scores and rankings.
- 📡 Supports **Raspberry Pi integration** for in-vehicle warnings and score updates.
- 🔄 Displays annotated videos with detection boxes, alerts, and violation markers.
- ⚙️ Built using **Python, Streamlit, Flask, OpenCV, NumPy, and YOLOv8**.
- 🚀 Suitable for fleet management, insurance analytics, driving schools, and road safety applications.

## 📈 Scoring System

| Violation | Daily Score Penalty | Fleet Score Penalty |
|------------|-------------------|-------------------|
| Red-Light Violation | -25 | -500 |
| Tailgating | -15 | -200 |
| Pothole Hit | -10 | -100 |

### Safety Rules

- Tailgating can be counted a maximum of **2 times per session**.
- Pothole penalties are capped per session to avoid excessive deductions.
- Cooldown periods prevent repeated counting of the same event.
- The Blast Shield mechanism blocks duplicate events during processing.

## 🏗️ System Architecture

```text
Raspberry Pi (Vehicle)
        │
        │ UDP
        ▼
Command Center (Flask Dashboard)
        │
        │ HTTP
        ▼
AI Analytics App (Streamlit + YOLOv8)
```

## 🛠️ Technology Stack

- Python
- Ultralytics YOLOv8
- OpenCV
- Streamlit
- Flask
- Flask-SocketIO
- NumPy
- Raspberry Pi Integration
- UDP Networking

## 📂 Main Components

### AI Analytics App (`driving-violations/`)

- Processes uploaded videos
- Runs live ADAS mode
- Detects violations
- Generates annotated output videos
- Sends events to the Command Center

### Command Center (`laptop_master_withSim.py`)

- Maintains official scores
- Updates fleet rankings
- Runs the Blast Shield event workflow
- Provides the live dashboard
- Communicates with the Raspberry Pi

## 🚀 Installation

Install dependencies:

```bash
cd driving-violations
pip install -r requirements.txt
```

## ▶️ Running DriveIQ

### Start Command Center

```bash
python laptop_master_withSim.py
```

### Start AI Analytics App

```bash
cd driving-violations
streamlit run app.py
```

Or run everything using:

```powershell
.\start_driveiq.ps1
```

## 💡 Applications

- Fleet Management
- Logistics Operations
- Driving Schools
- Insurance Risk Assessment
- Smart City Road Monitoring
- Driver Safety Analytics

## 🔮 Future Enhancements

- Lane Departure Detection
- Speed Limit Violation Detection
- Driver Drowsiness Monitoring
- GPS-Based Route Analytics
- Mobile Application Support
- Cloud Fleet Management

---

**DriveIQ combines AI, computer vision, and real-time analytics to promote safer driving and smarter fleet management.**
