# DriveIQ

DriveIQ is a smart driving-safety system. It watches the road (from a dashcam video or a live camera) and automatically spots **dangerous driving and road problems**, then gives the driver a **safety score** — a bit like a credit score, but for how you drive.

It looks for three things:

- **Red-light violations** — crossing while the traffic light is red.
- **Tailgating** — driving too close to the car in front of you.
- **Potholes** — holes in the road, and whether your car drives over one.

Every time something bad happens, DriveIQ marks it on the video, lowers your score, and updates your position on a **fleet leaderboard** (you vs. other drivers). It can even talk to a **Raspberry Pi** in the car to flash warnings and run a short "event capture" sequence.

---

## What DriveIQ does (in plain words)

1. You give it a video (a saved dashcam clip or a live camera feed).
2. The AI looks at every frame and finds cars, traffic lights, and potholes.
3. It decides if anything risky happened:
   - Did a car cross on red? → **Red-light violation**
   - Is the car ahead way too close? → **Tailgating warning**
   - Did the car drive over a pothole? → **Pothole hit**
4. It draws boxes and warning banners on the video so you can see it clearly.
5. It adjusts two scores:
   - A **daily score out of 100** (starts at 100, drops with each mistake).
   - A **fleet leaderboard score** (starts at 8800, ranks you against other drivers).
6. It shows everything on a clean **web dashboard** in your browser.

---

## The two main parts of the project

DriveIQ is made of two pieces that work together:

### 1. The AI Analytics app (the "eyes")
Folder: `driving-violations/`
This is the part that actually watches the video and detects violations. It's built with **Streamlit** (an easy web-app tool) and opens in your browser. You can:
- Upload a dashcam video and process it, **or**
- Run "Live ADAS" mode on a live camera/stream.

### 2. The Command Center (the "brain" + dashboard)
File: `laptop_master_withSim.py`
This is a **Flask web server** that shows the live dashboard, keeps the official scores, runs the **"13-second Blast Shield"** event sequence (more on that below), and talks to the Raspberry Pi in the car.

---

## How the scoring works

| Violation | Points lost (daily /100) | Points lost (leaderboard /8800) |
|---|---|---|
| Red-light violation | 25 | 500 |
| Tailgating | 15 | 200 |
| Pothole hit | 10 | 100 |

To keep things fair, there are some sensible limits:
- **Tailgating** can only be counted up to **2 times per session**.
- **Potholes** can only take away up to **30 points** (daily) and **800 points** (leaderboard) per session, so one bumpy road doesn't destroy your score.
- After any violation, there's a short **cooldown** so the same event isn't counted many times in a row.

The leaderboard has 9 fixed "rival" drivers, and **you** are the one whose score moves up and down based on your driving.

---

## The "13-second Blast Shield" (the event sequence)

When a violation is detected during live driving, the Command Center runs a fixed 13-second routine:

1. **0s** — Lockdown starts, the in-car camera turns ON, warning shows.
2. **7s** — Camera turns OFF, "processing" message shows.
3. **13s** — The penalty is applied, the leaderboard updates, and the new score is sent to the car's Raspberry Pi.

During these 13 seconds, no new event can start — this stops one incident from being counted over and over. (It's called a "Blast Shield" because it shields against double-counting.)

---

## What's inside the project

**Detection (the AI brains) — in `driving-violations/`**

| File | What it does |
|---|---|
| `app.py` | The Streamlit web app you open in the browser. |
| `detector.py` | The main pipeline — runs all three detectors on each video frame. |
| `red_light_detector.py` | Finds traffic lights, checks if they're red/green, and catches red-light crossings. |
| `tailgate_detector.py` | Finds the car ahead and measures if you're following too closely. |
| `pothole_detector.py` | Finds potholes and checks if your car drives over one. |
| `driver_score.py` | Manages your daily score out of 100. |
| `fleet_ranking.py` | Manages the 8800-point leaderboard and rankings. |
| `scoring_service.py` | Helper that ties scoring pieces together. |
| `live_view.py` / `process_view.py` | The "Live camera" and "Process a video" screens. |
| `pi_bridge.py` | Sends violations and scores to the Raspberry Pi / Command Center. |
| `config.py` | All the settings (thresholds, point values, limits) in one place. |
| `yolov8n.pt` | The AI model used to detect cars and traffic lights. |

**Command Center & car link — in the project root**

| File | What it does |
|---|---|
| `laptop_master_withSim.py` | The Command Center server (dashboard + official scoring + Pi link). |
| `event_orchestrator.py` | Runs the 13-second Blast Shield sequence. |
| `camera_stream_manager.py` | Handles the live camera video stream. |
| `pi_edge_node.example.py` | Example code for the Raspberry Pi side (reference only). |
| `start_driveiq.ps1` | One-click script to start everything on Windows. |
| `templates/` | The web pages for the dashboard (`portal.html`, `dashboard.html`). |

**Pothole research folder** — `pothole-detection-system-using-convolution-neural-networks-master/`
This is the original standalone pothole project (an earlier experiment). The main DriveIQ system reuses its pothole AI model.

---

## How the pieces talk to each other

```
Raspberry Pi (in car)  --UDP-->  Command Center  <--HTTP--  AI Analytics app
   sensors + camera               (scores +                 (detects red light,
   + warnings                      dashboard +               tailgating, potholes)
                                    Blast Shield)
```

- The **AI Analytics app** finds a violation and tells the **Command Center**.
- The **Command Center** runs the 13-second sequence, updates the official score, and sends the result to the **Raspberry Pi** in the car over the network (UDP).
- The **Pi** can show warnings and the latest score to the driver.

> The Raspberry Pi part is optional. You can run and test the whole detection + scoring + dashboard on just your laptop.

---

## What you need before running

- A **Windows computer** (the start script is made for Windows; the Python code itself is cross-platform).
- **Python 3.8 or newer**.
- An **internet connection** the first time (to download the AI models).
- *(Optional)* A Raspberry Pi with a camera, only if you want the full in-car setup.

---

## How to set it up and run

**Step 1 — Install the needed tools** (from inside the `driving-violations` folder):

```bash
pip install -r requirements.txt
```

**Step 2 — Start everything (easiest way on Windows):**

Double-click `start_driveiq.ps1`, or run it from PowerShell:

```powershell
.\start_driveiq.ps1
```

This opens two things:
- **Command Center / Dashboard** → http://localhost:5000
- **AI Analytics app** → http://localhost:8501

**Step 3 — Use it:**
- Open the **AI Analytics app** (port 8501), go to **"Process Video"**, upload a dashcam clip, and click **Process Video**. You'll get the marked-up video and a report of all violations.
- Or use the **"Live ADAS"** tab for a live camera/stream.
- Open the **Dashboard** (port 5000) to watch the live score, leaderboard, and event sequence.

> **Tip:** You can run the two apps separately too:
> - Command Center: `python laptop_master_withSim.py`
> - AI Analytics app: `streamlit run app.py` (inside `driving-violations/`)

---

## The tools it uses

- **Ultralytics YOLOv8** — the AI that detects cars, traffic lights, and potholes.
- **OpenCV** — reads video frames and draws the boxes/warnings.
- **Streamlit** — builds the AI Analytics web app.
- **Flask + Flask-SocketIO** — runs the live Command Center dashboard.
- **NumPy** — number-crunching behind the scenes.
- **UDP networking** — the fast link between the laptop and the Raspberry Pi.

---

## Good to know

- Use **MP4 dashcam videos** for the best results.
- **Longer videos take longer** to process — please be patient.
- The **first run is slower** because the AI models download once.
- Detection settings (how strict each detector is) can be adjusted with sliders in the AI Analytics app's sidebar.

---

## The idea behind DriveIQ

Unsafe driving — running red lights, tailgating — and bad roads full of potholes cause accidents and damage every day. DriveIQ shows how a computer can watch the road, catch these problems automatically, and turn them into a simple score that encourages safer driving. It could help fleet companies, driving schools, insurers, or city road teams keep both drivers and roads safer.
#   D r i v e I Q  
 