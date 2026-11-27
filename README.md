# CARDIOGUARD AI

A locally running ECG analysis project. It displays real MIT-BIH signals, applies filtering, detects peaks, and estimates BPM and RR intervals. This project is not intended for clinical diagnosis.

## Getting Started

Open two terminals. In both terminals, start from the project root directory.

### 1. Backend

```bash
cd backend
```

For Linux/macOS:

```bash
.venv/bin/python -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

For Windows:

```bash
python -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

### 2. Frontend

Open a second terminal:

```bash
cd frontend
npm run dev
```

Open the application in your browser:

[http://127.0.0.1:5173](http://127.0.0.1:5173)

In the **ECG Analyzer** section, select a channel and time interval, then click **Run analysis**. You can select **Normalization → Z-score** to normalize the signal.

To view the signal, BPM/RR measurements, and AI results together, click **MLII → Analyze with AI**. This button applies the model's predefined filtering and peak-detection settings. The regular **Run analysis** button uses your selected settings.

The model version and evaluation results are available in **AI Lab**.

## Initial Setup

This step is only required if dependencies or the dataset are missing. The project has been tested with Node.js 24 and Python 3.13.

### Backend

From the `backend` directory:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements-dev.txt
.venv/bin/python -m scripts.download_mitbih --annotations
```

### Frontend

From the `frontend` directory:

```bash
npm ci
```

On Windows, use `.venv\Scripts\python.exe` instead of `.venv/bin/python`.

## Project Structure

| Directory | Purpose |
|---|---|
| `backend/app/` | API, dataset loading, and ECG calculations |
| `backend/tests/` | Backend tests |
| `backend/scripts/` | Dataset download and result validation scripts |
| `frontend/src/` | Pages, buttons, and charts |
| `data/mitbih/` | Local ECG files |
| `docs/` | Short code guide; older reports are stored in `archive/` |
| `.cache/` | Automatically generated cache and test results |

Code documentation: [Quick Guide](https://github.com/aruuke2006/cardioguard-ai/blob/main/docs/development.md).

Dataset source: [MIT-BIH / PhysioNet](https://github.com/aruuke2006/cardioguard-ai/blob/main/data/README.md).

## Running Tests

### Backend

From the `backend` directory:

```bash
.venv/bin/python -m pytest
```

### Frontend

From the `frontend` directory:

```bash
npm test
npm run build
```

For additional browser tests:

```bash
npm run test:e2e:install
npm run test:e2e
```

## Features

- ECG records 100, 106, 119, 200, and 208; a maximum of 30 seconds per request.
- Zoom, pan, hover, and reset controls.
- Signal filtering, a separate Z-score-normalized signal, peak detection, and BPM/RR tables.
- Light and dark themes, with a resizable sidebar.

AI Lab displays actual N/PVC model predictions. The AI model uses MLII signals sampled at 360 Hz.

The **3D Heart Explorer** uses a local anatomical heart model.

**Algorithms Lab** demonstrates 9 data structures and 7 algorithms using actual Python execution steps.

The build process displays bundle-size warnings for Plotly and Three.js, but the build completes successfully.

## Training the AI Model

Run the following commands once from the `backend` directory:

```bash
.venv/bin/python -m pip install -r requirements-dev.txt
.venv/bin/python -m scripts.download_mitbih --ml
.venv/bin/python -m scripts.train_model
```

Then open **AI Lab**, select a record, and click **Classify beats**.

The model is not trained automatically when the backend starts. Existing dataset files are not overwritten when using the `--ml` option.

### Dataset Splits and Model Details

- Training records: 100, 106, and 200.
- Validation record: 119.
- Test record: 208.

Beats from the same individual are not mixed across the training, validation, and test splits. Only N and V annotations are used.

The model, metadata, and prepared dataset are stored locally in `data/models/`.

Detailed results: [Sprint 3](https://github.com/aruuke2006/cardioguard-ai/blob/main/docs/sprint-3.md).

MVP integration and final checks are documented in the [Sprint 4 report](https://github.com/aruuke2006/cardioguard-ai/blob/main/docs/sprint-4.md).

## 3D Heart Explorer

Open **3D Heart Explorer** from the menu.

- Drag with the mouse to rotate the model.
- Use the scroll wheel or pinch gestures to zoom.
- Right-click and drag, or use two fingers, to pan.
- Click **Reset view** to restore the initial view.
- When the model is focused, use the arrow keys to rotate it, `+` / `−` to zoom, and `Home` to reset the view.

Select the camera, blood vessels, and internal structures in the model or the list to read about their functions.

Hovering over an element displays a short description. Click **View inside** to explore internal structures.

The HRA Heart and Blood Vasculature Female v1.3 models, source maps, and CC BY 4.0 license are located in `frontend/public/models/heart/`.

The model does not require additional downloads or API keys. If WebGL is unavailable, the informational guide remains accessible.

Sources and validation results: [Sprint 5 report](https://github.com/aruuke2006/cardioguard-ai/blob/main/docs/sprint-5.md).

## Algorithms Lab

Open **Algorithms Lab** from the menu.

Select a category and algorithm, modify the input data, and click **Run**.

Use **Pause**, **Step Forward**, and **Reset** to follow the execution steps. The playback speed ranges from 0.5x to 4x.

In data structure demonstrations, you can add operations to the queue using **Add operation**.

For Bubble Sort and Merge Sort, the **Compare** option displays both algorithms side by side using the same input.

Binary Search requires sorted input. Dijkstra's algorithm does not support negative edge weights.

This laboratory is designed for educational purposes. It does not require additional dependencies, datasets, or API keys.

API, files, limits, and validation details: [Sprint 6 report](https://github.com/aruuke2006/cardioguard-ai/blob/main/docs/sprint-6.md).
