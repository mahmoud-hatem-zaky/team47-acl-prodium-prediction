# Team 47 — F1 Podium Prediction

The shared notebook is
[`notebooks/f1-podium-prediction-milestone-1-team-47-notebook.ipynb`](notebooks/f1-podium-prediction-milestone-1-team-47-notebook.ipynb).
Section 4 contains Member 4's models and experiments. Sections 1–3 belong to the other members;
the source changes there are the agreed `F1_DATA_DIR` override and saving the modeling table
under `data/processed/` instead of the notebook's working directory.

## Local setup with uv

Use Python 3.11 and [uv](https://docs.astral.sh/uv/). The project was initialized with
`uv init --bare --python 3.11 --vcs none`, followed by `uv python pin 3.11` and `uv add`.
uv manages `.venv`; do not create a separate environment with `python -m venv`.

On a fresh clone, initialize the local uv project once from the repository root in PowerShell:

```powershell
uv init --bare --python 3.11 --vcs none
uv python pin 3.11
uv add numpy pandas matplotlib scikit-learn tensorflow
uv add --dev jupyterlab ipykernel nbformat nbclient
New-Item -ItemType Directory -Force -Path .\data\raw
```

If the local uv project is already initialized, use `uv sync --locked` instead of running
`uv init` again. The environment, caches, `pyproject.toml`, `uv.lock`, `.python-version`, and
`uv.toml` are ignored by Git and remain local to each machine.

Copy the **same snapshot used by the team's Kaggle notebook** into `data/raw`:
`circuits.csv`, `constructors.csv`, `constructor_results.csv`, `constructor_standings.csv`,
`drivers.csv`, `driver_standings.csv`, `races.csv`, `results.csv`, `qualifying.csv`,
`lap_times.csv`, `pit_stops.csv`, `sprint_results.csv`, `seasons.csv`, and `status.csv`.
The local copy was obtained from [Kaggle version 24](https://www.kaggle.com/datasets/rohanrao/formula-1-world-championship-1950-2020/versions/24),
whose 14 table shapes match the saved audit. Rebuilding the modeling table also reproduced its
exact fingerprint, `17859401350359469056`, and split sizes, `22459 / 775 / 1352`.
The version-specific [CSV archive](https://www.kaggle.com/api/v1/datasets/download/rohanrao/formula-1-world-championship-1950-2020?datasetVersionNumber=24)
can be used to reproduce that snapshot. Raw CSVs remain untouched and are ignored by Git.

Then start Jupyter using the project environment:

```powershell
$env:F1_DATA_DIR = (Resolve-Path .\data\raw).Path
uv run jupyter lab
```

Open the shared notebook, select the project `.venv` Python kernel, restart the kernel, and Run All.
For VS Code, select `.venv\Scripts\python.exe` as the notebook interpreter and ensure the
Jupyter process inherits `F1_DATA_DIR` (for example, launch VS Code from this PowerShell session).

Runtime dependencies are NumPy, pandas, Matplotlib, scikit-learn, and TensorFlow/Keras.
JupyterLab, ipykernel, nbformat, and nbclient are development dependencies. The shallow network
runs on CPU; no GPU setup is required. Keep `pyproject.toml`, `uv.lock`, and `.python-version`
together locally so `uv sync --locked` can reuse the resolved environment. On another machine,
use the initialization commands above; the notebook records the versions used for its results.

## What Section 4 does

- Uses the existing 15 features and season splits: training through 2019, validation 2020–2021,
  and test 2022–2024. It fits fresh preprocessing on training rows only.
- Shows four attempts in separate cells: logistic regression, an unrestricted random forest,
  a regularized forest, and a one-hidden-layer TensorFlow/Keras network.
- Uses PR-AUC (average precision) as primary and F1 as secondary. Reports ROC-AUC, PR-AUC,
  F1, precision, and recall, including threshold-0.5 versus validation-selected F1.
- Selects the forest configuration, model family, and thresholds on validation only. Evaluates
  the three selected model families on test in one final reporting stage. Reproducing this
  stage during Run All is allowed; changing settings after seeing its scores is not.
- Retrains the selected model after dropping each feature group separately, using validation
  PR-AUC to show the change. These diagnostics do not change the selected full-feature model.

Generated files follow the repository's folders:

- `data/raw/`: the original 14 CSVs.
- `data/processed/model_table.csv`: the frozen modeling table exported by Section 3.
- `report/member4_artifacts/`: comparison/ablation CSVs, PNG figures, package versions, and
  `member4_report.md` with actual measured results for the team's PDF report.

These paths are relative to the repository root whether Jupyter starts in the root or `notebooks/`.
On Kaggle, the same `data/processed/` and `report/member4_artifacts/` folders are created under
`/kaggle/working`. Generated artifacts are ignored by Git;
copy the required figures/tables into the final report. Section 4's saved outputs come from the
matching real F1 data. Separate synthetic checks were kept outside the shared notebook.

The full 101-cell notebook passed a local Run All in a fresh kernel. Validation selected logistic
regression (AP `0.7157`, threshold `0.2462`); its final test AP is `0.7115` and F1 is `0.6654`.
Those settings were frozen before test reporting. Kaggle execution still needs the team's final
integration check.

## Handoff to Member 2

After Section 4, use `final_model`, `final_model_name`, `final_threshold`, and `final_features`.
For any of the three families:

```python
probabilities = final_model.predict_proba(VA[final_features])[:, 1]
prediction = predict_podium(record, final_model, final_threshold)
```

`final_model` contains the fitted preprocessing and classifier. The neural-network classifier
exposes its fitted Keras network as `final_model.named_steps['model'].model_`.
Permutation importance, SHAP, and LIME remain Member 2's section; use training background data
and validation records for explanations. Feature importance describes model reliance, not causation.

## Notes for integration and evaluation

- The existing deliberately leaky preprocessing experiment is a demonstration; Section 4 never
  reuses its fitted objects. The existing Section 3 test-row demo means the test data has been
  held out from model selection, rather than literally never inspected.
- Grid and qualifying groups overlap through fallback and `grid_penalty`. Dropping the grid
  columns does not remove every piece of grid information. Interpret ablations accordingly.
- Rolling features for a later validation/test race may use results of earlier completed races.
  This represents predicting races sequentially; the model itself is not retrained on test.
- The brief defines three metrics but mentions five in its deliverables; precision and recall
  are included to cover that ambiguity.
- Section 1 currently checks only some foreign-key relationships. Its owner should complete
  the full audit required by the brief; Member 4 does not edit those cells.
- Final Kaggle Run All and PDF assembly remain team integration steps. On Kaggle, leave
  `F1_DATA_DIR` unset to use the original dataset path and use a CPU session. The local uv
  environment does not automatically change Kaggle's package versions; Section 4 exports them.
