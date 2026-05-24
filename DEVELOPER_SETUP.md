# Developer Setup Guide

This repository contains three independent hackathon subproblems. You only need to set up the track your team is working on — each has its own hardware, dependencies, and workflow.

| Subproblem | Focus | Detailed Guide |
|---|---|---|
| **Autonomous Fleets** | Multi-robot coordination, PRIZM/Arduino firmware, fleet GUI | [Autonomous_Fleets/SETUP.md](Autonomous_Fleets/SETUP.md) |
| **Collaborative Robotics** | Dobot Magician arm + Orbbec camera, pick-and-place | [Collaborative_Robotics/SETUP.md](Collaborative_Robotics/SETUP.md) |
| **Fault Prediction** | ML on telemetry data, optional motor stall test rig | [Fault_Prediction/SETUP.md](Fault_Prediction/SETUP.md) |

---

## Quick-Start Checklist

### Autonomous Fleets (~30–60 min with hardware)

1. Install Python 3.8+ and the [Arduino IDE](https://www.arduino.cc/en/software)
2. Install the TETRIX PRIZM library ZIP in Arduino IDE
3. Upload firmware to the PRIZM controller (start with `testing-bot-controls.ino`)
4. Create a Python venv and run `pip install -r Autonomous_Fleets/python_scripts/requirements.txt`
5. Copy and edit `robot_a.env` with your serial port and robot ID
6. Start `central-arbiter.py`, then `client.py --env ../robot_a.env`

### Collaborative Robotics (~45–60 min with hardware)

1. Complete [Safety Instructions](Collaborative_Robotics/Safety%20Instructions/README.md) before powering on the arm
2. Install Dobot Link and DobotLab (see safety doc for download links)
3. Install Python 3.8+, create a venv, install `opencv-python`, `numpy`, `matplotlib`
4. Connect the Dobot arm and Orbbec camera via USB
5. Update the COM port in `dobotArm.py`
6. Run camera calibration (`getTransformationMatrix.py`), then test with `testDobot.py`

### Fault Prediction (~15–30 min, no hardware required)

1. Install Python 3.8+ and create a virtual environment
2. Install ML libraries: `scikit-learn`, `pandas`, `numpy`, `matplotlib`, `jupyter`
3. Download the [Toyota telemetry dataset](https://drive.google.com/drive/folders/1WH95WIw2kX9aDbsBe2MpxEcpnStesaIB?usp=sharing) from Google Drive
4. Verify imports: `python3 -c "import sklearn, pandas; print('OK')"`
5. (Optional) Set up the [Motor Stall Test Setup](Fault_Prediction/MotorStallTestSetup/README.md) for live data collection

---

## Requirements Matrix

| Subproblem | OS | Python | Hardware | Key Software | Est. Setup Time |
|---|---|---|---|---|---|
| Autonomous Fleets | Windows / macOS / Linux | 3.8+ | PRIZM controller, Arduino Uno-compatible board, robot platform, USB cable | Arduino IDE, PRIZM library, Python venv | 30–60 min |
| Collaborative Robotics | Windows recommended | 3.8+ | Dobot Magician, Orbbec camera, camera stand, USB cables | Dobot Link, DobotLab, OpenCV | 45–60 min |
| Fault Prediction | Windows / macOS / Linux | 3.8+ | None required; optional Arduino motor test rig | Jupyter, scikit-learn, PyTorch or TensorFlow | 15–30 min |

---

## Pre-Setup Checklist (All Tracks)

Install these before starting any subproblem:

- **Python 3.8+** — verify with `python3 --version`
- **pip** — verify with `python3 -m pip --version`
- **git** — to clone and pull this repository
- **Code editor** — VS Code with the Python extension is recommended
- **Virtual environment** — use `python3 -m venv venv && source venv/bin/activate` (Linux/macOS) or `venv\Scripts\activate` (Windows)

### USB / Serial Drivers

Many robot controllers use CH340 or PL2303 USB-serial chips. If your port does not appear:

- **Windows:** Check Device Manager under **Ports (COM & LPT)**
- **Linux:** Check `/dev/ttyUSB*` or `/dev/ttyACM*` with `ls /dev/tty*`
- **macOS:** Check `/dev/cu.usbserial*` or `/dev/cu.usbmodem*`

---

## Troubleshooting Index

| Symptom | Likely Cause | See |
|---|---|---|
| `python3: command not found` | Python not installed or not on PATH | Pre-Setup Checklist above |
| `pip: command not found` | pip not installed | Use `python3 -m pip install ...` |
| Serial port not found | Wrong port, missing driver, or cable issue | [Autonomous_Fleets/SETUP.md](Autonomous_Fleets/SETUP.md#troubleshooting) |
| `ModuleNotFoundError` after pip install | Wrong venv or missing requirements | Per-subproblem SETUP.md → Software Setup |
| Cannot connect to Dobot arm | Wrong COM port or DobotLab still connected | [Collaborative_Robotics/SETUP.md](Collaborative_Robotics/SETUP.md#troubleshooting) |
| Camera not detected | Driver or USB issue | [Collaborative_Robotics/SETUP.md](Collaborative_Robotics/SETUP.md#troubleshooting) |
| `Connection refused` on fleet client | Arbiter not running or wrong IP | [Autonomous_Fleets/SETUP.md](Autonomous_Fleets/SETUP.md#troubleshooting) |
| PyTorch / sklearn install fails | Platform-specific wheel issue | [Fault_Prediction/SETUP.md](Fault_Prediction/SETUP.md#troubleshooting) |
| Dataset file not found | Data not downloaded from Google Drive | [Fault_Prediction/SETUP.md](Fault_Prediction/SETUP.md#dataset-access) |

---

## Detailed Setup Guides

- [Autonomous_Fleets/SETUP.md](Autonomous_Fleets/SETUP.md) — PRIZM firmware, Python fleet stack, `.env` configuration
- [Collaborative_Robotics/SETUP.md](Collaborative_Robotics/SETUP.md) — Dobot arm, Orbbec camera, calibration workflow
- [Fault_Prediction/SETUP.md](Fault_Prediction/SETUP.md) — ML environment, dataset access, motor test rig

For challenge context and rubric, see the [main README](README.md).
