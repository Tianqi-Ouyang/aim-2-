# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@~/.claude/hospital-privacy.md

## Project Overview

Aim 2 is a clinical-research analysis investigating **acute interstitial
nephritis (AIN)** in melanoma patients receiving **immune checkpoint inhibitors
(ICI)**. It is a **standalone project** with its own longitudinal REDCap project,
cohort, and Statistical Analysis Plan (`Tianqi_SAP.docx`).

The cohort is three study arms:

1. Metastatic melanoma receiving ICI
2. Adjuvant stage II–III melanoma receiving ICI
3. Control: stage I–IIIA melanoma **not** receiving ICI

The current scope is **variable creation** — translating the SAP into reproducible
derived variables. Downstream Table 1 / analysis files are not built yet.

## Running the Analysis

Quarto project. Render the website from the project root:

```bash
quarto render
```

`qmd/aim2_variables.qmd` builds every SAP-specified analysis variable and writes
`aim2_master.rds`; downstream analysis files (to be added) consume that rds.
Output lands in `docs/`. `execute.eval` is `false` in `_quarto.yml`, so the site
renders for code review even before the REDCap/RPDR data files are present.

## Files

- **`qmd/aim2_variables.qmd`** — the variable-creation pipeline. Loads the
  longitudinal REDCap export + RPDR labs and produces `aim2_master.rds`.
- **`index.qmd`** — study overview (arms, data sources, key variables).
- **`Tianqi_SAP.docx`** — Statistical Analysis Plan; the source of truth for every
  variable definition. Git-ignored (not committed).
- **`Data/`** — REDCap CSV + RPDR `.txt` inputs (PHI; git-ignored). Set the paths
  in the **Configuration** chunk (`#| label: config`) of `qmd/aim2_variables.qmd`.

## Data Pipeline & Architecture

### Longitudinal REDCap structure (critical)

Aim 2's REDCap export is **longitudinal**: one row per
(`record_id` × `redcap_event_name`). Variables must be pulled from the correct
event, then joined back to one row per `record_id`.
`qmd/aim2_variables.qmd` splits the export into three event frames up front:

| Event | Variables sourced |
|---|---|
| `prebaseline_arm_1` | demographics, comorbidities, concomitant meds (`ppi`, `nephrotox_drugs___*`, `con_med`) |
| `baseline_arm_1` | `egfr`, baseline `visit_date`, `EMPI` |
| `early_treatment_we_arm_1` | `ici_type` (defines baseline ICI regimen) |

### Derived-variable conventions

- **Checkbox fields** export as `field___N` coded 0/1. Composite indicators OR
  the relevant boxes via the local `any_box()` helper (NA → 0).
- **`autoimmune_history`** — 1 if any of 18 specified autoimmune comorbidity
  boxes is checked. The exact box list is in the `autoimmune-history` chunk.
- **`baseline_ckd_stage`** — KDIGO G1–G5 via `cut(egfr, breaks, right = FALSE)`;
  band boundaries are inclusive at the top (G2 = 60–89, G1 ≥90).
  **`baseline_ckd_binary`** collapses to 1 if eGFR <60 (G3a–G5).
- **`baseline_ici_treatment`** comes from `ici_type` at the **early-treatment**
  event (not pre-baseline) — the SAP defines the established regimen there.
  **`ici_regimen_class`** collapses the 11 types into 4 mechanism classes
  (PD-(L)1 mono = 1,2,5,6,7; CTLA-4 = 3,8; LAG-3 = 4,9; Other/None = 10,11).
- **AIN concomitant meds** — six classes (PPI, antibiotics, TMP-SMX, NSAIDs,
  allopurinol, anti-epileptics). Each is present if its **structured REDCap field
  equals 1 OR** a brand/generic search term matches the free-text `con_med` field
  (case-insensitive, regex-escaped). **`AIN_con_med_any`** = any of the six.
- **`baseline_hgb` / `baseline_albumin`** — RPDR lab with the smallest absolute
  day-difference from the baseline visit; ties prefer the **prior** value; >30
  days → missing. Implemented by the local `closest_lab()` helper. This chunk is
  `eval: false` until the RPDR pull + an `EMPI`↔`record_id` crosswalk are available.

### Reference categories / coding

- **sex**: 1=Male, 2=Female, 3=Other (labelled factor)
- **study_arm**: 1=Metastatic+ICI, 2=Adjuvant+ICI, 3=Control no-ICI
- Binary comorbidity / medication indicators: 0 = Absent, 1 = Present

## Key R Packages

tidyverse / dplyr / readxl (data), lubridate (dates), stringr + regex (free-text
`con_med` search). Downstream analysis is expected to use gtsummary/tableone for
Table 1 and tidycmprsk for competing-risks (if AIN is modelled with death as a
competing event).

## Project Conventions

- `execute.eval: false` so the site renders without PHI data present.
- PHI inputs (REDCap CSV, RPDR `.txt`, `.xlsx`, `.rds`) are git-ignored; only code
  and rendered `docs/` HTML are committed.
- Baseline RPDR labs are selected by smallest absolute day-difference from the
  baseline visit (ties prefer the prior value); see `closest_lab()`.
