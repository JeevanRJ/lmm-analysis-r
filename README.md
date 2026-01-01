# LMM Analysis in R: Fatigue × GVS (CF vs SD)

This notebook performs **within-activity statistical modeling** and **visualization** for sensorimotor outcomes under the interactive effects of:

- **Fatigue time point** (e.g., *Pre/Post* for CF or *6pm/10am* for SD)
- **Stimulation** (*NG* vs *G*)
- Random effect: **Subject** (random intercept)

It is designed to run on either the **Cognitive Fatigue (CF)** dataset or the **Sleep Deprivation (SD)** dataset by switching a small set of input paths and factor levels.

---

## What this notebook does

### 1) Loads libraries
Uses `tidyverse` for data handling + `lmerTest`/`afex` for LMM inference and `emmeans` for contrasts. `openxlsx` is used to export results to Excel.

### 2) Selects dataset: CF or SD
You manually choose one of the two “set inputs” blocks:

- **CF block**
  - `input_folder` → CF CSV folder
  - `fatigue_levels = c("pr","po")`
  - `Fatigue_type <- "CF"`

- **SD block**
  - `input_folder` → SD CSV folder
  - `fatigue_levels = c("6pm","2am","6am","10am")` (or subset)
  - `Fatigue_type <- "SD"`

> ✅ Tip: Keep only ONE of the two blocks active (comment the other) to avoid accidental overwrites.

### 3) Runs LMM / GLMM statistics per Activity
The function `run_fatigue_gvs_analysis()` loops over `Activity` and does:

- **Normality check** (Shapiro–Wilk) per `Fatigue × Stimulation`
- Optional **log-transform**: `Value = log(Value + 1e-6)`
- Fits either:
  - **LMM**: `Value ~ Fatigue * Stimulation + (1 | Subject)` (default)
  - **GLMM (Gamma)** via `glmmTMB` if enabled
- Saves (per activity):
  - Type-III ANOVA table
  - Model coefficients
  - Normality results (before/after log)
  - Pairwise `emmeans` contrasts for `Fatigue * Stimulation` (LMM/afex modes)

### 4) Generates publication-style plots
For a selected input CSV + variable, the script produces:

- Individual subject traces (with jittered points)
- Group mean line by stimulation condition
- Faceted by **Activity**
- Options:
  - outlier removal (IQR rule)
  - trial averaging (mean within Subject × Condition)
  - log-scale y-axis
  - split plots by gender

---

## Inputs

### Required CSV format
The notebook expects each `input_file` (e.g., `TW_TWP.csv`) to include at minimum:

- `Subject`
- `Activity`
- `Fatigue`
- `Stimulation` (values expected: `NG` and `G`)
- `Trial` (only needed if `average_trials = FALSE`)
- The dependent variable column (e.g., `TWP`, `XYZ_RMS`, `Steps`, etc.)

### Gender lists
Gender is inferred using the provided vectors:
- `male_subjects`
- `female_subjects`

---

## Key toggles you can change safely

### Plot toggles
- `remove_outliers <- TRUE/FALSE`
- `average_trials <- TRUE/FALSE`
- `use_log_scale <- TRUE/FALSE`
- `separate_by_gender <- TRUE/FALSE`

### Statistical toggles
Run this at the bottom:

```r
run_fatigue_gvs_analysis(
  df = plot_df,
  apply_log_transform = TRUE,
  use_glmm = FALSE,
  separate_by_gender = separate_by_gender,
  package_choice = "lmerTest",   # "lmerTest" or "afex"
  output_excel_base = paste0(Fatigue_type, "_", file_path_sans_ext(output_file), "_Stats")
)
```

- `apply_log_transform`: recommended for strictly-positive, skewed outcomes
- `use_glmm`: use `glmmTMB` Gamma model when LMM assumptions are poor
- `package_choice`:
  - `"lmerTest"` (default; straightforward)
  - `"afex"` (nice ANOVA tables; Satterthwaite method)

---

## Outputs

### Plots
Saved to the working directory with names like:
- `CF_TWP.png` (or gender-specific variants)
- `SD_TWP_Male.png`, `SD_TWP_Female.png` (if enabled)

### Statistics (Excel)
One Excel file per run, e.g.:
- `CF_TWP_Stats_All_Subjects.xlsx`
- `SD_TWP_Stats_Male.xlsx` / `SD_TWP_Stats_Female.xlsx` (if gender split)

Each worksheet corresponds to an **Activity** and contains:
- ANOVA table (main + interaction effects)
- Model coefficients
- Normality tests (before/after log)
- Pairwise `emmeans` contrasts (if applicable)

---

## How to run (recommended order)

1. Choose **CF or SD** input block and set:
   - `input_folder`
   - `fatigue_levels`
   - subject lists (if needed)
2. Choose the outcome to plot by setting:
   - `input_file`
   - `y_variable`
   - `plot_title`
   - `output_file`
3. Run the notebook top-to-bottom.
4. Check:
   - saved PNG plot(s)
   - Excel summary file(s)

---

## Notes / Common gotchas

- **Factor ordering & labels**:
  - For CF you map `pr/po` → `Pre/Post`. Make sure the raw data uses `pr` and `po` exactly.
- **Stimulation labels**:
  - Plot colors assume stimulation values are exactly `"NG"` and `"G"`.
- **Log-transform**:
  - If your outcome can be zero/negative, log-transform may be invalid.
- **Outlier removal**:
  - The IQR method is applied within `Activity × Fatigue × Stimulation`. If sample sizes are small, consider turning this off.

---

## License / Usage
MIT licence

