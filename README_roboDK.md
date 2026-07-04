# Multi-Robot Manufacturing Process Simulation — RoboDK

> A full industrial manufacturing process simulation built in **RoboDK**, featuring **3 collaborative robots**, a **conveyor belt**, a **vision system (OpenCV)**, and an intelligent **color-sorting palletizer**. Programmed entirely in Python using the RoboDK API.

---

## Simulation Overview

![Full simulation layout — 3D view](3_Multimedia/Capture.PNG)

![Alternative angle — robots and conveyor](3_Multimedia/Capture4.PNG)

---

## Project Description

This project simulates a complete multi-station robotic manufacturing cell. The system coordinates three industrial robots working in sequence to:

1. **Feed** colored cubic workpieces from a rotating wheel mechanism.
2. **Machine** the pieces and transport them onto a conveyor belt.
3. **Detect** piece color and orientation via computer vision, then sort them into color-coded pallets.

The simulation was developed as an academic project (`Trabajo Practico RoboDK`) and is fully configurable through Python scripts attached to the RoboDK station file.

---

## Repository Structure

```
RoboDK_Project/
│
├── 1_Complete_Project/
│   └── Process_simulation_with_3_robots.rdk   # Full RoboDK station file
│
├── 2_Utils/
│   ├── 1_RoboDK_Robots/
│   │   ├── ABB-IRB-140-6-0-8.robot            # ABB IRB 140 robot model
│   │   └── UR3e.robot                          # Universal Robots UR3e model
│   │
│   ├── 2_RoboDK_Tools/
│   │   ├── RobotiQ-EPick-Vacuum-Gripper-1-Cup.tool   # Vacuum gripper (Robot 3)
│   │   ├── Teknomotor-C31-40-C-DBL-ER20-Spindle.tool # Machining spindle (Robot 2)
│   │   └── Tool2.tool                                 # Gripper (Robot 2)
│   │
│   └── 3_Codes/
│       ├── 1_Main_Codes/
│       │   ├── POSICIONAR_ESTACIONES.py    # Initializes all station poses
│       │   ├── Run_Conveyor.py             # Conveyor belt simulation loop
│       │   ├── SECUENCIA_COMPLETA.py       # Master orchestration script
│       │   ├── SECUENCIA_ESTACION1.py      # Robot 1 pick-from-wheel sequence
│       │   ├── SECUENCIA_ESTACION2.py      # Robot 2 machining + conveyor load sequence
│       │   └── SECUENCIA_ESTACION3.py      # Robot 3 vision-guided palletizing sequence
│       │
│       └── 2_Auxiliary_debug_codes/
│           ├── Camara_test.py              # Standalone camera & color detection test
│           └── move_tool_TCP_to_Target.py  # IK utility / TCP positioning test
│
└── 3_Multimedia/
    ├── Capture.PNG      # Simulation screenshot 1
    ├── Capture2.PNG     # Simulation screenshot 2
    ├── Capture3.PNG     # Simulation screenshot 3
    ├── Capture4.PNG     # Simulation screenshot 4
    └── Demo_video.mp4   # Full simulation demo video
```

---

## System Architecture

The manufacturing cell is divided into **3 stations** that run concurrently and sequentially through a master program:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        MASTER: SECUENCIA_COMPLETA                        │
│  1. POSICIONAR_ESTACIONES  →  2. Run_Conveyor  →  3. SECUENCIA_ESTACION3 │
│         (init)                   (background)         (background)        │
│                                                                           │
│  Loop: SECUENCIA_ESTACION1 → SECUENCIA_ESTACION2 → repeat               │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Stations Detail

### Station 1 — Wheel Feeder (UR3e — Robot 1)
![Station 1 close-up](3_Multimedia/Capture2.PNG)
- A rotating wheel mechanism (**Mi Mecanismo**) holds colored cubic workpieces arranged radially.
- **Robot 1** (UR3e) picks one cube at a time from the wheel using a vacuum gripper (`Tool1`).
- The cube is placed on **Mesa1** (work table) using inverse-kinematics-based TCP positioning.
- After each pick, the wheel advances **15°** to present the next piece.
- Robot returns to the home position after each cycle.

**Key functions in `SECUENCIA_ESTACION1.py`:**
| Function | Description |
|---|---|
| `grab_object_with_tool(tool)` | Attaches the closest cube to the tool |
| `move_tool_TCP_to_Target(robot, tool, frame)` | Solves IK and moves robot TCP to a target frame |
| `drop_object_from_tool_to_place(tool, place)` | Detaches and places the cube on the work table |
| `Move_Wheel_step()` | Rotates the wheel -15° to advance to the next piece |
| `go_home_joints(robot)` | Returns the robot to its safe home position |

---

### Station 2 — Machining & Conveyor Loading (ABB IRB 140 — Robot 2)

- **Robot 2** (ABB IRB 140) operates with a **dual-tool** setup:
  - `Tool2_A`: Gripper for picking and placing the workpiece.
  - `Tool2_M`: Machining spindle (Teknomotor ER20).
- Sequence:
  1. Picks the cube from **Mesa1** using `Tool2_A`.
  2. Places the cube at the machining target (`Target_Mecanizado`).
  3. Switches to `Tool2_M` and performs a **milling/machining motion** (5-step joint trajectory).
  4. Re-grabs the machined piece with `Tool2_A`.
  5. Moves to the conveyor input (`Target_Conveyor_Input`) and **drops the piece onto the belt**.
  6. Applies a **random Z-axis spin** to the piece before releasing (simulating real-world orientation variability).

**Key functions in `SECUENCIA_ESTACION2.py`:**
| Function | Description |
|---|---|
| `machining(robot)` | Executes a 5-waypoint milling trajectory |
| `random_spin(tool, range)` | Randomly rotates tool TCP around Z-axis (0°–90°) |
| `go_convetor_joints(robot)` | Moves robot to the conveyor drop-off pose |

---

### Station 3 — Vision-Guided Palletizing (UR3e — Robot 3)

![Station 3 top view](3_Multimedia/Capture3.PNG)

- **Robot 3** (UR3e) monitors two virtual sensors:
  - `Sensor_Camara`: Triggers image capture when a cube passes.
  - `Sensor_Robot3`: Triggers the robot to grip when the piece is in reach.
- A **virtual camera** (`Camera 1`) captures an image of the approaching cube.
- **OpenCV** processes the image to detect:
  - **Color** (Red / Green / Blue) via HSV color space masking.
  - **Orientation angle** via `cv2.minAreaRect()` on the largest detected contour.
- The robot **adjusts its wrist joint (J6)** to compensate for the detected angle before grabbing.
- The piece is then sorted into the matching color pallet using the `Palletizer` class:
  - `Pallet_Verde` (Green)
  - `Pallet_Azul` (Blue)
  - `Pallet_Rojo` (Red)
- Each pallet stacks pieces in a **4×4 grid**, with automatic layer advancement (Z stacking).

**Key classes/functions in `SECUENCIA_ESTACION3.py`:**
| Component | Description |
|---|---|
| `detectar_cubo(frame)` | Detects color and orientation angle via OpenCV HSV masking |
| `capture_frame()` | Captures image bytes directly from RoboDK virtual camera |
| `class Palletizer` | Manages 4-position pallet layout with Z-layer stacking |
| `Palletizer.advance()` | Returns next absolute pose for placing a piece |
| `sensor_detecta(sensor, obj)` | Checks collision between a sensor object and a cube |

---

## Technologies & Dependencies

| Tool | Version / Notes |
|---|---|
| **RoboDK** | Simulation & robot programming platform |
| **Python** | 3.x (embedded in RoboDK) |
| **robolink** | RoboDK Python API — station communication |
| **robodk** | RoboDK math utilities (transforms, kinematics) |
| **OpenCV (`cv2`)** | Computer vision — color detection & angle estimation |
| **NumPy** | Array operations for image processing |

### Robot Models Used
| Robot | Model | Station |
|---|---|---|
| Robot 1 | Universal Robots **UR3e** | Station 1 (Wheel Feeder) |
| Robot 2 | **ABB IRB 140-6/0.8** | Station 2 (Machining) |
| Robot 3 | Universal Robots **UR3e** | Station 3 (Palletizer) |

---

## How to Run

### Prerequisites
- [RoboDK](https://robodk.com/) installed (trial or licensed).
- Python packages: `opencv-python`, `numpy`.
  ```bash
  pip install opencv-python numpy
  ```

### Steps

1. **Open the station file** in RoboDK:
   ```
   1_Complete_Project/Process_simulation_with_3_robots.rdk
   ```

2. **Run the master program** from within RoboDK:
   - In the station tree, right-click **`SECUENCIA_COMPLETA`** → *Run Python Script*.
   - This will automatically:
     - Call `POSICIONAR_ESTACIONES` to set up all object poses.
     - Launch the conveyor belt simulation (`Run_Conveyor`).
     - Start the vision-guided palletizer (`SECUENCIA_ESTACION3`).
     - Enter the main loop cycling through Station 1 and Station 2 sequences.

3. **Individual programs** can also be run independently for testing:
   - `SECUENCIA_ESTACION1` — test Robot 1 pick cycle.
   - `SECUENCIA_ESTACION2` — test Robot 2 machining cycle.
   - `SECUENCIA_ESTACION3` — test Robot 3 vision + palletizing cycle.

### Debug/Test Scripts
| Script | Purpose |
|---|---|
| `Camara_test.py` | Standalone test: captures a frame and runs color+angle detection |
| `move_tool_TCP_to_Target.py` | Quickly validates IK TCP positioning for any robot/tool/frame |

---

## Key Features

- ✅ **Full multi-robot coordination** — 3 robots working in a synchronized production loop.
- ✅ **Inverse Kinematics (IK)** — TCP-to-target positioning computed dynamically via `SolveIK`.
- ✅ **Virtual conveyor belt** — real-time object tracking, pickup detection, and fall-off handling.
- ✅ **Computer vision integration** — RoboDK virtual camera + OpenCV for color and angle detection.
- ✅ **Angle-compensated grasping** — Robot 3 adjusts its wrist orientation based on the detected piece angle before gripping.
- ✅ **Automatic palletizing** — `Palletizer` class handles grid layout and multi-layer Z stacking for each color.
- ✅ **Programmatic scene setup** — All object poses, robot homes, and tool TCPs are set programmatically via `POSICIONAR_ESTACIONES`.

---

## Academic Context

This project was developed as part of a practical assignment (*Trabajo Práctico*) in industrial robotics. The full project report is included in [`Trabajo Practico RoboDK.pdf`](Trabajo%20Practico%20RoboDK.pdf).

---

## Demo

Watch the full simulation demo on YouTube:

[![Watch the demo on YouTube](https://img.youtube.com/vi/gurtF5G5MJY/maxresdefault.jpg)](https://youtu.be/gurtF5G5MJY)

> Local copy also available at [`3_Multimedia/Demo_video.mp4`](3_Multimedia/Demo_video.mp4).

