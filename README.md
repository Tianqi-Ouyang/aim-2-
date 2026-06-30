# Aim 2 — Melanoma · ICI · Acute Interstitial Nephritis

Clinical-research analysis examining **acute interstitial nephritis (AIN)** in
melanoma patients receiving **immune checkpoint inhibitors (ICI)**, with its own
longitudinal REDCap project and Statistical Analysis Plan (`Tianqi_SAP.docx`).

## Study arms

1. Metastatic melanoma receiving ICI
2. Adjuvant stage II–III melanoma receiving ICI
3. Control: stage I–IIIA melanoma not receiving ICI

## Render

```bash
quarto render
```

`qmd/aim2_variables.qmd` builds every SAP-specified analysis variable and writes
`aim2_master.rds`. Rendered website lands in `docs/`.

## Inputs (not committed)

| File | Source |
|---|---|
| Longitudinal REDCap export (CSV) | Demographics, comorbidities, ICI regimen, concomitant meds, baseline eGFR |
| RPDR `*_Lab.txt` | Baseline hemoglobin & albumin |

Set the paths in the **Configuration** chunk of `qmd/aim2_variables.qmd`. Data
files contain PHI/EMPI and are excluded by `.gitignore`.

See `CLAUDE.md` for the variable-derivation logic and REDCap event structure.
