# Autonomous Fleets — Developer Setup

Setup guide for the fleet coordination track: PRIZM/Arduino firmware, Python central arbiter, and robot client bridge.

For challenge context, see [Autonomous_Fleets/README.md](README.md). For the parent overview, see [DEVELOPER_SETUP.md](../DEVELOPER_SETUP.md).

---

## Quick Start (5 minutes)

1. Verify Python: `python3 --version` (3.8+ required)
2. Create virtual environment:
   ```bash
   cd Autonomous_Fleets/python_scripts
   python3 -m venv venv
   source venv/bin/activate        # Linux/macOS
   # venv\Scripts\activate         # Windows
   ```
3. Install dependencies: `pip install -r requirements.txt`
4. Connect the PRIZM controller via USB
5. Copy and edit an environment file:
   ```bash
   cp ../robot_a.env ../robot_a.local.env
   # Edit LOCAL_SERIAL_PORT and ROBOT_ID
   ```
6. Test serial communication (with firmware uploaded):
   ```bash
   python3 auto-send-serial-example.py
   ```

---

## What You Need

### Hardware

- TETRIX PRIZM motor controller (Arduino Uno-compatible)
- Robot platform with motors, gripper servo, and optional ultrasonic sensors
- USB cable (data-capable, not charge-only)
- Laptop for each robot (client) plus one operator laptop (arbiter)
- Optional: ESP32 for wireless bridge

### Software

- Python 3.8+
- Arduino IDE
- [TETRIX PRIZM Arduino library ZIP](https://go.pitsco.com/tetrix-prizm-legacy-resources?#downloads)
- USB serial drivers (CH340 or PL2303, if needed)

### Operating Systems

Windows, macOS, and Linux are all supported. Serial port names differ by OS (see Hardware Setup).

---

## Software Setup

### 1. Python Virtual Environment

From `Autonomous_Fleets/python_scripts/`:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Packages installed:

| Package | Used by |
|---|---|
| `pyserial` | Serial communication with PRIZM |
| `python-dotenv` | Loading `.env` configuration in `client.py` |
| `matplotlib` | Fleet GUI arena visualization |
| `keyboard` | `auto-send-serial-example.py` manual drive testing |

### 2. Environment Configuration

Each robot laptop uses an `.env` file passed to `client.py`. Example files are provided at the repo root of this folder:

- `robot_a.env` — wired robot A
- `robot_b.env` — robot B (example with WiFi bridge enabled)

Copy one and edit for your machine:

```bash
cp robot_a.env robot_a.local.env
```

Key variables:

| Variable | Description | Example |
|---|---|---|
| `SERVER_HOST_IP_ADDRESS` | IP of the laptop running `central-arbiter.py` | `127.0.0.1` |
| `SERVER_PORT` | TCP port for the arbiter | `9000` |
| `ROBOT_ID` | Robot identity (must match firmware) | `robot_A` |
| `CLIENT_NAME` | Human-readable laptop name | `robot_A_laptop` |
| `LOCAL_SERIAL_PORT` | Serial device for PRIZM | `COM5` (Windows) or `/dev/ttyUSB0` (Linux) |
| `SERIAL_BAUD` | Baud rate | `115200` |
| `USE_WIFI_BRIDGE` | Use ESP32 instead of USB serial | `false` or `true` |
| `ESP32_HOST` | ESP32 IP (wireless only) | `10.37.85.99` |
| `ESP32_PORT` | ESP32 TCP port (wireless only) | `81` |

Run the client:

```bash
cd python_scripts
python3 client.py --env ../robot_a.local.env
```

---

## Hardware Setup

### PRIZM Board and Arduino IDE

1. Download the [PRIZM Arduino library ZIP](https://go.pitsco.com/tetrix-prizm-legacy-resources?#downloads).
2. In Arduino IDE: **Sketch → Include Library → Add .ZIP Library...**
3. Restart Arduino IDE if the library does not appear.
4. Connect the PRIZM controller via USB.
5. Select **Tools → Board → Arduino AVR Boards → Arduino Uno** (PRIZM emulates Uno).
6. Select **Tools → Port** — choose the port that appears when you plug in the controller.
   - Windows: `COM3`, `COM5`, etc.
   - Linux: `/dev/ttyUSB0` or `/dev/ttyACM0`
   - macOS: `/dev/cu.usbserial-*` or `/dev/cu.usbmodem*`
7. If multiple ports appear, unplug and reconnect the controller to identify the correct one.

Reference: [TETRIX PRIZM Coding Essentials (PDF)](https://asset.pitsco.com/sharedimages/resources/44720_tetrix_prizm_codingessentialssg.pdf)

### Firmware Selection

Choose a sketch from `prizm_firmware/`:

| Sketch | Use case |
|---|---|
| `testing-bot-controls/testing-bot-controls.ino` | Manual drive and gripper testing over serial |
| `telemetry_and_communicate_to_aribiter/telemetry_and_communicate_to_aribiter.ino` | Full telemetry, waypoints, arbiter communication (wired) |
| `wireless-bot-controls/wireless-bot-controls.ino` | Same as above, via ESP32 WiFi bridge |

Before uploading, confirm `#include <PRIZM.h>` compiles without errors.

### Wiring Notes

- Ensure motor directions, servo positions, and ultrasonic sensor pins match your physical setup
- Review constants in the `.ino` file if robot geometry or sensor layout differs from the starter kit
- Set `ROBOT_ID` in the Arduino sketch to match your `.env` file

---

## Validation

Run these checks in order:

### 1. Firmware Upload

Upload `testing-bot-controls.ino` and open the Arduino Serial Monitor at **115200 baud**. Confirm the board responds to serial commands.

### 2. Serial Test (Python)

With firmware running:

```bash
cd python_scripts
python3 auto-send-serial-example.py
```

Use keyboard controls to verify forward, backward, turn, and stop. Update the serial port in the script if needed.

### 3. Fleet Stack

Terminal 1 — start the arbiter (operator laptop):

```bash
cd python_scripts
python3 central-arbiter.py
```

Terminal 2 — start a robot client:

```bash
cd python_scripts
python3 client.py --env ../robot_a.local.env
```

Expected: the GUI opens, the robot registers with a `hello` message, and telemetry appears in the dashboard.

### 4. GUI Commands

From the arbiter GUI:

- Pause, resume, or stop the robot
- Toggle the gripper
- Send a straight-line or L-shaped test path

---

## Troubleshooting

| Problem | Solution |
|---|---|
| `python3: command not found` | Install Python 3.8+ and ensure it is on your PATH |
| `pip: command not found` | Use `python3 -m pip install -r requirements.txt` |
| Serial port error / permission denied | Check USB cable, install CH340/PL2303 driver; on Linux run `sudo usermod -aG dialout $USER` and re-login |
| `ModuleNotFoundError: No module named 'serial'` | Activate venv and run `pip install -r requirements.txt` |
| Arduino upload fails | Close Serial Monitor, double-press RESET to enter bootloader, retry |
| `#include <PRIZM.h>` not found | Re-install PRIZM ZIP library in Arduino IDE |
| `Connection refused` on client | Start `central-arbiter.py` first; verify `SERVER_HOST_IP_ADDRESS` |
| Telemetry reaches client but not GUI | Check TCP logs; verify arbiter is listening on port 9000 |
| Robot ignores commands | Ensure `ROBOT_ID` in `.env` matches the firmware; check baud rate is `115200` |
| Path motion looks wrong | Recalibrate wheel diameter, wheel base, and motor directions in the `.ino` file |

---

## Resources

- [Autonomous Fleets README](README.md) — challenge description and milestone roadmap
- [Python Starter Code Tutorial](python_scripts/README.md) — protocol, client, arbiter, and GUI details
- [Message Protocol](python_scripts/MESSAGE_PROTOCOL.md) — newline-delimited JSON message format
- [Arduino Official Docs](https://docs.arduino.cc/)
- [PRIZM Legacy Resources](https://go.pitsco.com/tetrix-prizm-legacy-resources?#downloads)
- [Raspberry Pi / Computer Vision Tutorial](rasp_pi_tutorial/README.md) — optional vision extension
- [Parent Developer Setup Guide](../DEVELOPER_SETUP.md)
