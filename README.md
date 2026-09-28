# Excess-rate and set-rate mechanical power and mortality

Analysis code for the study:

> Orso D, Borrelli C, Brussa A, Della Rocca G. *Excess-Rate and Set-Rate Mechanical Power and Mortality in Mechanically Ventilated Adults: A Two-Cohort Study.* Submitted to *Critical Care Medicine*.

Calculated mechanical power (MP) is partitioned, using separate charting of set and total respiratory rate, into a **set-rate** component and an **excess-rate** component (power associated with breaths more than 2 breaths/min above the set rate). The partition is arithmetic: the excess-rate component is not a measurement of respiratory muscle power. Associations with 28-day mortality are estimated in MIMIC-IV (derivation) and validated in eICU-CRD under a prespecified plan. An exploratory analysis compares excess-rate power with excess respiratory rate.

Figures, tables, and aggregate model estimates are archived on Figshare: https://doi.org/10.6084/m9.figshare.34012659

## Data access

This repository contains **code only**. No patient-level data are included or may be shared.

Both databases are available on [PhysioNet](https://physionet.org) to credentialed users who complete the required training and sign the data use agreement:

- MIMIC-IV, version TODO — https://physionet.org/content/mimiciv/
- eICU Collaborative Research Database, version 2.0 — https://physionet.org/content/eicu-crd/

## Directory layout

The scripts read the compressed CSV files (`.csv.gz`) directly with DuckDB; no database server is needed. Set the environment variable `MP_DATA_DIR` to the folder containing the data (default on Windows: `%USERPROFILE%/Desktop/mimiciv`):

```
MP_DATA_DIR/                           MIMIC-IV .csv.gz files (any subfolders, e.g. hosp/, icu/)
MP_DATA_DIR/eICU/                      eICU-CRD .csv.gz files
MP_DATA_DIR/mp_study/                  created by the scripts (MIMIC-IV outputs)
MP_DATA_DIR/eICU/mp_study_eicu/        created by the scripts (eICU-CRD outputs)
MP_DATA_DIR/mp_study/revisione/        created by the revision scripts (figures, sensitivity analyses)
```

When searching for MIMIC-IV files, subfolders named `eICU` and `nwc` are ignored.

```r
Sys.setenv(MP_DATA_DIR = "path/to/data")   # or set it in .Renviron
```

## Requirements

R (≥ 4.2) with `data.table`, `DBI`, `duckdb`, `survival`, `mice`, `MASS`, `splines`, `ordinal`, `ggplot2`, `patchwork`, `scales`, `flextable`, and `officer`. Run `R/00_install_packages.R` to install them. The versions used (R 4.5.0; data.table 1.18.4, duckdb 1.5.5, mice 3.19.0, survival 3.8-6) are listed in `session_info.txt`.

## Run order

### Primary pipeline (`R/`)

| Script | Purpose | Main outputs |
|---|---|---|
| `01_mimic_feasibility.R` | MIMIC-IV cohort, hourly ventilator table, MP calculation and partition | `mp_orario_24h.csv`, `mp.duckdb` |
| `02_mimic_analytic_dataset.R` | Confounders, demand markers, Charlson index, diagnoses, ventilator-free days, passivity, measurement error | `mp_dataset_analitico.csv` |
| `03_mimic_models_cox.R` | Cox models 1–3b, sensitivity analyses, passive stratum, E-values | `mp_risultati_modelli.csv` |
| `04_mimic_negative_control_feasibility.R` | Pre-intubation respiratory rate (negative-control exposure) | `mp_controllo_negativo_fattibilita.csv` |
| `05_mimic_additional_analyses.R` | Primary logistic models, split follow-up, negative-control models | `mp_risultati_aggiuntivi.csv` |
| `06_eicu_reconnaissance.R` | eICU-CRD label inventory and charting quality by hospital | `eicu_etichette_resp.csv` |
| `07_eicu_feasibility.R` | eICU-CRD cohort and MP partition | `eicu_coorte.csv`, `eicu_orario_24h.csv` |
| `08_eicu_analytic_dataset.R` | APACHE IV, blood gases, neuromuscular blockade, diagnoses | `eicu_dataset_analitico.csv` |
| `09_eicu_models.R` | Prespecified validation models and replication criteria | `eicu_risultati_modelli.csv`, `eicu_criteri_replica.csv` |
| `10_pooled_estimates.R` | Pooled estimates and heterogeneity | `mp_stima_combinata.csv` |
| `11_figures.R` | Original figures (superseded for submission by `R/revision/`) | TIFF, PDF |
| `12_flowchart.R` | Original flow diagram (superseded by `R/revision/17_figure1_flowchart.R`) | TIFF, PDF |
| `13_tables.R` | Tables 1, 2 and supplemental tables | `mp_tables.docx`, CSV |
| `14_collect_outputs_for_figshare.R` | Copies aggregate outputs and checks for patient-level identifiers | Figshare folder |
| `99_session_info.R` | Records R and package versions | `session_info.txt` |

### Revision analyses and submission figures (`R/revision/`)

Run after the primary pipeline and after the model bundles (`model_bundle*.rds`, 20 imputations per cohort) have been saved.

| Script | Purpose | Main outputs |
|---|---|---|
| `00_common.R` | Paths and file search shared by the revision scripts | — |
| `15_nmba_sensitivity.R` | Models 2, 3a, 3b without neuromuscular blocker exposure and in unexposed patients | `nmba_sensitivity.csv` |
| `16_flowchart_check.R` | Checks the 24-hour record-span criterion and deaths before the landmark | `flowchart_check.txt` |
| `17_figure1_flowchart.R` | Figure 1, counts computed from the analytic files | `Figure1.tiff`, `Figure1.pdf` |
| `18_figures_2_3_S2.R` | Figure 2 (composition), Figure 3 (adjustment steps), Supplemental Figure 2 (fixed-effect pooling) | TIFF |
| `19_figure4_gcomputation.R` | Figure 4, g-computation from model 3a (20 imputations × 50 coefficient draws) | `Figure_4.tiff`, CSV |
| `20_figureS3_or_curves.R` | Supplemental Figure 3, Rubin-pooled odds-ratio curves | `FigureS3.tiff`, `FigureS3.pdf` |

### Exploratory analyses (`R/exploratory/`)

Added after the primary analyses; not part of the prespecified replication criteria.

| Script | Purpose | Main outputs |
|---|---|---|
| `m3a_or_curves_20_imputations.R` | Model 3a odds-ratio curves (Rubin pooling, 20 imputations); compares with an existing `M3a_curves_20_imputations.csv` | `M3a_curves_20_imputations_rebuilt.csv` |
| `supplemental_table4_rr_vs_mp.R` | Excess respiratory rate vs excess-rate power: separate, joint, and spline-adjusted models | `supplemental_table4_rr_vs_mp.csv` |
| `supplemental_figure4_rr_mp_map.R` | Supplemental Figure 4, joint distribution of the two exposures | TIFF, PDF, `figure_summary.csv` |

`R/supplementary/nwicu_reconnaissance.R` documents why the Northwestern ICU database (v0.1.0) could not be used: it contains no ventilator settings.

The first runs of scripts 01, 02, 04, 06, and 08 scan large tables (`chartevents`, `labevents`, `nurseCharting`) and may take several minutes. Extracted tables are cached in DuckDB files; delete them to force re-extraction. Multiple imputation (20 data sets, 10 iterations, seed 20260928) takes 5–15 minutes per script.

## Validation plan

The eICU-CRD validation plan and replication criteria are reported in `docs/eicu_validation_plan.md` and in the header of `R/09_eicu_models.R`. The date of the plan is stated in the document; the commit date of this repository is not evidence of when the plan was fixed.

## Notes

- Comments and console messages are in English or Italian. Variable names keep the Italian labels used during the analysis, for example: `evento_28` = 28-day death; `tempo_28` = time to death or censoring (days); `mp_b_set_s2` / `mp_b_ecc_s2` = set-rate / excess-rate MP (> 2 breaths/min threshold); `quota` = excess-rate share; `passivo` = neuromuscular blockade in at least two thirds of observations; `coorte_primaria` = primary cohort; `stima` = estimate.
- Earlier versions labeled the excess-rate component "patient-generated"; the current label reflects that the partition does not measure patient muscle power.
- Figures are produced in American English, at 600 dpi (TIFF, LZW compression), without titles; legends are provided with the manuscript.
- `tools/prepare_release.R` assembles this repository and the Figshare package and checks that no patient-level file is included.

## License

Code: MIT License (see `LICENSE`). Figures, tables, and aggregate results on Figshare: CC BY 4.0.

## Citation

See `CITATION.cff`.
