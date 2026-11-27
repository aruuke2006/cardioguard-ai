## Heart Model Assets and Anatomical Mapping

| **File** | **Information** |
|---|---|
| `frontend/public/models/heart/heart.glb` | 1,745,560 bytes, 14 meshes |
| `frontend/public/models/heart/crosswalk.csv` | Mapping of source nodes to ontology terms |
| `frontend/public/models/heart/source.json` | URLs, authors, license, and SHA-256 |
| `frontend/public/models/heart/LICENSE.txt` | License information |

The GLB SHA-256 hash is `98f66e46b149bfc3836592d9404c50290e21d302ec55c98813435e2d9847b1d9`.

Selection validation checks both the source node name and the UBERON ID stored in the GLB extras.

| **Displayed Name** | **Source Node** | **Ontology ID** |
|---|---|---|
| Right atrium | `VH_F_right_cardiac_atrium` | `UBERON:0002078` |
| Right ventricle | `VH_F_right_ventricle` | `UBERON:0002080` |
| Left atrium | `VH_F_left_cardiac_atrium` | `UBERON:0002079` |
| Left ventricle | `VH_F_left_ventricle` | `UBERON:0002084` |

This model does not contain separate aorta or pulmonary artery meshes. These structures are explained in the educational guide. The selectable highlights cover the walls of the four heart chambers. The model also preserves the geometry of the valves, septum, and papillary muscles.

The educational texts are concise adaptations based on NIH/NHLBI explanations of heart anatomy and blood circulation:

- [Heart Anatomy](https://www.nhlbi.nih.gov/health/heart/anatomy)
- [How the Heart Works: Blood Flow](https://www.nhlbi.nih.gov/health/heart/blood-flow)

## 4. Created and Modified Files

| **File** | **Purpose** |
|---|---|
| New `frontend/src/features/heart/HeartExplorer.tsx` | Page and educational panel |
| New `frontend/src/features/heart/HeartViewer.tsx` | Viewer lifecycle, loading, retry, and toolbar |
| New `frontend/src/features/heart/heart-scene.ts` | Camera, lighting, model, picking, and cleanup |
| New `frontend/src/features/heart/anatomy.ts` | Verified anatomical mapping and educational texts |
| New `frontend/src/features/heart/heart.css` | Responsive styling using the existing color palette |
| New `HeartExplorer.test.tsx`, `heart-model.test.tsx` | UI, source mapping, hash, and disposal tests |
| New `frontend/e2e/heart.spec.ts` | Real WebGL and integration tests |
| New `frontend/public/models/heart/` | Model and provenance files |
| `frontend/src/App.tsx` | Lazy integration of Heart Explorer into the existing hash-based routing |
| `frontend/src/components/Icon.tsx` | Icons for zoom controls |
| `frontend/src/features/overview/Overview.tsx` | Available status for Heart Explorer and Phase 2 indicator |
| `frontend/e2e/ecg.spec.ts` | Stabilization of a StrictMode request race condition in an error scenario |
| `frontend/package.json`, `frontend/package-lock.json` | Three.js and type dependencies |
| `README.md`, `docs/development.md`, this report | Usage and code documentation |

## 5. UI Integration

The route is `#/heart-explorer`. On desktop screens, the viewer is positioned on the left, while the selection controls and explanations are displayed on the right. On narrow screens, the panels are stacked vertically. The blood circulation guide can be expanded below.

The page uses the existing design system, including its headings, borders, spacing, Inter font, and theme.

When the model is focused, the arrow keys rotate it, `+` / `−` control zoom, and `Home` resets the view. Mouse controls include left-drag to rotate, right-drag to pan, and scrolling to zoom. Touch controls support one-finger rotation and two-finger zoom/pan gestures.

The selection list and toolbar use native buttons and support keyboard interaction. Educational text remains accessible if a WebGL error occurs.

## 6. Verification Results

| **Command** | **Result** |
|---|---|
| `backend/` — `.venv/bin/python -m pytest -q` | **154 passed**, no skipped tests |
| `frontend/` — `npm test` | **49 passed**, 11 test files |
| `frontend/` — `npm run build` | TypeScript checks and production build passed |
| `frontend/` — `npm run test:e2e -- e2e/heart.spec.ts` | **15 passed** |
| `frontend/` — `npm run test:e2e` | **39 passed**, all ECG/AI/3D scenarios |
| Project root — `git diff --check` | Passed |

A total of **242 automated tests** passed. Browser tests were executed in Chromium at desktop (1440×1000), tablet (1024×1366), and mobile (393×852) viewport sizes.

The following were verified: actual GLB rendering, view transformations, restoration of the original view after reset, mouse panning and scrolling, keyboard controls, one-finger rotation, and two-finger pinch gestures. Highlighting of all four heart chambers and picking directly on the model were also tested.

When leaving the page, the previous canvas was removed from the DOM and its WebGL context was closed. During an idle-state check, the `requestAnimationFrame` counter did not increase over a 350 ms interval.

A 404 error and WebGL context loss were simulated. Recovery of the actual model through the retry mechanism was verified. WebGL support was also disabled to test the fallback screen, educational panel, and navigation to the ECG page. Screenshots were used to review both light and dark themes, as well as responsive layouts.

During the first full regression run, an existing ECG error test encountered an issue: the first StrictMode request, which was cancelled, consumed a one-time mock response. The error response was retained until the retry step, stabilizing the scenario. The final full test run passed all 39 tests.

No separate lint command is configured. The build process also performs TypeScript checks.

## 7. Remaining Limitations

- Selectable geometry is limited to the walls of the four heart chambers. Explanations of the blood vessels are provided separately in the educational guide.
- WebGL 2 support is required. Unsupported devices display an error screen and the educational guide.
- The lazy-loaded Heart Explorer chunk is approximately **653 KB**, or **163 KB** after gzip compression. The Plotly chunk also continues to trigger the existing build-size warning.
- Browser verification was performed using Chromium.

## 8. Completion Decision

**Sprint 5 COMPLETE.** The licensed anatomical asset is rendered locally. Rotation, zoom, pan, reset, verified chamber selection, educational content, responsive layouts, and error recovery have been tested. Phase 1 regression tests passed.

## 9. Sprint 6 — Algorithms Lab

1. Use the existing functions from `processing.py` to demonstrate filtering and R-peak detection step by step.
2. Compare the input and output signals at each stage and demonstrate the effects of parameter changes.
3. Explain BPM/RR calculations using the selected peaks.
4. Prepare UI tests and numerical consistency tests using short real ECG intervals.
