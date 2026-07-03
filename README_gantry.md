# Cartesian 3DoF Robot for Egg Classification & Sorting

<p align="center">
  <img src="7_Images_and_Videos/1_Final_Robot/Robot_en_la_cinta.jpg" alt="R1 Robot on conveyor belt" width="48%"/>
  &nbsp;&nbsp;
  <img src="7_Images_and_Videos/1_Final_Robot/Interfaz_Grafica.png" alt="Graphical Interface" width="48%"/>
</p>

<p align="center">
  <em>Left: R1 robot on the conveyor belt. &nbsp;|&nbsp; Right: Real-time control GUI.</em>
</p>

---

## Overview

**R1 – Estación 1** is a custom-built **3-axis Cartesian (Gantry) robot** that autonomously detects, picks up, and sorts eggs on a conveyor belt.

- **Motion**: GRBL 0.9j on Arduino UNO — 3× NEMA 17 steppers driven by A4988 drivers (7000 steps/mm per axis)
- **Vision**: Overhead smartphone camera — YOLOv8 detection + ArUco homography for cm-level localization
- **Gripper**: 3D-printed servo gripper (MG995) on Arduino MEGA, controlled via serial (`$on` / `$off`)
- **GUI**: PyQt5 app with live camera feed, 2D workspace map (27.6 × 11 cm), and terminal log

> **Quick run:** `cd 3_Code/1_PC_Python/4_Complete_Project` → `python main5.py`
> See [3_Code/README.md](3_Code/README.md) for full setup and configuration.

---

## Repository Contents

| Folder | Description | Details |
|---|---|---|
| [1_Structure/](1_Structure/) | Mechanical design — 3D models (ZIP), welded frame, ArUco platforms | [README](1_Structure/README.md) |
| [2_Components_Used/](2_Components_Used/) | Bill of materials, datasheets, hardware block diagram | [README](2_Components_Used/README.md) |
| [3_Code/](3_Code/) | All software — Python app + Arduino firmware | [README](3_Code/README.md) |
| [4_Universal_Gcode_Sender_App/](4_Universal_Gcode_Sender_App/) | UGS desktop app for manual robot jogging and testing | — |
| [5_YOLOv8_Detection_Model/](5_YOLOv8_Detection_Model/) | Training notebook, datasets, trained weights, metrics | [README](5_YOLOv8_Detection_Model/README.md) |
| [6_Gripper_3D_Model/](6_Gripper_3D_Model/) | 3D-printable gripper STL files + editable source | [README](6_Gripper_3D_Model/README.md) |
| [7_Images_and_Videos/](7_Images_and_Videos/) | Final build photos and demo videos | [README](7_Images_and_Videos/README.md) |

---

## System Architecture

<p align="center">
  <img src="2_Components_Used/R1_Hardware_Block_Diagram.png" alt="Hardware Block Diagram" width="75%"/>
</p>

<p align="center">
  <img src="3_Code/1_PC_Python/R1_Logic_Flow_Diagram_Final.drawio.png" alt="Logic Flow Diagram" width="75%"/>
</p>

---

## License

Academic robotics prototype. All source code and design files included for educational purposes.
