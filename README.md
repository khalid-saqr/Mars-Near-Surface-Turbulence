# Mars Near-Surface Turbulence

Companion repository for the manuscript:

**Evidence for non-Universal Reduced Closures and Sol-Dependent State-Space Relations in Mars Near-Surface Turbulence**

This repository contains the code and bundled assets used to reproduce a cross-mission analysis of near-surface Martian turbulence using **InSight** and **Perseverance/MEDA** data. The workflows cover data acquisition from NASA PDS, harmonization into common state vectors, regression and spectral diagnostics, dimensionless-regime analysis, and generation of report figures.

Repository URL: `https://github.com/khalid-saqr/Mars-Near-Surface-Turbulence`

---

## What this repository reproduces

The paper compares harmonized high-frequency atmospheric intervals from two Mars surface missions:

- **InSight**: sols **0101**, **0400**, **0500**
- **Perseverance / MEDA**: sols **0044**, **0100**, **0315**
- Common analysis window per sol: **4410 s** (**73.5 min**)

The comparative analysis focuses on the atmospheric triplet:

- wind speed
- air temperature
- pressure

For MEDA, a surface/ground-temperature channel is also ingested during preprocessing.

Main diagnostics reproduced by this repository:

- wind power spectral density and fitted spectral slopes
- reduced momentum surrogates using `dv/dt` versus `v²` and `v`
- acoustic-style scaling using `v` versus `sqrt(T)`
- pressure-temperature state-space fits
- dimensionless diagnostics:
  - Mach number
  - Knudsen number
  - Reynolds number
  - Prandtl number
  - Strouhal number
  - Eckert number

---

## Repository contents

Current top-level files visible in the repository:

| Path | Purpose |
|---|---|
| `README.md` | Project overview and reproduction instructions |
| `TheMartianTheory_InSight.zip` | Archived/bundled InSight workflow assets |
| `TheMartianTheory_MEDA.ipynb` | MEDA / Perseverance notebook workflow |
| `TheMartianTheory_MEDA.zip` | Archived/bundled MEDA workflow assets |

### Runtime-generated structure

When the notebooks are run, they create and populate a working structure similar to:

```text
data/
├── raw/
│   ├── pds_discovery_registry.csv
│   ├── multi_sol_master_manifest.csv
│   ├── meda_pds_discovery_registry.csv
│   └── sol_XXXX/...
├── processed/
│   ├── sol_0101/sol0101_state_hf.npz
│   ├── sol_0400/sol0400_state_hf.npz
│   ├── sol_0500/sol0500_state_hf.npz
│   ├── sol_0044/sol0044_state_hf.npz
│   ├── sol_0100/sol0100_state_hf.npz
│   ├── sol_0315/sol0315_state_hf.npz
│   └── meda_processed_state_manifest.csv
└── reports/
    ├── Nature_Physics_Fig1.pdf
    └── Nature_Physics_Dimensionless_Final.pdf
```

The exact filenames of report products may evolve slightly as the notebooks evolve, but the workflow is designed to write processed `.npz` state files, CSV manifests, and publication-style figure PDFs.

---

## Data sources

The workflows are built around calibrated NASA PDS4 atmospheric products.

### InSight sources

- Calibrated TWINS collection  
  `https://atmos.nmsu.edu/PDS/data/PDS4/InSight/twins_bundle/data_calibrated/`
- Calibrated pressure products from the InSight APSS archive
- Relevant PDS bundle references listed in the manuscript:
  - `10.17189/1518950`
  - `10.17189/1518939`
  - `10.17189/jb1w-7965`

### Perseverance / MEDA sources

- Calibrated MEDA environmental collection  
  `https://atmos.nmsu.edu/PDS/data/PDS4/Mars2020/mars2020_meda/data_calibrated_env/`
- Relevant PDS bundle reference listed in the manuscript:
  - `10.17189/1522849`

---

## Recommended execution environment

## 1) Google Colab (recommended)

The notebooks were authored in a Colab-style workflow and expect Google Drive mounting in the setup cell. This is the easiest way to reproduce the results with minimal path editing.

### Why Colab is recommended

- the notebooks already mount Google Drive
- package installation is performed inline
- output paths are written under `/content/drive/MyDrive/...`
- the workflows are designed as ordered notebook pipelines

## 2) Local Jupyter / JupyterLab

Running locally is possible, but you must adapt the Colab-specific setup:

- remove or replace:
  - `from google.colab import drive`
  - `drive.mount('/content/drive')`
- replace `/content/drive/MyDrive/...` paths with a local project directory
- preserve the relative folder structure:
  - `data/raw`
  - `data/processed`
  - `reports`

### Core Python packages

At minimum, install:

```bash
pip install numpy pandas matplotlib scipy requests beautifulsoup4 pydmd pysr deeptime MFDFA jupyter
```

Notes:

- `PySR` may require additional local setup depending on platform.
- If you want the closest reproduction path, use Colab first.

---

## Quick start

### A. Reproduce from the repository as distributed

1. Clone or download this repository.
2. Extract any bundled ZIP archives you intend to use.
3. Open the notebook(s) in **Google Colab** or **JupyterLab**.
4. Run cells **in order**, from top to bottom.
5. Allow the notebooks to:
   - install dependencies
   - crawl/download the required PDS files
   - crop the selected sol windows
   - harmonize the state vectors
   - write processed `.npz` files
   - generate figures and summary outputs

### B. Minimal command-line start

```bash
git clone https://github.com/khalid-saqr/Mars-Near-Surface-Turbulence.git
cd Mars-Near-Surface-Turbulence
```

Then open the relevant notebook in Colab or Jupyter.

---

## Important reproduction notes

> [!IMPORTANT]
> These notebooks are **stateful sequential pipelines**. Do not jump directly to later analysis cells on a clean run. Later cells depend on manifests and `.npz` state files created by earlier cells.

> [!IMPORTANT]
> If you run locally instead of in Colab, the only required conceptual change is path handling. The scientific workflow should remain the same.

> [!IMPORTANT]
> If you already have the source PDS files downloaded, you can reuse them, but keep the notebook’s expected folder structure intact so the discovery/manifest logic continues to work.

---

## Notebook workflow summary

## InSight workflow

The InSight workflow:

1. defines the target sols:
   - `0101`
   - `0400`
   - `0500`
2. discovers and downloads the required calibrated atmospheric files from the InSight PDS archive
3. crops each record to the common **4410 s** window
4. synchronizes the atmospheric variables onto a common master grid
5. builds a processed state matrix and saves:
   - `time`
   - `X_state`
6. generates publication-style comparison figures and summary diagnostics

### InSight harmonization logic

The paper and notebook workflow use:

- common atmospheric variables:
  - wind speed
  - air temperature
  - pressure
- a nominal target frequency of about **2 Hz**
- spline-based synchronization to a common time base
- bounded clipping to suppress nonphysical interpolation overshoot

### Expected processed output contract

Each processed InSight sol is saved as:

```python
np.savez(..., time=t_sync, X_state=X_state)
```

with the common column order:

```text
X_state[:, 0] = wind speed
X_state[:, 1] = air temperature
X_state[:, 2] = pressure
```

---

## MEDA / Perseverance workflow

The MEDA workflow:

1. defines the target sols:
   - `0044`
   - `0100`
   - `0315`
2. downloads/calibrates the required environmental channels from the Mars 2020 MEDA PDS archive
3. crops each record to the common **4410 s** window
4. applies physical gates and interpolation to create synchronized high-frequency state vectors
5. writes processed `.npz` state files
6. generates comparison figures and dimensionless-regime summaries

### MEDA channels used during preprocessing

- `CAL_WS` — wind speed
- `CAL_ATS` — atmospheric temperature
- `CAL_PS` — pressure
- `CAL_TIRS` — thermal-infrared surface/ground temperature

### MEDA harmonization logic

The paper and notebook workflow use:

- a **0.5 s** grid
- physical gating of variable ranges
- shape-preserving interpolation
- no extrapolation beyond valid temporal support

### Expected processed output contract

Each processed MEDA sol is saved as:

```python
np.savez(..., time=t_unified, X_state=X_state)
```

with the current notebook storing columns as:

```text
X_state[:, 0] = wind speed
X_state[:, 1] = atmospheric temperature
X_state[:, 2] = pressure
X_state[:, 3] = surface/ground temperature
```

For the cross-mission comparison, the shared atmospheric triplet is taken from the first three columns.

---

## Scientific method summary

The repository implements the paper’s comparative methodology:

1. **Acquisition**
   - fetch calibrated PDS atmospheric files for the target sols

2. **Cropping**
   - isolate a synchronized **4410 s** analysis interval per sol

3. **Harmonization**
   - construct common elapsed-time state vectors

4. **Diagnostics**
   - spectral fits using Welch PSD
   - reduced momentum-response surrogates
   - thermal/pressure state-space fits
   - dimensionless-regime analysis

5. **Robustness checks**
   - bootstrap confidence intervals
   - blocked cross-validation
   - sensitivity to derivative estimation
   - sensitivity to spectral settings
   - sensitivity to characteristic length scale

---

## Expected outputs

A successful full run should produce:

- raw PDS-derived CSV assets under `data/raw/`
- discovery/manifest CSV files
- processed synchronized state files:
  - `solXXXX_state_hf.npz`
- figure PDFs under `reports/`
- printed notebook summaries and tables for the analyzed sols

Representative figure outputs from the current notebook versions include:

- `Nature_Physics_Fig1.pdf`
- `Nature_Physics_Dimensionless_Final.pdf`

---

## Reproducing the paper results exactly

To stay as close as possible to the manuscript:

1. use the exact sol sets listed above
2. keep the default **4410 s** analysis window
3. do not change the variable order in the processed `X_state`
4. do not skip the preprocessing/harmonization cells
5. run the notebooks end-to-end in a clean environment
6. preserve the default report-generation cells

If you modify any of the following, your results will no longer be a strict reproduction:

- target sols
- time window length
- interpolation method
- filtering/gating thresholds
- frequency bands used for spectral fits
- characteristic length scale used in regime diagnostics

---

## Reusing the code on other sols or datasets

This repository is reusable beyond the exact paper setup.

### To analyze other sols

1. change the `TARGET_SOLS` list in the notebook
2. rerun the acquisition cells
3. rerun the harmonization cells
4. rerun the figure and summary cells

### To adapt the workflow to a new Mars atmospheric dataset

Preserve the processing contract:

- one `time` array
- one `X_state` matrix
- consistent column order for the atmospheric triplet:
  - wind speed
  - air temperature
  - pressure

That will let you reuse most downstream diagnostics with minimal edits.

### To compare additional missions or sites

Use the same structure:

- select comparable time windows
- harmonize onto a clearly defined time base
- standardize the state-vector layout
- keep the shared diagnostic functions unchanged
- document all mission-specific preprocessing decisions

---

## Data provenance and traceability

The notebooks write registry/manifest files specifically to preserve data provenance.

These files are useful for:

- tracking which PDS files were discovered and used
- confirming which sol-level assets were processed
- rerunning only part of the pipeline
- auditing the exact source inputs behind a figure

Do not delete these manifests if you want a clean audit trail.

---

## Troubleshooting

### The notebook fails at Drive mounting
Run in Google Colab, or replace the Colab mount block with a local directory path.

### A later cell fails because a file is missing
You probably skipped an earlier acquisition or preprocessing cell. Restart the runtime and rerun from the top.

### Local execution fails during package installation
Use Colab first for baseline reproduction, then port locally after the workflow is confirmed.

### PDS discovery returns unexpected files
Keep the sol filters and target duration unchanged for exact paper reproduction.

### Figures differ from the manuscript
Check:
- selected sols
- time window
- processed manifests
- package versions
- whether the full notebook was rerun from a clean state

---

## Citation

If you use this repository in research, please cite:

1. the companion manuscript:
   - K.M. Saqr (2026) **Evidence for non-Universal Reduced Closures and Sol-Dependent State-Space Relations in Mars Near-Surface Turbulence**
2. the relevant NASA PDS bundles used as source data
3. this GitHub repository

A simple repository citation format is:

```text
Saqr, Khalid M. Mars-Near-Surface-Turbulence. GitHub repository.
https://github.com/khalid-saqr/Mars-Near-Surface-Turbulence
```
