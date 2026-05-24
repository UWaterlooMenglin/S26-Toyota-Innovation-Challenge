# Fault Prediction — Developer Setup

Setup guide for the machine learning and fault detection track.

For challenge context, see [Fault_Prediction/README.md](README.md). For the parent overview, see [DEVELOPER_SETUP.md](../DEVELOPER_SETUP.md).

---

## Quick Start (5 minutes)

1. Verify Python: `python3 --version` (3.8+ required)
2. Create virtual environment:
   ```bash
   cd Fault_Prediction
   python3 -m venv venv
   source venv/bin/activate        # Linux/macOS
   # venv\Scripts\activate         # Windows
   ```
3. Install ML dependencies:
   ```bash
   pip install scikit-learn pandas numpy matplotlib jupyter
   ```
4. (Optional) Install PyTorch:
   ```bash
   pip install torch torchvision torchaudio
   ```
5. Download the [Toyota telemetry dataset](https://drive.google.com/drive/folders/1WH95WIw2kX9aDbsBe2MpxEcpnStesaIB?usp=sharing) from Google Drive
6. Verify your environment:
   ```bash
   python3 -c "import sklearn, pandas, numpy; print('OK')"
   ```
7. (Optional) Explore sample motor data:
   ```bash
   python3 MotorStallTestSetup/plotValues.py
   ```

---

## What You Need

### Software (Required)

- Python 3.8+
- pip and git
- Core ML libraries: `scikit-learn`, `pandas`, `numpy`, `matplotlib`
- Jupyter Notebook (recommended for experimentation)
- Deep learning framework (optional): PyTorch or TensorFlow

### Hardware (Optional)

- Arduino (Uno or compatible) for the [Motor Stall Test Setup](MotorStallTestSetup/README.md)
- Shunt resistor, 5V DC motor, flyback diode, external 5V power supply
- USB cable for Arduino data logging

### Operating Systems

Windows, macOS, and Linux are all supported. No special drivers are required unless using the optional motor test rig (standard Arduino USB-serial).

---

## Software Setup

### 1. Python Virtual Environment

From `Fault_Prediction/`:

```bash
python3 -m venv venv
source venv/bin/activate
pip install scikit-learn pandas numpy matplotlib jupyter
```

### 2. Deep Learning Framework (Optional)

Install **one** of the following depending on your approach:

**PyTorch:**

```bash
pip install torch torchvision torchaudio
```

If the default install fails, use the [PyTorch install selector](https://pytorch.org/get-started/locally/) for your platform.

**TensorFlow:**

```bash
pip install tensorflow
```

### 3. Jupyter Kernel

If Jupyter cannot find your venv kernel:

```bash
python3 -m ipykernel install --user --name=fault-prediction --display-name="Fault Prediction"
jupyter notebook
```

### 4. Code Editor

VS Code with the Python and Jupyter extensions is recommended for notebook and script development.

---

## Dataset Access

### Toyota Telemetry Data (Primary)

Download from [Google Drive](https://drive.google.com/drive/folders/1WH95WIw2kX9aDbsBe2MpxEcpnStesaIB?usp=sharing).

The dataset contains time-series telemetry from an 8-DOF robotic manipulator:

- Joint positions and end-effector pose
- Per-joint current, temperature, torque, and load percentage
- Timestamps for sequential / time-series modeling

Store downloaded files in a local `data/` folder (not committed to git) and reference paths in your notebooks or scripts.

### Sample Data (In Repo)

| File | Description |
|---|---|
| `MotorStallTestSetup/data/sample_data.csv` | Example current measurements from the motor stall rig |

Load sample data:

```python
import pandas as pd
df = pd.read_csv("MotorStallTestSetup/data/sample_data.csv")
print(df.head())
```

### Open-Source Alternatives

You may also use public datasets. Examples from the challenge README:

- [NASA Turbofan Jet Engine (C-MAPSS)](https://www.kaggle.com/datasets/behrad3d/nasa-cmaps)
- [Azure Predictive Maintenance](https://www.kaggle.com/datasets/arnabbiswas1/microsoft-azure-predictive-maintenance)
- [AI4I 2020 Predictive Maintenance (UCI)](https://archive.ics.uci.edu/dataset/601/ai4i+2020+predictive+maintenance+dataset)

Additional sources: [Kaggle](https://www.kaggle.com/datasets), [AWS Open Data Registry](https://registry.opendata.aws/), [Microsoft Research Open Data](https://msropendata.com/).

---

## Optional Hardware Setup

The [Motor Stall Test Setup](MotorStallTestSetup/README.md) lets you collect live fault data from a stalling DC motor.

### Components

- Arduino (Uno or compatible)
- Shunt resistor (≈ 0.1Ω – 1Ω)
- 5V DC motor with flyback diode
- External 5V power supply
- Jumper wires and USB cable

### Setup Steps

1. Wire the circuit per the schematic in [MotorStallTestSetup/README.md](MotorStallTestSetup/README.md)
2. Upload `MotorStallTestSetup/motorStallTestSetup.ino` via Arduino IDE
3. Edit the COM port in `MotorStallTestSetup/saveValues.py` (line 6)
4. Install motor rig dependencies:
   ```bash
   pip install pyserial matplotlib
   ```
5. Run data collection:
   ```bash
   cd MotorStallTestSetup
   python3 saveValues.py
   ```
6. Visualize logged data:
   ```bash
   python3 plotValues.py
   ```

Output CSV files are saved to `MotorStallTestSetup/data/`.

---

## Validation

Run these checks to confirm your setup:

### 1. Python Imports

```bash
python3 -c "import sklearn, pandas, numpy, matplotlib; print('Core ML stack OK')"
```

With PyTorch (if installed):

```bash
python3 -c "import torch; print('PyTorch', torch.__version__)"
```

### 2. Load Sample Data

```bash
python3 -c "import pandas as pd; df=pd.read_csv('MotorStallTestSetup/data/sample_data.csv'); print(df.shape)"
```

Expected: prints a non-empty shape (rows, columns).

### 3. Jupyter Notebook

```bash
jupyter notebook
```

Create a new notebook, run:

```python
import sklearn, pandas, numpy
print("Environment ready")
```

### 4. Simple ML Pipeline (Smoke Test)

```python
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification

X, y = make_classification(n_samples=200, n_features=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
model = RandomForestClassifier(random_state=42)
model.fit(X_train, y_train)
print("Accuracy:", model.score(X_test, y_test))
```

Expected: prints an accuracy score without errors.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| `ModuleNotFoundError: No module named 'sklearn'` | Run `pip install scikit-learn` inside your activated venv |
| PyTorch install fails | Use the [official install command](https://pytorch.org/get-started/locally/) for your OS and CUDA version |
| Jupyter kernel error | Run `python3 -m ipykernel install --user` from your venv |
| Dataset file not found | Download from Google Drive; verify the path in your script or notebook |
| `saveValues.py` serial error | Update COM port; close Arduino Serial Monitor before running |
| Arduino upload fails (motor rig) | Double-press RESET for bootloader mode; close Serial Monitor |
| Plot shows time gaps | Normal — UART batch transmission pauses sampling (see Motor Stall README) |
| Out of memory during training | Reduce batch size, use fewer features, or subsample the dataset |

---

## Resources

- [Fault Prediction README](README.md) — challenge description and ML roadmap
- [Motor Stall Test Setup](MotorStallTestSetup/README.md) — hardware rig for live fault data
- [Toyota Telemetry Dataset (Google Drive)](https://drive.google.com/drive/folders/1WH95WIw2kX9aDbsBe2MpxEcpnStesaIB?usp=sharing)
- [scikit-learn Documentation](https://scikit-learn.org/)
- [PyTorch Documentation](https://pytorch.org/)
- [Pandas Documentation](https://pandas.pydata.org/)
- [Kaggle Datasets](https://www.kaggle.com/datasets)
- [UCI ML Repository](https://archive.ics.uci.edu/ml/index.php)
- [Parent Developer Setup Guide](../DEVELOPER_SETUP.md)
