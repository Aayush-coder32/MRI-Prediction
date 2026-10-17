# NeuroScan AI

NeuroScan AI is a local research and education workstation for brain MRI image classification, visual explanation, and volumetric segmentation experiments. It combines a static HTML/CSS/JavaScript frontend with a FastAPI and PyTorch backend.

> **Research use only.** This project is not a medical device and must not be used for diagnosis, patient management, or treatment decisions. Model output is experimental and requires qualified review.

## What is included

- **2D MRI classification:** Four output labels (`glioma`, `meningioma`, `notumor`, `pituitary`) and optional Grad-CAM visualization.
- **3D NIfTI research workflow:** `.nii` and `.nii.gz` upload, axial/coronal/sagittal slice inspection, tumor-region measurements, evaluation metrics, and 3D surface meshes.
- **Example images:** Five 2D scans in `frontend/assets/samples/`.
- **Volumetric benchmark data:** Six MRI/mask pairs and a manifest in `backend/segmentation/data/`.
- **API:** FastAPI endpoints documented at `/docs` when the server is running.

The frontend contains an introductory landing page. Select **Open the workstation** to enter the existing application interface.

## Requirements

- Windows, macOS, or Linux
- Python 3.11 recommended
- Internet access for installing Python packages
- A GPU is optional; CPU inference is supported but may be slow for 3D volumes

The root `requirements.txt` includes PyTorch, FastAPI, MONAI, Nibabel, scikit-image, and other packages needed by the backend. Installing PyTorch can take time. If pip reports that no compatible PyTorch wheel is available for your Python version, use Python 3.11 and create a fresh environment.

## Run locally

Run these commands from the repository root.

### Windows PowerShell

```powershell
# Check which Python installations are available
py -0p

# Create and activate an environment (Python 3.11 recommended)
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python run_server.py
```

If `py -3.11` is unavailable, install Python 3.11 first. Alternatively, use the installed Python command if it is compatible:

```powershell
python --version
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python run_server.py
```

If PowerShell blocks activation, run this in that PowerShell window and activate again:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

Activation is optional. You can use the virtual environment's Python directly:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe run_server.py
```

### macOS / Linux

```bash
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python run_server.py
```

Then open:

- Workstation: <http://127.0.0.1:8000>
- Interactive API documentation: <http://127.0.0.1:8000/docs>
- 2D backend status: <http://127.0.0.1:8000/api/status>
- Segmentation status: <http://127.0.0.1:8000/api/segmentation/status>

Stop the server with **Ctrl+C** in the terminal.

## Model checkpoints and expected behavior

The backend looks for these optional checkpoint files in `models/`:

| File | Feature | Behavior when absent |
| --- | --- | --- |
| `NeuroScan_AI_model.pth` | 2D classification | The server initializes an untrained model. The endpoint may return output, but it is not a meaningful trained prediction. |
| `NeuroScan_Seg3D_UNet.pth` | 3D segmentation | Segmentation inference is reported unavailable; the app does not fabricate a prediction. |

Neither checkpoint is included in this repository checkout. Do not treat demo output as clinically valid. The 2D weights must match the `BrainTumorCNN` architecture and expected checkpoint structure in `backend/model.py`. The 3D model loader and architecture are in `backend/segmentation/model.py`.

The benchmark MRI and ground-truth files are present in `backend/segmentation/data/`; they are not trained model weights. Uploading or selecting a benchmark case does not make segmentation inference available without a valid 3D checkpoint.

## Project layout

```text
backend/
  main.py                  FastAPI application and 2D endpoints
  model.py                 2D CNN architecture and checkpoint loader
  predict.py               Image preprocessing, prediction, and Grad-CAM
  segmentation/            3D model, NIfTI pipeline, API, metrics, and training code
  segmentation/data/       Six benchmark MRI/mask pairs and manifest
frontend/
  index.html               Landing page and workstation UI
  style.css                Workstation and landing page styles
  script.js                UI interactions and API integration
  assets/samples/          Five 2D example images
models/                    Optional model checkpoint location
scripts/                   Dataset and training helper scripts
tests/                     Pytest test suite
run_server.py              Local Uvicorn launcher
requirements.txt           Backend dependencies
DATA_PROVENANCE.md          Dataset provenance notes
```

## API overview

### 2D classification

- `GET /api/status` — backend/device and 2D model status
- `POST /predict` — accepts an image file and returns class probabilities and Grad-CAM data when available

### 3D segmentation

- `GET /api/segmentation/status` — checkpoint and pipeline status
- `GET /api/segmentation/metadata` — labels and pipeline metadata
- `GET /api/segmentation/samples` — available benchmark cases
- `GET /api/segmentation/slice?case_key=case_001&plane=axial&slice_idx=71` — benchmark slice preview
- `POST /api/segmentation/predict` — NIfTI upload or benchmark-case analysis; requires a valid segmentation checkpoint for prediction

Open `/docs` for request schemas and interactive API calls.

## Data and helper scripts

The included dataset manifest is `backend/segmentation/data/dataset_manifest.json`. The detailed attribution and provenance notes are in [DATA_PROVENANCE.md](DATA_PROVENANCE.md). Do not assume that the repository's demo dataset or model outputs are suitable for clinical use.

Available helper scripts include:

- `scripts/check_dataset.py` — inspect dataset files
- `scripts/validate_and_manifest.py` — validate data and update manifest information
- `scripts/prepare_samples.py` — prepare sample data
- `scripts/download_real_brats.py` and `scripts/download_additional_cases.py` — dataset download helpers; review their source and upstream dataset terms before using them
- `scripts/train_baseline.py` — baseline training helper

The training module can also be invoked with `python -m backend.segmentation.train --help`. Training requires appropriate labeled data, compute, and configuration; the small included benchmark set should not be represented as independent clinical validation.

## Tests

Install dependencies first, then run from the repository root:

```bash
python -m pytest -v
```

## Troubleshooting

### `No suitable Python runtime found`

The Windows Python launcher could not find the requested version. Check installed interpreters with `py -0p`, install Python 3.11, then recreate the environment with `py -3.11 -m venv .venv`.

### `Activate.ps1` is not recognized

The virtual environment was not created, or the terminal is in a different directory. From the project root, run `python -m venv .venv` and confirm `.venv\Scripts\Activate.ps1` exists. You can skip activation by calling `.venv\Scripts\python.exe` directly.

### Missing modules or package installation errors

Make sure the virtual environment is active and use `python -m pip install -r requirements.txt`. For PyTorch wheel compatibility errors, switch to Python 3.11 and create a new virtual environment.

### Server starts but predictions are not useful

Check that the required checkpoint exists in `models/`. With no 2D weights, predictions come from an untrained model. With no 3D weights, segmentation inference is unavailable.

## License

See [LICENSE](LICENSE) for the repository license. Dataset and third-party component terms may differ; consult their original sources and [DATA_PROVENANCE.md](DATA_PROVENANCE.md).
