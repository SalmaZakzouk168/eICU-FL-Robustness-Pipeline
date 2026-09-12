# Robust Federated Learning for Intensive Care Unit Mortality Prediction Across a Heterogeneous Hospital Network

Reproducible pipeline accompanying:

> Robust Federated Learning for Intensive Care Unit Mortality Prediction Across a
> Heterogeneous Hospital Network. *Engineering Applications of Artificial Intelligence*, 2026 (submitted).

This repository contains the complete, single-notebook pipeline used to produce every table and figure
in the paper: cohort filtering → leakage/duplication diagnostics → feature extraction → centralized
architecture selection → class-imbalance handling → federated learning (FedAvg / FedProx) → robustness
stress-testing (label noise, input noise, Byzantine attacks: random-norm-matched, sign-flip, scaled-replacement)
→ hospital-size stratification → APACHE IVa benchmark comparison → coefficient-based explainability.
Multiple-comparisons correction (Holm–Bonferroni) and bootstrap confidence intervals are applied throughout
to support the paper's equivalence and robustness claims.

## Contents

| Path | Description |
|---|---|
| `eICU_FL_Robustness_Pipeline.ipynb` | The full pipeline, organized into 13 numbered sections (Setup → Final Summary), including cohort-flow, leakage, and train/val/test split-integrity diagnostics. |
| `requirements.txt` | Python package versions needed to run the notebook. |
| `eicu_data/` | **Empty placeholder.** You must populate this yourself with credentialed eICU-CRD v2.0 files (see below). Not included in this repository. |
| `cohort_output/`, `feature_output/`, `split_output/` | Intermediate pipeline artifacts (cohort tables, extracted features, train/val/test splits), generated when you run the notebook. Empty here. |
| `figures/` | Output figures (confusion matrix, ROC/PR curves, robustness-vs-severity plots for all three attack/noise types, combined robustness-degradation figure, feature-importance chart), generated when you run the notebook. |
| `results/` | Output summary tables (per-seed metrics, Wilcoxon/Mann-Whitney test results, Holm–Bonferroni-corrected significance tables, communication-cost and latency tables) corresponding to the paper's Tables 2–15. Generated when you run the notebook. |

## Important: this repository does NOT include patient data

The eICU Collaborative Research Database (eICU-CRD) is a restricted-access, credentialed dataset. Its
Data Use Agreement prohibits redistribution of the raw data by users. Accordingly:

- No raw patient-level files (`patient.csv`, `lab.csv`, `vitalPeriodic.csv`, `apachePatientResult.csv`) are
  included here, and none of the notebook's cell outputs contain row-level patient data — only aggregate
  statistics and model results.
- To reproduce the analysis, you must independently obtain credentialed access to eICU-CRD v2.0 via
  PhysioNet (see below) and place the four files above inside `eicu_data/`.

## Reproducing the analysis

1. **Get access to eICU-CRD v2.0.**
   - Register at [PhysioNet](https://physionet.org/) and become a credentialed user.
   - Complete the required training: *CITI Data or Specimens Only Research*.
   - Sign the eICU-CRD Data Use Agreement.
   - Download `patient.csv`, `lab.csv`, `vitalPeriodic.csv`, and `apachePatientResult.csv` from
     <https://physionet.org/content/eicu-crd/2.0/> and place them in `eicu_data/`.

2. **Set up the environment.**
   ```bash
   python -m venv venv
   source venv/bin/activate   # Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. **Run the notebook.**
   - Launch with `jupyter notebook eICU_FL_Robustness_Pipeline.ipynb`.
   - Run cells top to bottom. `BASE_DIR` defaults to the repository root (`./`), so no path edits are
     needed as long as `eicu_data/` is populated as above.
   - A GPU is optional; the notebook auto-detects CUDA and falls back to CPU (`DEVICE` in Section 0).
     All reported wall-clock figures in the paper (communication cost, inference latency) were measured
     on CPU.

4. **Outputs.**
   - Cohort tables → `cohort_output/`
   - Extracted feature tables and splits → `feature_output/`, `split_output/`
   - All figures → `figures/`
   - All summary/statistical-test tables → `results/`

## Reproducibility notes

- All experiments use a fixed 5-seed budget (`SEEDS = [42, 43, 44, 45, 46]`) and a unified 60-communication-round
  budget (`NUM_ROUNDS = 60`) across every federated configuration, including every robustness condition, so that
  natural-condition and stress-condition results are directly comparable.
- Hospital inclusion tier: hospitals must have ≥300 qualifying stays and ≥15 positive (mortality) cases to be
  retained as federated clients (`HOSPITAL_TIER = (300, 15)`). A global one-stay-per-`uniquepid` patient-level
  deduplication step is applied before this filter, yielding the **94-hospital, 83,909-patient cohort (7,957
  mortality events)** reported in the paper.
- A train/val/test split-leakage check confirms zero patient overlap across splits, and a within-hospital
  duplicate-`uniquepid` check is included alongside the cohort-flow table (raw records → unique patients at
  each filtering step).
- The full unfiltered cohort (Section 10b) reconciles to 207 hospitals / 97,963 patients, consistent with the
  deduplicated primary-cohort pool.
- Input-noise perturbation excludes the `-5` missingness sentinel and spans severities σ ∈ {0.01, 0.05, 0.10,
  0.25, 0.50, 1.00}. Label-noise threshold recalibration is selected on validation data only and frozen before
  a single application to the test set.
- Running the full pipeline end-to-end (all four phases, all severities, both aggregators, 5 seeds each) takes
  a nontrivial amount of time on CPU; budget accordingly.

## Citation

If you use this pipeline, please cite the paper above, and cite the eICU-CRD dataset itself following the
citation guidance at <https://physionet.org/content/eicu-crd/2.0/>.

## License

Code in this repository is released under the MIT License (see `LICENSE`). This license applies only to the
pipeline code, not to the eICU-CRD dataset, which remains governed by its own PhysioNet Data Use Agreement.
