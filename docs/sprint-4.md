# Phase 1 — Sprint 4

**Final verification date: November 17, 2026**

## A. Sprint Status

| Sprint | Status | Verified Results |
|---|---|---|
| 1 | COMPLETE | Local record selection, real ECG signals, zoom, pan, hover, and reset |
| 2 | COMPLETE | Filtering, Z-score normalization, peak detection, BPM/RR calculations, and annotation comparison |
| 3 | COMPLETE | Real N/PVC training, saved model, inference API, and AI Lab |
| 4 | COMPLETE | Combined analysis, error responses, and desktop/tablet/mobile testing |

## B. Completed Changes

- The **Analyze with AI** feature in ECG Analyzer passes the same signal and detected peaks to both BPM/RR calculations and the model. N/PVC counts, probabilities, and chart markers are displayed together.
- The existing analysis endpoint was extended with `include_ai`. The AI response does not resend the signal array. The separate AI prediction endpoint also works.
- The AI button applies the model's 0.5–40 Hz order-3 filter and default peak-detection settings for MLII/360 Hz. These settings are displayed in the interface. Regular analysis accepts custom settings.
- Metadata from the record list is reused. The number of API requests required for the initial ECG loading was reduced from three to two.
- The model is stored in a cache that is revalidated when the model version or file changes. Loading is protected against concurrent initial requests. Corrupted or modified files return 503.
- Inference calculates `predict_proba` only once. Response dimensions, finite values, and probability sums are validated. Peak refinement no longer recalculates the median of the entire interval for every peak.
- The chart is not re-rendered for every form change. AI markers are displayed as filled circles/diamonds, while detector markers use outlines.
- Unexpected backend errors produce a JSON `detail` response and a server log entry. OpenAPI error responses are documented. Dataset recovery messages identify the selected record.
- A warning was added for peaks that do not match reference annotations. The model version, two-class limitation, and uncalibrated probabilities are displayed.

### Sprint 3 Re-verification

The dataset was rebuilt using the `.hea`, `.dat`, and `.atr` files of five real MIT-BIH records. A RandomForest model was trained separately in a temporary directory.

The current model identifier was `603d5a015e2f5cc4`, while the verification training identifier was `6ea0ce5de8fe37c0`.

Source hashes, selected samples, and evaluation results matched.

| Split | Records | Number of Samples |
|---|---|---:|
| Train | 100, 106, 200 | 6706 |
| Validation | 119 | 1949 |
| Test | 208 | 2528 |

Patient groups are separated across the splits. Validation and test data are not included in training.

Held-out test results:

- Accuracy: **99.2089%**
- Macro F1: **99.1677%**
- Confusion matrix: `[[1535,19],[1,973]]`

These metrics apply to classification at the centers of selected N/V annotations. Detector errors are not included in these metrics.

## C. Modified Files

| Directory | Files and Purpose |
|---|---|
| `backend/app/` | `analysis.py`, `ml.py`, `processing.py` — shared analysis and caching |
| `backend/app/` | `routes.py`, `schemas.py`, `main.py`, `dataset.py` — API contracts and error handling |
| `backend/scripts/` | `train_model.py` — removes repeated median calculations |
| `backend/tests/` | `test_ml.py`, new `test_pipeline.py` — integration, cache, and error checks |
| `frontend/src/features/ecg/` | `ECGAnalyzer.tsx`, `SignalAnalysis.tsx`, `ECGChart.tsx`, `ecg.css`, and three related test files |
| `frontend/src/features/ai/` | `AILab.tsx`, new `ClassificationResults.tsx` — shared results display |
| `frontend/src/lib/` | `analysis.ts`, `ai.ts` — API types |
| `frontend/e2e/` | `ecg.spec.ts`, new `pipeline.spec.ts` — regression testing and complete analysis workflow |
| `frontend/` | `playwright.config.ts` — tablet viewport configuration |
| Documentation | `README.md`, `docs/development.md`, and this report |

## D. Completed Verification

| Directory and Command | Actual Result |
|---|---|
| `backend/` — `.venv/bin/python -m pytest -q` | **154 passed**, no skipped tests |
| `frontend/` — `npm test` | **44 passed**, 9 test files |
| `frontend/` — `npm run build` | TypeScript checks and production build passed |
| `frontend/` — `npm run test:e2e` | **24 passed**, Chromium desktop/tablet/mobile |
| `backend/` — `.venv/bin/python -m pip check` | No dependency conflicts |
| `backend/` — `.venv/bin/python -m scripts.evaluate_ecg --full-record` | Record 100, 122 intervals across two channels |
| Project root — `git diff --check` | No whitespace errors |

A total of **222 automated tests** passed.

Browser testing covered the following viewport sizes:

- 1440 × 1000
- 1024 × 1366
- 393 × 852

The combined results were also tested at a tablet viewport of 820 × 1180.

The tests covered real signals, BPM/RR, model version, N/PVC counts, marker coordinates, probability tables, interval changes, navigation, theme, and sidebar behavior.

Missing model/dataset scenarios and API errors were tested using controlled browser responses. Retry functionality was verified using actual backend results.

Backend tests also exercised the complete API workflow for five real records.

Additional tests verified that regular analysis works when the model is unavailable, as well as invalid parameters, corrupted artifacts, concurrent cache loading, and unexpected 500 responses.

Full-record comparison for record 100 matched previously saved results:

- Channel 0: TP/FP/FN = **2261/0/12**
- Channel 1: TP/FP/FN = **2253/0/20**

### Model Training Verification

The training verification used `.venv/bin/python` to execute the following calls:

```python
from pathlib import Path
from tempfile import TemporaryDirectory
from app.ml import load_model
from scripts.train_model import train

_, current = load_model()
with TemporaryDirectory(dir="../.cache", prefix="sprint-4-model-") as folder:
    result = train(directory=Path(folder))
    model, restored = load_model(Path(folder))
    assert current["records"] == result["records"]
    assert current["evaluation"] == result["evaluation"]
```

### Performance Measurements

In local measurements:

- Median model loading time without cache: **12.258 ms**, across 30 repetitions.
- Median model loading time with cache: **0.053 ms**, across 100 repetitions.
- Combined 30-second TestClient requests for five records: **14.389–21.201 ms**.

These are process-level measurements obtained on the local machine.

The build produced a bundle-size warning for the Plotly chunk.

No separate lint command is configured. Browser verification was performed using Chromium.

## E. Remaining Limitations

| Severity | Status |
|---|---|
| Critical/high technical blockers | None identified in the verified local MVP workflow |
| Medium | The detector misses peaks in some morphologies. In the first 30 seconds of record 208, 35 of 52 annotated beats were matched, while 17 were missed. This affects BPM/RR and AI counts. The interface reports this mismatch. |
| Medium | The model is limited to five records and N/V classes. Other rhythms may be assigned to one of these two classes. Probabilities are not calibrated. |
| Low | The lazily loaded Plotly chunk is approximately 1.17 MB, or 388 KB when gzipped. |

This detector result is retained as a measured limitation of the research MVP.

The product displays estimated algorithmic results. Broader accuracy claims require additional detector and classifier evaluation across diverse records.

## F. Phase 1 Decision

**COMPLETE — as a local educational and research MVP. Ready to proceed to Phase 2 development.**

The following workflow operates with real data:

Dataset → Filtering → Peak Detection → BPM/RR → Classification → API → Dashboard

Sample/time alignment, model retraining, error handling, and previous features have been verified through testing.

The accuracy limitations described above define the scope of this completion decision.

## G. Next Steps for Sprint 5

1. Select a locally stored anatomical 3D heart asset with a clearly defined license.
2. Add rotation, zoom, and reset functionality to the existing Heart Explorer section.
3. Implement heart structure selection, names, and short educational explanations.
4. Test model loading, WebGL errors, tablet controls, and rendering performance.

Usage instructions and API request examples are available in the [Quick Guide](https://github.com/aruuke2006/cardioguard-ai/blob/main/docs/development.md).
