# Stereo Matching Testing Notebook

This document describes how to use `Stereo_Matching_Testing.ipynb` in this repository.

The notebook is part of the implementation set for your article and is designed as a companion to the stereo calibration workflow in:

- https://github.com/doc-ceb/Stereo_Calibration_Guide

In short:

- `Stereo_Calibration_Guide` covers camera calibration and rectification.
- `Stereo_Matching_Testing.ipynb` covers disparity estimation experiments and quantitative benchmarking on Middlebury data.

## Purpose

The notebook implements and evaluates a custom stereo block matching pipeline with:

- Matching costs: SAD, NCC, ZNCC
- Optional preprocessing: grayscale and Sobel
- Optional left-right consistency filtering
- Speckle filtering and hole filling
- Sub-pixel disparity refinement
- Multi-year Middlebury benchmarking

It produces per-scene visual outputs and per-year aggregated metrics that are ready for article tables and plots.

## Dataset Layout Expected by the Notebook

Dataset root expected by default:

- `./Middlebury_Stereo_Datasets`

Years used in the notebook:

- `2001`, `2003`, `2005`, `2006`, `2014`, `2021`

The loader contains year-specific filename logic. Example conventions inside scene folders include:

- 2001: `im2.ppm`, `im6.ppm`, `disp2.pgm`
- 2003: `im2.png`, `im6.png`, `disp2.png`
- 2005/2006: `view1.png`, `view5.png`, `disp1.png`
- Newer sets: `im0.png`, `im1.png`, `disp0.pfm`

## Software Requirements

Recommended:

- Python 3.11+
- Jupyter Notebook or JupyterLab

Core packages imported by the notebook:

- numpy
- opencv-python
- pandas
- matplotlib
- numba

Install with pip:

```bash
pip install numpy opencv-python pandas matplotlib numba jupyter
```

If you use conda, install equivalent packages in your active environment.

## Quick Start

1. Open the repository root.
2. Launch Jupyter:

```bash
jupyter notebook Stereo_Matching_Testing.ipynb
```

3. Run cells in order from top to bottom.
4. Verify `dataset_root` points to your local Middlebury folder.
5. Start with a single experiment config before running all benchmarks.

## Notebook Workflow

## 1) Data Loading and Parsing

The first section defines:

- `read_pfm(...)` for PFM disparity files
- `parse_middlebury_calib(...)` for `calib.txt`
- `MiddleburyLoader` with year-aware image/GT loading and optional downsampling

Important detail:

- When downsampling, disparity values are spatially resized and also scaled by the same factor to preserve geometric consistency.

## 2) Dataset Visualization

A pass over all configured years/scenes verifies loading and displays left/right/GT examples.

## 3) Stereo Matcher Implementation

The notebook includes a custom `StereoBMClone` built around numba-accelerated kernels:

- `calculate_cost_jit`: SAD, NCC, ZNCC patch costs
- `block_match_jit`: disparity search with uniqueness filtering
- `get_subpixel_disparity_jit`: parabola-based sub-pixel refinement
- `check_consistency_jit`: left-right check
- `speckle_filter_jit`: connected-component style speckle removal

Post-processing includes inpainting-based hole filling for invalid disparity regions.

## 4) Metric Computation

`compare_disparity_maps(...)` computes, on valid GT pixels:

- RMSE
- MAE
- Bad Pixel Percentage (thresholded absolute error)
- Coverage Percentage

These are the core metrics used for experiment comparisons.

## 5) Benchmark Runner

`run_stereo_benchmark(...)` handles:

- per-year scene iteration
- scene-level matcher execution per metric (SAD/NCC/ZNCC)
- saving per-scene visual comparisons
- writing per-year CSV files
- writing final per-year aggregated CSV summaries

## 6) Experiment Blocks in This Notebook

The notebook defines and runs multiple experiment families:

- `Exp1_Baseline_Raw` (grayscale prefilter)
- `Exp2_Sobel_Prefilter`
- `Exp3_Sobel_Prefilter_ExtendedDisp`
- `Exp4_Sobel_Prefilter_ExtendedDisp_BS_20`
- `Exp4_Sobel_Prefilter_ExtendedDisp_BS_40`
- `Exp6_Sobel_Prefilter_Ext_LRCheck`
- `Exp5_Sobel_Prefilter_LRCheck_SmallBlock`

Each experiment changes one or more factors:

- prefilter type
- max disparity range
- block size
- LR consistency enabled/disabled
- year-specific thresholds and scales

## 7) Depth Reconstruction Example

A later section demonstrates depth recovery from disparity using `calib.txt` parameters:

- focal length `f`
- baseline `b`
- disparity offset `doffs`

Depth is computed as:

```text
depth = (f * b) / (disparity + doffs)
```

This is useful for qualitative analysis figures in the article.

## Outputs Generated

For each experiment directory (for example `Exp2_Sobel_Prefilter`), the notebook typically creates:

- `Results_<year>/scene_compare.png` visual comparisons
- `Results_<year>/metrics_<year>.csv` per-scene metrics
- `FINAL_AGGREGATED_RESULTS.csv` year-level mean metrics

Some experiment folders in this repo already contain finalized aggregate files with custom names.

## Reproducibility Notes for Article Writing

- Keep the same Middlebury version and scene set between runs.
- Keep year-specific scale and threshold settings fixed.
- Run one warm-up call before timing due to numba JIT compilation.
- Report both quality metrics and runtime metrics.
- Keep LR-check settings explicit when comparing experiments.

## Relationship to Stereo_Calibration_Guide

Use both repositories as one pipeline in your manuscript:

1. Calibrate and rectify stereo cameras in `Stereo_Calibration_Guide`.
2. Run disparity algorithm experiments and quantitative benchmarking in this notebook.
3. Use output tables and visual comparisons for article figures and discussion.

## Troubleshooting

- Dataset not found: update `dataset_root` to your local path.
- Missing package errors: confirm active environment and install required packages.
- Very slow first run: expected due to numba compilation.
- Empty metrics for a scene: inspect GT validity and filename mapping for that year.

## License

This repository is under MIT License. See `LICENSE`.
