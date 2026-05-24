# Collaborative Robotics — Developer Setup

Setup guide for the Dobot Magician arm and Orbbec camera pick-and-place track.

For challenge context, see [Collaborative_Robotics/README.md](README.md). For the parent overview, see [DEVELOPER_SETUP.md](../DEVELOPER_SETUP.md).

**Important:** Read [Safety Instructions/README.md](Safety%20Instructions/README.md) and complete safety training before operating the arm.

---

## Quick Start (5 minutes)

1. Verify Python: `python3 --version` (3.8+ required)
2. Create virtual environment:
   ```bash
   cd Collaborative_Robotics
   python3 -m venv venv
   source venv/bin/activate        # Linux/macOS
   # venv\Scripts\activate         # Windows
   ```
3. Install Python dependencies:
   ```bash
   pip install opencv-python numpy matplotlib
   ```
4. Install Dobot Link and DobotLab (see [Safety Instructions](Safety%20Instructions/README.md))
5. Connect the Dobot arm and Orbbec camera via USB; power on the arm
6. Update the COM port in `dobotArm.py` (see Hardware Setup)
7. Verify basic arm control: `python3 testDobot.py`

---

## What You Need

### Hardware

- Dobot Magician robotic arm with gripper
- Orbbec depth/RGB camera (Astra or similar)
- Camera stand (clip mount provided in kit)
- USB cables for arm and camera
- Red velcro tags (provided) and optional 3D-printed car parts
- Clear workspace free of obstacles within the arm's range of motion

### Software

- Python 3.8+
- [Dobot Link](https://uofwaterloo-my.sharepoint.com/:u:/g/personal/mstachow_uwaterloo_ca/IQCMyzfbytafTpc4qCPQD6YwAbgJf8-Cn81QLpLcJRNMF0o?e=Ff2Rbj) — background driver service
- [DobotLab](https://uofwaterloo-my.sharepoint.com/:u:/g/personal/mstachow_uwaterloo_ca/IQBbYKSMl_JLSITR646OPVyuAQqpUINluDmnRVfS9tAhdjM?e=zsudQp) — connection testing and port discovery
- OpenCV (`opencv-python`), NumPy, Matplotlib

### Operating Systems

Windows is recommended (Dobot DLLs in `lib/` are platform-specific). Linux/macOS may require additional configuration.

---

## Software Setup

### 1. Python Virtual Environment

From `Collaborative_Robotics/`:

```bash
python3 -m venv venv
source venv/bin/activate
pip install opencv-python numpy matplotlib
```

The Dobot SDK is **not** installed via pip. The repo includes pre-built DLL bindings in `lib/` (see `lib/DobotDllType.py`). Do not modify `lib/` unless you know what you are doing.

### 2. Dobot Link and DobotLab

Follow the full procedure in [Safety Instructions/README.md](Safety%20Instructions/README.md):

1. Download and install Dobot Link and DobotLab
2. Launch Dobot Link from the system tray → **Launch Developer Interface → Device Test**
3. Connect to the arm and note the COM port
4. Launch DobotLab, select the Python tab, and click **Connect**
5. Select **Dobot Magician** if prompted for arm model

### 3. Configure COM Port

In `dobotArm.py`, find the connection line:

```python
state = dType.ConnectDobot(api, "COM7", 115200)[0]
```

Change `"COM7"` to match the port shown in DobotLab (e.g., `COM3` on Windows, `/dev/ttyUSB0` on Linux).

**Disconnect from DobotLab** before running your Python scripts — only one application can hold the serial connection at a time.

### 4. Camera Index

The starter code opens the Orbbec camera by index (typically `0` or `1`). If the camera feed is blank, try changing the index in `pickCVBlock.py` or `calibrateCamera.py`.

---

## Hardware Setup

### Workspace

1. Clear the arm's range of motion of obstacles, loose cables, and personal items (see safety doc for motion envelope diagram).
2. Mount the Orbbec camera on the provided stand so it can see both the tabletop and the arm workspace.
3. Position red velcro tags on the table within the camera field of view.

Example kit layout photos are in [README.md](README.md) under **Starter Material Introduction**.

### Arm Power-Up

1. Press the lock button and move the arm to the approximate home pose shown in the safety doc.
2. Plug in the USB cable.
3. Press the power button on the base. Wait for the initialization sweep to complete (the arm beeps when ready).

### Arm Power-Down

Follow the safety doc's shutdown procedure — the arm will drop if power is cut while motors are not in a safe pose.

---

## Camera Calibration

Calibration maps camera pixels to robot coordinates. Required before reliable pick-and-place.

### Hand-Eye Calibration (`getTransformationMatrix.py`)

1. Ensure the camera can see the full tabletop range of arm movement.
2. Run:
   ```bash
   python3 getTransformationMatrix.py
   ```
3. When prompted, place a red tag between the gripper fingers (center aligned).
4. Press **Space** as instructed. The arm moves to collect additional calibration points.
5. Repeat until the script finishes. Output is saved to `H_matrix.json`.

### Intrinsic Calibration (`calibrateCamera.py`) — Optional

Only needed if using a different camera than the kit Orbbec:

1. Print a 4×4 ArUco calibration board.
2. Run:
   ```bash
   python3 calibrateCamera.py
   ```
3. Follow on-screen prompts. Output is saved to `camera_params.npz`.

Use calibrated intrinsics in your code:

```python
data = np.load("camera_params.npz")
camera_matrix = data["camera_matrix"]
dist_coeffs = data["dist_coeffs"]
```

See [README.md](README.md) for undistortion examples.

---

## Safety Considerations

- Complete [Safety Instructions](Safety%20Instructions/README.md) before first use
- Keep hands and objects out of the arm's motion envelope during autonomous operation
- Use reduced speed during development (`testDobot.py` before `pickCVBlock.py`)
- Never modify `lib/` DLL files unless necessary
- Always use the documented shutdown procedure to avoid arm damage

---

## Validation

Run these checks in order:

### 1. Dobot Connection

```bash
python3 testDobot.py
```

Expected: arm connects, moves through basic motions, and returns without errors.

### 2. Camera Feed

Run `pickCVBlock.py` or a short OpenCV script:

```python
import cv2
cap = cv2.VideoCapture(0)
ret, frame = cap.read()
print("Frame captured:", ret, frame.shape if ret else None)
cap.release()
```

Expected: `Frame captured: True` with a valid shape (e.g., `(480, 640, 3)`).

### 3. Calibration

Run `getTransformationMatrix.py` and confirm `H_matrix.json` is created.

### 4. Pick-and-Place Demo

```bash
python3 pickCVBlock.py
```

Expected: camera detects red velcro tags and the arm executes pick-and-place motions.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| `Unable to connect to robot` | Verify COM port in `dobotArm.py`; disconnect DobotLab; re-plug USB and re-check port |
| COM port changed after unplug | Re-run DobotLab connect step and update `dobotArm.py` |
| DobotLab install incomplete | Run each installer three times (Windows may flash during install) |
| Camera not detected | Try camera index `1` instead of `0`; check USB connection |
| Blank camera frame | Ensure no other app holds the camera; try a different USB port |
| Calibration failed | Improve lighting; ensure tags are fully visible and not occluded by the arm |
| Red tags not detected | Adjust HSV thresholds in `pickCVBlock.py` → `phase_detect_targets()`; check lighting |
| `ImportError` for `DobotDllType` | Do not delete or rename files in `lib/`; run from the `Collaborative_Robotics/` directory |
| Arm drops on shutdown | Follow the power-down procedure in the safety doc |

---

## Resources

- [Collaborative Robotics README](README.md) — challenge overview and milestone roadmap
- [Safety Instructions](Safety%20Instructions/README.md) — required reading before operation
- [Dobot Official Site](https://www.dobot.cc/)
- [OpenCV Documentation](https://docs.opencv.org/)
- [Orbbec Developer Resources](https://orbbec.github.io/)
- Starter scripts: `dobotArm.py`, `pickCVBlock.py`, `testDobot.py`, `getTransformationMatrix.py`, `calibrateCamera.py`
- [Parent Developer Setup Guide](../DEVELOPER_SETUP.md)
