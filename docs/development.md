# Quick Code Guide

For everyday development, starting the two servers described in the README is sufficient.

## Backend: Key Files to Review First

| File | Purpose |
|---|---|
| `backend/app/main.py` | Starts FastAPI and prepares error responses |
| `backend/app/routes.py` | Contains the existing 7 ECG/AI/health API endpoints |
| `backend/app/algorithms/` | Contains the laboratory's data structures, algorithms, metadata, and 2 API routes |
| `backend/app/schemas.py` | Defines the data fields sent to and received from the API |
| `backend/app/dataset.py` | Reads local MIT-BIH files |
| `backend/app/processing.py` | Handles filtering, peak detection, normalization, and BPM/RR calculations |
| `backend/app/analysis.py` | Combines these steps and returns the analysis response |
| `backend/app/ml.py` | Contains shared heartbeat features and predictions from the saved model |

The basic workflow is:

**Request → routes → dataset/analysis → response**

### Main Functions in `processing.py`

- `preprocess`: Filters the signal.
- `detect_peaks`: Detects candidate QRS peaks.
- `normalize`: Calculates Z-score normalization.
- `rr_statistics`: Calculates RR intervals and BPM from the distances between peaks.
- `compare_peaks`: Compares detected peaks with reference annotations.

## Frontend: Main Screen Files

| File | Purpose |
|---|---|
| `frontend/src/App.tsx` | Determines which page should be displayed |
| `frontend/src/features/ecg/ECGAnalyzer.tsx` | Provides record, channel, and interval selection |
| `frontend/src/features/ecg/SignalAnalysis.tsx` | Contains the analysis button, settings, and results |
| `frontend/src/features/ecg/ECGChart.tsx` | Displays the signal and detected peaks on a chart |
| `frontend/src/hooks/useApiResource.ts` | Handles API requests, loading states, errors, and retries |
| `frontend/src/styles.css` | Defines the overall appearance |

## API Endpoints

- `GET /api/v1/health`
- `GET /api/v1/ecg/records`
- `GET /api/v1/ecg/records/100`
- `GET /api/v1/ecg/records/100/signal`
- `POST /api/v1/ecg/analyze`
- `GET /api/v1/ai/model` — Returns the model's availability status and actual version.
- `POST /api/v1/ai/predict` — Returns N/PVC model predictions for the selected interval.
- `GET /api/v1/algorithms` — Returns 16 options, explanations, pseudocode, and working examples.
- `POST /api/v1/algorithms/execute` — Generates actual execution steps and results from the supplied input.

### Combined Analysis Request

The following is an example of a combined analysis request for `POST /api/v1/ecg/analyze`:

```json
{
  "record_id": "208",
  "channel": 0,
  "start_seconds": 0.55,
  "duration_seconds": 10,
  "include_ai": true
}
```

The `include_ai` parameter defaults to `false`.

When set to `true`, the same filtered signal and detected peaks are used for both BPM/RR calculations and `ai_classification`.

AI analysis requires MLII at 360 Hz, an order-3 filter with a 0.5–40 Hz frequency range, and the default detection settings. Other settings return a 422 response.

Z-score normalization can be displayed separately on the chart.

### Error Handling and Limits

- Unknown records return 404.
- Invalid input returns 422.
- Missing datasets or models return 503.
- Unexpected errors return 500 with a `detail` message; technical information is written to the backend log.

When the model is unavailable, the model status endpoint returns HTTP 200 with `available=false`.

Each signal request is limited to a maximum of 30 seconds. The analysis interval must be between 0.5 and 30 seconds.

Frontend requests are cancelled after 15 seconds and display a retry option.

The complete machine-readable API specification is available locally at `/openapi.json` on the running backend.

### Signal Units and Interpretation

Both the original and filtered signals remain in mV.

The Z-score signal is displayed separately on the right axis without physical units.

Peak detection and BPM calculations are based on the filtered signal.

The results are not medical diagnoses.

## AI Model Implementation

The AI training script is located at:

`backend/scripts/train_model.py`

The AI Lab interface is located at:

`frontend/src/features/ai/AILab.tsx`

Feature extraction is shared between training and prediction.

Annotations are used only for training and evaluation; the prediction process does not read them.

The training process and actual model evaluation metrics are documented in [Sprint 3](https://github.com/aruuke2006/cardioguard-ai/blob/main/docs/sprint-3.md).

## 3D Heart Explorer

`App.tsx` lazy-loads the Heart Explorer page when it is opened from the menu.

The 3D functionality uses Three.js directly, together with its `GLTFLoader` and `OrbitControls` modules.

| File | Purpose |
|---|---|
| `frontend/src/features/heart/HeartExplorer.tsx` | Page layout, structure selection, layers, and anatomical explanations |
| `frontend/src/features/heart/HeartViewer.tsx` | Loading, error handling, retry functionality, and control buttons |
| `frontend/src/features/heart/heart-scene.ts` | GLB loading, lighting, camera, picking, and resource disposal |
| `frontend/src/features/heart/anatomy.ts` | Source mapping and explanations for the camera, blood vessels, and internal structures |
| `frontend/public/models/heart/` | Local models, crosswalk, source hashes, and license |

Structure selection is based on the source mapping in `anatomy.ts`.

The camera, blood vessels, and internal structures can be selected. Hovering displays brief information, while the layer controls reveal the internal view.

Rendering occurs only when the view, selection, or dimensions change.

When the user leaves the page, active fetch requests are cancelled, event listeners and observers are removed, and WebGL resources are released.

Retry creates a new viewer instance.

The model sources, coverage of the available geometry, and tests are documented in the [Sprint 5 report](https://github.com/aruuke2006/cardioguard-ai/blob/main/docs/sprint-5.md).

## Algorithms Lab

The execution workflow is:

**Input → Pydantic validation → Python execution → execution steps → React playback**

The backend stores a separate copy of each execution step.

The frontend displays these steps. Pause and Reset do not start a new calculation.

When the input changes, clicking Run again sends a new request.

| File | Purpose |
|---|---|
| `backend/app/algorithms/structures.py` | Original implementations of 9 data structures |
| `backend/app/algorithms/execution.py` | 7 algorithms, data structure operations, execution steps, and results |
| `backend/app/algorithms/catalog.py` | Explanations, complexity information, operation forms, and examples |
| `backend/app/algorithms/schemas.py`, `routes.py` | Pydantic contracts and 2 API endpoints |
| `frontend/src/features/algorithms/AlgorithmsLab.tsx` | Selection, input forms, shared controls, and execution step display |
| `frontend/src/features/algorithms/AlgorithmVisualization.tsx` | Bar, graph, tree, and data structure visualizations |
| `frontend/src/features/algorithms/algorithms.css` | Styling consistent with the existing design |
| `frontend/src/lib/algorithms.ts` | TypeScript request and response types |

Invalid formats and incompatible input return 422.

Operation errors, such as attempting to remove an element from an empty data structure, are reported in the `steps[].error` and `result.errors` fields of a 200 response. Subsequent valid operations continue to execute.

User-supplied code is not executed.

Limits and API examples are documented in the [Sprint 6 report](https://github.com/aruuke2006/cardioguard-ai/blob/main/docs/sprint-6.md).

## Tests and Archived Documentation

Python tests are located in `backend/tests/`.

Frontend component tests use the `*.test.tsx` naming pattern.

Browser tests are located in `frontend/e2e/`.

Run the relevant tests after making changes.

Browser tests use separate ports 18000 and 15173. Their results are written to `.cache/browser-tests` and `.cache/browser-report`.

Previous sprint and audit reports are stored in the [archive/](https://github.com/aruuke2006/cardioguard-ai/blob/main/docs/archive) directory.

File paths, CI information, and coverage data in those archived reports reflect the project state at the time they were written.

In the current simplified version, there is no separate CI workflow, coverage threshold, or Python lock file.
