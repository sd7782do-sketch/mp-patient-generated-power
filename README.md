# Patient-generated and ventilator-set mechanical power and mortality

Analysis code for the study:

> Orso D, Borrelli C, Brussa A, Della Rocca G. *Patient-Generated and Ventilator-Set Mechanical Power and Mortality in Mechanically Ventilated Adults: A Two-Cohort Study.* (submitted)

Mechanical power (MP) calculated from airway pressures is decomposed into a **ventilator-set** component (computed with the set respiratory rate) and a **patient-generated** component (computed with the rate in excess of the set rate). Associations with 28-day mortality are estimated in MIMIC-IV (derivation) and validated in eICU-CRD under a prespecified plan.

Figures, tables, and aggregate model estimates are archived on Figshare: https://doi.org/10.6084/m9.figshare.34012659

## Data access

This repository contains **code only**. No patient-level data are included or may be shared.

Both databases are available on [PhysioNet](https://physionet.org) to credentialed users who complete the required training and sign the data use agreement:

- MIMIC-IV <!-- TODO: version used, e.g. v2.2 --> — https://physionet.org/content/mimiciv/
- eICU Collaborative Research Database v2.0 — https://physionet.org/content/eicu-crd/

## Directory layout

The scripts read the compressed CSV files (`.csv.gz`) directly with DuckDB; no database server is needed. Set the environment variable `MP_DATA_DIR` to the folder containing the data (default on Windows: `~/Desktop/mimiciv`):

```
MP_DATA_DIR/                     MIMIC-IV .csv.gz files (any subfolders, e.g. hosp/, icu/)
MP_DATA_DIR/eICU/                eICU-CRD .csv.gz files
MP_DATA_DIR/mp_study/            created by the scripts (MIMIC-IV outputs, figures, tables)
MP_DATA_DIR/eICU/mp_study_eicu/  created by the scripts (eICU-CRD outputs)
```

When searching for MIMIC-IV files, subfolders named `eICU` and `nwc` are ignored.

```r
Sys.setenv(MP_DATA_DIR = "path/to/data")   # or set it in .Renviron
```

## Requirements

R (≥ 4.2) with: `data.table`, `DBI`, `duckdb`, `survival`, `mice`, `MASS`, `ggplot2`, `scales`, `flextable`, `officer`. Run `R/00_install_packages.R` to install them. The versions used are listed in `session_info.txt`.

## Run order

| Script | Purpose | Main outputs |
|---|---|---|
| `01_mimic_feasibility.R` | MIMIC-IV cohort, hourly ventilator table, MP calculation and decomposition | `mp_orario_24h.csv`, `mp.duckdb` |
| `02_mimic_analytic_dataset.R` | Confounders, demand markers, Charlson index, diagnoses, ventilator-free days, passivity, measurement error | `mp_dataset_analitico.csv` |
| `03_mimic_models_cox.R` | Cox models 1–3b, sensitivity analyses, passive stratum, E-values | `mp_risultati_modelli.csv` |
| `04_mimic_negative_control_feasibility.R` | Pre-intubation respiratory rate (negative control exposure) | `mp_controllo_negativo_fattibilita.csv` |
| `05_mimic_additional_analyses.R` | Primary logistic models, split follow-up, negative control models | `mp_risultati_aggiuntivi.csv` |
| `06_eicu_reconnaissance.R` | eICU-CRD label inventory and charting quality by hospital | `eicu_etichette_resp.csv` |
| `07_eicu_feasibility.R` | eICU-CRD cohort and MP decomposition | `eicu_coorte.csv`, `eicu_orario_24h.csv` |
| `08_eicu_analytic_dataset.R` | APACHE IV, blood gases, neuromuscular blockade, diagnoses | `eicu_dataset_analitico.csv` |
| `09_eicu_models.R` | Prespecified validation models and replication criteria | `eicu_risultati_modelli.csv`, `eicu_criteri_replica.csv` |
| `10_pooled_estimates.R` | Pooled estimates and heterogeneity; Supplementary Figure S1 | `mp_stima_combinata.csv`, `figS1_forest_combinato.tiff` |
| `11_figures.R` | Figures 2–5 | `fig2`–`fig5` (TIFF, PDF) |
| `12_flowchart.R` | Figure 1 (flow diagram) | `fig1_flowchart` (TIFF, PDF) |
| `13_tables.R` | Tables 1, 2, S1, S2 | `mp_tables.docx`, CSV |
| `14_collect_outputs_for_figshare.R` | Copies final figures, tables, and aggregate results into the Figshare folder and checks that no file contains patient-level identifiers | Figshare folder |
| `99_session_info.R` | Records R and package versions | `session_info.txt` |

`R/supplementary/nwicu_reconnaissance.R` documents why the Northwestern ICU database (v0.1.0) could not be used: it contains no ventilator settings.

The first runs of scripts 01, 02, 04, 06, and 08 scan large tables (`chartevents`, `labevents`, `nurseCharting`) and may take several minutes. Extracted tables are cached in DuckDB files; delete them to force re-extraction. Multiple imputation (20 data sets) in scripts 03, 05, 09, and 11 takes 5–15 minutes per script.

## Prespecified validation plan

The eICU-CRD validation plan and replication criteria were fixed before the validation models were estimated; they are reported in `docs/eicu_validation_plan.md` and in the header of `R/09_eicu_models.R`.

## Notes

- Comments and console messages are in American English. Variable names, column names, and some output labels keep the Italian labels used during the analysis, so that the scripts remain compatible with the intermediate files, for example:
  `evento_28` = 28-day death; `tempo_28` = time to death or censoring (days); `mp_b_set_s2` / `mp_b_ecc_s2` = ventilator-set / patient-generated MP (> 2 breaths/min threshold); `quota` = patient-generated share; `passivo` = passive (neuromuscular blockade in at least two thirds of observations); `coorte_primaria` = primary cohort; `stima` = estimate; `primaria` = primary analysis.
- Figures are produced in American English, at 600 dpi (TIFF, LZW compression), without titles or captions; legends are provided with the manuscript.

## License

Code: MIT License (see `LICENSE`). Figures, tables, and aggregate results on Figshare: CC BY 4.0.

## Citation

See `CITATION.cff`.

