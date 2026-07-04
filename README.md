# Website Fingerprinting: Tamaraw Trade-off in the One-Page Setting

This repository is a standalone research codebase built on top of Xiao Deng’s Website-Fingerprinting-Library (WFlib). It evaluates how the Tamaraw defense trades off bandwidth overhead and website-fingerprinting accuracy in the **one-page** closed-world setting, using four attacks: **DF, TikTok, Var-CNN, and RF**.

The code is intended for research purposes only.

---

## Overview

This project studies Tamaraw schedules in the one-page setting by:

- generating Tamaraw-defended closed-world datasets for selected schedule parameters,
- converting those datasets into one-page subdatasets,
- training and evaluating website-fingerprinting attacks on those one-page datasets,
- aggregating and merging per-schedule summaries to compare schedules in terms of security and bandwidth cost.

The repository is organized around a Phase 2 experimental workflow rather than a general-purpose WFlib release.

---

## Repository layout

The cleaned-up repository structure at the root looks like:

```text
README.md
LICENSE
requirements.txt
Tamaraw.py
run_Tamaraw_CW.py
run_onepage_experiments.sh
run_onepage_experiments_cuda1.sh
run_tamaraw_padl_p_grid.sh

datasets/
docs/
exp/
figures/
scripts/
results/
results_padl/
results_tpr_fpr/
WFlib/
checkpoints/
logs/
logs_onepage/
logs_onepage_varcnn/
logs_tamaraw/
```

Main roles:

- `Tamaraw.py`: Tamaraw implementation and schedule logic.
- `run_Tamaraw_CW.py`: entry point for generating Tamaraw-defended CW datasets for given schedules.
- `run_tamaraw_padl_p_grid.sh`: helper script for running Tamaraw over a grid of pad lengths and parameters.
- `run_onepage_experiments.sh`, `run_onepage_experiments_cuda1.sh`: one-page experiment pipelines for selected schedule tags.
- `scripts/`: helper Python scripts (dataset generation, aggregation, merging, parsing).
- `results/`: final one-page summary CSVs and merged tables.
- `results_padl/`: pad-length level summaries and related outputs.
- `results_tpr_fpr/`: TPR/FPR-related summaries and plots.
- `figures/`: plots such as Accuracy vs Bandwidth and TPR/FPR vs Bandwidth.
- `datasets/`: CW, Tamaraw, and one-page datasets used in the experiments.
- `exp/`: WFlib-based training and evaluation code for DF, TikTok, Var-CNN, and RF.
- `logs*/` and `checkpoints/`: raw experiment logs and model checkpoints, kept for reproducibility.

---

## Environment setup

A dedicated conda or virtual environment is recommended, with Python 3.8 and a CUDA-enabled PyTorch installation if GPU training is used.

```bash
conda create -n wf38 python=3.8
conda activate wf38

pip install --user -e .
pip install -r requirements.txt
```

If needed, set `PYTHONPATH` to the repository root before running scripts:

```bash
export PYTHONPATH=$(pwd):$PYTHONPATH
```

---

## Base dataset setup

This project assumes the original closed-world dataset from WFlib.

1. Create the datasets directory:

```bash
mkdir -p datasets
```

2. Download the WFlib datasets, including `CW.npz`, from Zenodo (see WFlib documentation for the exact record).

3. Place the dataset file at:

```text
./datasets/CW.npz
```

4. Split the CW dataset into train, validation, and test sets if needed:

```bash
python exp/dataset_process/dataset_split.py --dataset CW
```

This produces the standard split files under `datasets/`.

---

## Phase 2 workflow

The main Phase 2 workflow is:

1. Generate Tamaraw-defended CW datasets for chosen schedules.
2. Convert those datasets into one-page subdatasets.
3. Run one-page attack experiments on selected schedule tags.
4. Aggregate per-tag results.
5. Merge all schedule summaries into combined CSVs and plot Accuracy vs Bandwidth and TPR/FPR vs Bandwidth.

Schedule tags follow the form:

```text
legacyPadl100_pinXpY_poutXpY_LZ_GW
```

For example:

```text
legacyPadl100_pin0p02_pout0p06_L1_G9
legacyPadl100_pin0p025_pout0p075_L3_G1
```

The tags encode pad length, Tamaraw parameters (`p_in`, `p_out`), and schedule parameters (`L`, `G`).

---

## Generating Tamaraw datasets

Use `run_Tamaraw_CW.py` to generate Tamaraw-defended CW datasets for a given schedule:

```bash
python run_Tamaraw_CW.py \
  --legacy_padl 100 \
  --p_in PIN \
  --p_out POUT \
  --L LVAL \
  --G GVAL
```

Example commands used in the experiments:

```bash
python run_Tamaraw_CW.py --legacy_padl 100 --p_in 0.025 --p_out 0.075 --L 1 --G 3
python run_Tamaraw_CW.py --legacy_padl 100 --p_in 0.025 --p_out 0.075 --L 1 --G 5
python run_Tamaraw_CW.py --legacy_padl 100 --p_in 0.025 --p_out 0.075 --L 1 --G 7
python run_Tamaraw_CW.py --legacy_padl 100 --p_in 0.025 --p_out 0.075 --L 1 --G 9

python run_Tamaraw_CW.py --legacy_padl 100 --p_in 0.025 --p_out 0.075 --L 3 --G 1
python run_Tamaraw_CW.py --legacy_padl 100 --p_in 0.025 --p_out 0.075 --L 5 --G 1
python run_Tamaraw_CW.py --legacy_padl 100 --p_in 0.025 --p_out 0.075 --L 7 --G 1
python run_Tamaraw_CW.py --legacy_padl 100 --p_in 0.025 --p_out 0.075 --L 9 --G 1
```

Other parameter settings explored during the project include, for example:

- `p_in = 0.02`, `p_out = 0.06`
- `p_in = p_out = 0.005`
- `p_in = p_out = 0.01`
- `p_in = p_out = 0.02`

Generated Tamaraw datasets are written under `datasets/` with names derived from the schedule tag.

---

## Building one-page datasets

After generating a Tamaraw-defended dataset, convert it into one-page subdatasets with `scripts/make_onepage_dataset.py`.

Example commands:

```bash
for TAG in \
  legacyPadl100_pin0p015_pout0p045_L1_G1 \
  legacyPadl100_pin0p025_pout0p075_L1_G1
do
  echo "============================================"
  echo "Generating one-page subdatasets for ${TAG}"
  echo "============================================"

  python scripts/make_onepage_dataset.py \
    --in_path "datasets/CW_tamaraw_${TAG}.npz" \
    --out_dir "datasets/CW_tam_${TAG}_pages" \
    --n_neg_per_pos 1 \
    --all_pages
done
```

Another batch used in the experiments:

```bash
for TAG in \
  legacyPadl100_pin0p025_pout0p075_L1_G3 \
  legacyPadl100_pin0p025_pout0p075_L1_G5 \
  legacyPadl100_pin0p025_pout0p075_L1_G7 \
  legacyPadl100_pin0p025_pout0p075_L1_G9
do
  echo "============================================"
  echo "Generating one-page subdatasets for ${TAG}"
  echo "============================================"

  python scripts/make_onepage_dataset.py \
    --in_path "datasets/CW_tamaraw_${TAG}.npz" \
    --out_dir "datasets/CW_tam_${TAG}_pages" \
    --n_neg_per_pos 1 \
    --all_pages
done
```

These commands create one-page dataset directories under `datasets/` for downstream experiments.

---

## Running one-page experiments

The repository includes shell scripts for launching one-page attack experiments on selected schedule tags.

Example batch:

```bash
for TAG in \
  legacyPadl100_pin0p02_pout0p06_L1_G9 \
  legacyPadl100_pin0p02_pout0p06_L3_G1 \
  legacyPadl100_pin0p02_pout0p06_L5_G1 \
  legacyPadl100_pin0p02_pout0p06_L7_G1
do
  bash run_onepage_experiments_cuda1.sh "${TAG}"
done
```

In general:

- `run_onepage_experiments.sh` provides the main one-page workflow (across attacks and metrics).
- `run_onepage_experiments_cuda1.sh` is used when targeting a specific GPU or device configuration.

These experiment scripts cover DF, TikTok, Var-CNN, and RF in the one-page setting, following the training and testing logic under `exp/`.

---

## Attack models

The main attacks evaluated in this repository are:

- **DF**
- **TikTok**
- **Var-CNN**
- **RF**

Training and evaluation follow the WFlib-based experiment structure under `exp/`. For each Tamaraw schedule and one-page dataset:

- DF and TikTok use direction/timing features directly from the Tamaraw dataset.
- Var-CNN uses its own timing-based feature configuration.
- RF can use Tamaraw-specific features derived from processed one-page datasets.

Experiment outputs (metrics, logs, checkpoints) are stored under `logs*/` and `checkpoints/`.

---

## Aggregating per-tag results

After experiments finish for a schedule tag, summarize the outputs with:

```bash
python scripts/aggregate_onepage_results.py --tag "${TAG}"
```

Example usage:

```bash
for TAG in \
  legacyPadl100_pin0p02_pout0p06_L9_G1 \
  legacyPadl100_pin0p025_pout0p075_L7_G1
do
  python scripts/aggregate_onepage_results.py --tag "${TAG}"
done
```

This produces per-tag summary CSV files such as:

```text
results/onepage_summary_legacyPadl100_pin0p02_pout0p06_L9_G1.csv
```

These files are used as inputs to the final merged summary.

---

## Merging all schedule summaries

To combine all per-tag summary CSV files into one master file, run:

```bash
python scripts/merge_onepage_summaries.py
```

The script reads a fixed list of per-tag summary CSV files and writes:

```text
results/onepage_all_schedules.csv
```

If a listed input file is missing, the script skips it and prints a warning. The merged table is the main artifact used to compare attack performance across Tamaraw schedules and to generate summary figures under `figures/`, `results_padl/`, and `results_tpr_fpr/`.

---

## Mixed and exploratory experiments

The repository also contains exploratory code for mixed settings. For example:

```bash
python scripts/make_onepage_mixed.py \
  --in_path datasets/CW_mix_K4_deltat0p01_N3.npz \
  --all_pages
```

These mixed or cross-circuit experiments are exploratory and should be interpreted separately from the main Tamaraw Phase 2 one-page schedule analysis.

---

## Reproducibility notes

This repository keeps large `logs*` folders and `checkpoints/` for reproducibility and later inspection. These directories may contain many files, but they preserve raw outputs from completed runs.

Key final artifacts include:

- per-tag one-page summary CSV files in `results/`,
- the merged `results/onepage_all_schedules.csv`,
- pad-length summaries in `results_padl/`,
- TPR/FPR-related summaries in `results_tpr_fpr/`,
- plots and tables under `figures/`.

---

## Attribution

This project builds on WFlib and prior work on website fingerprinting attacks and defenses. If you use this repository in academic work, please cite:

- X. Deng, Q. Li, and K. Xu, “Robust and Reliable Early-Stage Website Fingerprinting Attacks via Spatial-Temporal Distribution Analysis,” CCS 2024 (WFlib),
- the Tamaraw trade-off work associated with this project.

---

## Contact

For questions about this repository:

- Dong Quan Tran (Johnny) — [dxt9721@mavs.uta.edu](mailto:dxt9721@mavs.uta.edu)
- Dong Quan Tran (Johnny) — [dongquan.tran.johnny@gmail.com](mailto:dongquan.tran.johnny@gmail.com)
