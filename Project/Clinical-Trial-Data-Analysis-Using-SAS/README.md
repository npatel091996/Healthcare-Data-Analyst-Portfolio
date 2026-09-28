# Clinical Trial Data Analysis – SAS

**Course:** Health Data Analytics with SAS (BINF5210) · Rutgers University · Fall 2023
**Tools:** SAS (DATA step, PROC SORT, MEANS, FREQ, CONTENTS, SGPLOT, CORR, SURVEYSELECT, UNIVARIATE, GLM)
**Data:** `[RESTRICTED]` Primarily synthetic dataset provided by the course instructor. Stand-in data can be generated with [`data/generate_synthetic_data.py`](data/generate_synthetic_data.py).

---

## 1. Overview

This project tested whether Drug A lowers fasting blood glucose compared with a placebo. I built a SAS pipeline that cleaned and validated two patient-level files, joined them into an analysis set of 2,602 patients, and profiled the data. I then ran a one-way ANOVA on a simple random sample of 1,000 patients. **No significant difference was found in glucose test scores across the Placebo, Low Dose, and High Dose groups: F(2, 997) = 2.17, p = 0.1142.** Because the scores were not normally distributed, a non-parametric test is recommended to confirm the result.

## 2. Clinical / Business Problem

Participants were randomly assigned to one of three treatment arms, and their glucose reading was taken after ten weeks:

| Group | Treatment arm |
|---|---|
| Group 1 | Placebo |
| Group 2 | Low Dose Drug A |
| Group 3 | High Dose Drug A |

**Question:** Does Drug A reduce fasting glucose compared with placebo?

- **H0:** There is no difference in mean glucose test score across the groups.
- **H1:** At least one group's mean glucose test score differs.
- **Significance level:** α = 0.05, set before the test.

A second question looked at utilization: how length of stay relates to total charges and age.

The raw files had data-quality problems that are common in clinical and claims data: missing values, invalid entries in numeric fields, and records with no valid treatment group. These had to be fixed before any analysis could be trusted.

## 3. Data Sources & Standards

### Data provenance

| Dataset | Publisher | Version / year | URL | Date accessed |
|---|---|---|---|---|
| `Project2_Data-1.csv`: patient demographics and utilization (3,233 rows) | Course instructor, BINF5210, Rutgers University | Fall 2023 | Not publicly available `[RESTRICTED]` | November 13, 2023 (SAS import date) |
| `Project2_Data-2.csv`: treatment group and glucose test score (3,200 rows) | Course instructor, BINF5210, Rutgers University | Fall 2023 | Not publicly available `[RESTRICTED]` | November 13, 2023 (SAS import date) |

Both files are headerless and comma-delimited, and link on `PatientID`. The dataset is primarily synthetic and contains no protected health information (PHI). The original files are not redistributed in this repository.

### Healthcare data standards and code sets

None. The dataset has no diagnosis codes, procedure codes, provider identifiers, or interoperability formats (such as ICD-10, CPT/HCPCS, NPI, or HL7/FHIR).

## 4. Data Dictionary

No official data dictionary was provided with the files. Types come from the SAS import (PROC CONTENTS), and ranges come from the raw files.

**Dataset 1: `Project2_Data-1.csv`**

| Column | Data type (SAS) | Description | Allowed values / range in raw data |
|---|---|---|---|
| `PatientID` | Character | Unique patient identifier; join key | 1 – 3,239 (3,233 unique values) |
| `Age` | Numeric | Patient age in years | 4 – 100; 16 blank |
| `State` | Character | Patient state | `AZ` (1,222), `CA` (2,011) |
| `Lenght_of_Stay` | Numeric | Length of stay. Unit not stated in source `[TODO: confirm, likely days]` | 0 – 242; 2 invalid entries (`C`) |
| `Total_Charge` | Numeric | Total charges. Currency not stated in source `[TODO: confirm, likely USD]` | 474 – 931,294; 203 blank and 2 invalid entries (`C`) |

**Dataset 2: `Project2_Data-2.csv`**

| Column | Data type (SAS) | Description | Allowed values / range in raw data |
|---|---|---|---|
| `PatientID` | Character | Unique patient identifier; join key | 1 – 3,239 (3,200 unique values) |
| `Group` | Character | Treatment arm | `Placebo` (1,200), `Low` (800), `High` (800), `n/a` (400, invalid) |
| `Test_Score` | Numeric | Glucose test score after ten weeks. Unit not stated in source `[TODO: confirm unit]` | 20.48 – 1,962.41; none missing |

> The column name `Lenght_of_Stay` keeps the spelling used in the SAS code so the code runs as written.

## 5. Data Preparation

| Step | What was done | SAS | Rows after step |
|---|---|---|---|
| 1. Import | Read both CSVs with an explicit `INPUT` statement (`dlm="," dsd`) | DATA step | 3,233 / 3,200 |
| 2. Profile missing values | Counted missing values per column: Age 16, Length of Stay 2, Total Charge 205. Checked category counts for `State` and `Group` | PROC MEANS (NMISS), PROC FREQ | — |
| 3. Remove incomplete records | Dataset 1: dropped any row with a missing value (`cmiss(of _all_)`). Dataset 2: dropped rows where `Group = "n/a"` and any row with a missing value | DATA step | 3,012 / 2,800 |
| 4. Deduplicate | Sorted by `PatientID` with numeric collation and removed duplicate IDs. No duplicates were found | PROC SORT (NODUPKEY, SORTSEQ=LINGUISTIC) | 3,012 / 2,800 |
| 5. Validate | Confirmed row counts, variables, and sort order before merging | PROC CONTENTS | — |
| 6. Inner join | Merged on `PatientID`, keeping only patients present in both files (`if a=1 and b=1`) | DATA step MERGE | **2,602** |
| 7. Post-merge check | Confirmed zero missing values in all analysis variables | PROC MEANS, PROC FREQ | 2,602 |
| 8. Analysis sample | Drew a simple random sample for the ANOVA (seed = 8553) | PROC SURVEYSELECT (METHOD=SRS) | **1,000** |

The final analysis set had 1,037 AZ patients (39.85%) and 1,565 CA patients (60.15%). By treatment arm it had 1,118 Placebo (42.97%), 744 Low Dose (28.59%), and 740 High Dose (28.44%). No derived columns were created.

## 6. Methodology and Tools

**Exploratory analysis**
- Descriptive statistics for continuous variables, overall and by state (PROC MEANS)
- Frequency tables for state and treatment group (PROC FREQ)
- Box plots of age, length of stay, and total charges by state, plus pie charts of state and group
- Pearson correlation of length of stay, total charges, and age, with a scatter plot and regression line (PROC CORR, PROC SGPLOT)

**Hypothesis testing (one-way ANOVA)**
1. **Sampling.** Drew a simple random sample of 1,000 patients (selection probability 0.384).
2. **Normality.** Built histograms with normal curves and distribution statistics for each group (PROC UNIVARIATE).
3. **Equal variances.** Ran Levene's test with the `HOVTEST` option.
4. **Model.** Ran a one-way ANOVA in PROC GLM rather than PROC ANOVA, because group sizes in the sample were unbalanced (Placebo 431, Low 284, High 285). Diagnostic plots were requested with `plot=diagnostics`.

**Key code** (from the project program)

```sas
/* Remove incomplete and invalid records */
data P1.new_PF2;
    set P1.PF2;
    if Group = "n/a" then delete;
    if cmiss(of _all_) then delete;
run;

/* Deduplicate on Patient ID with numeric sort order */
proc sort data=P1.new_PF1 out=P1.final_PF1
          sortseq=linguistic (numeric_collation=on) nodupkey;
    by PatientID;
run;

/* Inner join */
data P1.merge_PF1_PF2;
    merge P1.final_PF1 (in=a) P1.final_PF2 (in=b);
    by PatientID;
    if a=1 and b=1;
run;

/* Simple random sample for ANOVA */
proc surveyselect data=P1.merge_PF1_PF2 method=srs
                  sampsize=1000 seed=8553 out=P1.final;
    id _all_;
run;

/* One-way ANOVA with Levene's test */
proc glm data=P1.final plot=diagnostics;
    class Group;
    model Test_Score = Group;
    means Group / hovtest;
run;
```

## 7. Results

**Descriptive statistics (analysis set, n = 2,602)**

| Variable | Mean | Std Dev | Min | Max |
|---|---|---|---|---|
| Age | 54.76 | 18.91 | 7 | 100 |
| Length of stay | 5.01 | 7.40 | 0 | 242 |
| Total charge | 25,203.83 | 44,494.55 | 474 | 931,294 |
| Test score | 96.75 | 77.95 | 20.48 | 1,736.58 |

By state, CA patients were older on average than AZ patients (56.80 vs. 51.68 years) and had longer mean stays (5.42 vs. 4.39) and higher mean charges (28,468.10 vs. 20,277.52). Both states had heavy right tails in length of stay and charges. No test for differences between states was run.

**Correlation (Pearson, n = 2,602)**

| Pair | r | p-value |
|---|---|---|
| Length of stay vs. total charge | 0.618 | < .0001 |
| Length of stay vs. age | 0.093 | < .0001 |
| Total charge vs. age | 0.035 | 0.0733 |

**ANOVA assumption checks (sample, n = 1,000)**

| Assumption | Result |
|---|---|
| Normality | Not met. Histograms were strongly right-skewed in all three groups (skewness 2.20 to 2.26) |
| Sample size | More than 30 per group, but group sizes were unbalanced, so PROC GLM was used |
| Equal variances | Met. Levene's test F = 1.19, p = 0.3046 |

**One-way ANOVA: glucose test score by treatment group**

| Group | n | Mean | Std Dev | Median |
|---|---|---|---|---|
| Placebo | 431 | 102.41 | 77.34 | 76.64 |
| Low Dose Drug A | 284 | 90.82 | 65.21 | 67.96 |
| High Dose Drug A | 285 | 98.89 | 73.96 | 73.61 |

| Source | DF | Sum of squares | Mean square | F | p |
|---|---|---|---|---|---|
| Group | 2 | 23,243.55 | 11,621.77 | 2.17 | 0.1142 |
| Error | 997 | 5,329,058.47 | 5,345.09 | | |
| Corrected total | 999 | 5,352,302.01 | | | |

R² = 0.0043. Because p = 0.1142 is greater than α = 0.05, the study **failed to reject the null hypothesis**. There was no statistically significant effect of Drug A on glucose test scores.

## 8. Healthcare Impact & Insights

- **No evidence of efficacy at either dose.** Neither dose of Drug A produced a glucose reduction that could be told apart from placebo. Treatment group explained less than 0.5% of the variation in scores (R² = 0.0043). A result like this would not support moving the drug forward on glucose-lowering grounds without further study.
- **Length of stay drives cost.** Length of stay had a strong positive correlation with total charges (r = 0.618). This makes length of stay a practical lever to watch in utilization and cost reporting. Age had almost no relationship with charges.
- **Data quality decides the denominator.** Cleaning and joining cut the usable population from 3,233 to 2,602 records (about 20%). Most of the loss came from missing charges and invalid group labels. Checking missing-value counts, key uniqueness, and row counts at each step, as done here, keeps trial and claims reporting traceable and defensible.
- **Check the test's assumptions before trusting the p-value.** The group distributions were strongly skewed with extreme values. A regulatory or clinical audience would expect the parametric result to be confirmed by a test that doesn't assume normality.

## 9. Limitations & Future Work

**Limitations**
- The glucose test scores were not normally distributed, so the ANOVA result may not be reliable. The project assumed ANOVA was robust given the sample size.
- Group sizes were unbalanced in the sample (431 / 284 / 285).
- The analysis used a random sample of 1,000 from 2,602 eligible patients, which lowers statistical power compared with using the full set.
- Records with any missing value were deleted. If the missing values were not random, this could bias the results.
- Extreme values in test scores, length of stay, and charges were kept, and they influence means and correlations.
- The data is primarily synthetic teaching data, so the findings don't describe a real drug or patient population.
- Units for length of stay, charges, and test score were not documented.

**Future work**
- Run a Kruskal-Wallis test to confirm whether the ANOVA conclusion holds without the normality assumption. *(Recommended in the original project.)*
- `[TODO: Add any other next steps you want to list. Only the Kruskal-Wallis test was recommended in the project materials.]`

## 10. How to Reproduce

The original data is restricted, so these steps use the stand-in data. The stand-in has the same structure but simulates **no drug effect**, so your numbers will differ from the results above.

1. **Clone the repository**
   ```bash
   git clone https://github.com/[TODO:GITHUB_USERNAME]/Healthcare-Data-Analyst-Portfolio.git
   cd Healthcare-Data-Analyst-Portfolio/projects/clinical-trial-sas
   ```
2. **Set up Python and install requirements**
   ```bash
   python -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```
3. **Generate the stand-in data**
   ```bash
   python data/generate_synthetic_data.py --validate
   ```
   This writes `data/synthetic/Project2_Data-1.csv` and `data/synthetic/Project2_Data-2.csv`, then prints row counts at each cleaning step next to the original counts.
   If you have authorized access to the original files, put them in `data/` instead. `.gitignore` keeps them from being committed.
4. **Open the SAS program.** Open `code/[TODO: SAS program file name].sas` in SAS 9.4 or SAS OnDemand for Academics.
5. **Update the paths.** Change the three lines at the top:
   - `libname P1` should point to a folder for the SAS library.
   - `filename project1` should point to `data/synthetic/Project2_Data-1.csv`.
   - `filename file2` should point to `data/synthetic/Project2_Data-2.csv`.
6. **Run the program top to bottom.** Check that the output shows these counts:
   - After cleaning: about 3,010 and 2,800 rows.
   - After the merge: about 2,600 rows.
   - The sample has 1,000 rows, since the SURVEYSELECT seed is fixed at 8553.
7. **Compare the output with this case study.** Review the PROC MEANS, CORR, UNIVARIATE, and GLM output against sections 5–7.
