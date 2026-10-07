# Delivery 4 - Analysis Design and Modelling Strategy

> **Revision note (response to feedback received):** the feedback points to two specific mismatches between what was intended to be produced and the data available - a weekly table cannot justify predictions at 24–72 hours, and an annual average occupancy does not represent real bed availability for recommending transfers -; a serious methodological problem in the episode-level scoring approach (splitting by `episodio_id` does not prevent several episodes of the same patient from appearing simultaneously in training and test; a patient identifier and group-based validation, preferably also temporal, are required); and the risk of training and validating with synthetic readmissions, which may lead the model to reproduce the rules used to generate the label instead of demonstrating real clinical validity. It also asks to prioritise a single module and to strictly align its granularity, horizon and source of truth. This document applies those four points: the detailed analysis and modelling focuses on **Module 1 (demand forecasting)**, with an explicit verification that its horizon (4–8 weeks) is consistent with its granularity (weekly) - predictions at 24–72 hours on weekly data are never proposed. Modules 2 and 3 are moved to an annex of conceptual, non-implemented design, in which the methodological corrections indicated (patient identifier and group/temporal validation, and the risk of circularity with synthetic labels) are explicitly incorporated in case they are activated in the future.

---

## 1. Problem to Be Solved

### Current Situation

SNS hospitals manage the planning of care demand in a predominantly reactive manner. The absence of data-based tools makes it difficult to anticipate admission peaks with sufficient lead time to plan staffing and bed allocation.

### Who Uses the Result and for Which Decision

| Module | End user | Decision supported | Status |
|---|---|---|---|
| **M1 - Demand** | Medical directorate / bed management | Plan staffing 4–8 weeks in advance | **MVP of this deliverable** |
| M2 - Transfer | Medical directorate of a health area / network planning | Prioritise the pathologies/CCAA in which to reinforce specialised capacity ahead of the season | Conceptual design, see Annex |
| M3 - Readmission risk | Medical directorate / care quality | Prioritise the segments in which to reinforce discharge or post-discharge follow-up protocols | Conceptual design, see Annex |

Module 1 does not operate in real time: the source (EMH) is published with a delay and at aggregated granularity. Its value lies in **structural planning several weeks ahead**, not in day-to-day operational alerting – for this reason the horizon is kept at 4–8 weeks, and a 24–72-hour horizon, which would require daily or hourly updated data that the EMH does not provide, is at no point proposed.

### What the Project Must Produce to Be Considered Useful

A dashboard with two blocks: historical EDA and demand forecasting (M1). It is considered useful if M1 outperforms the naive baseline. The M2/M3 panels are not part of the success criterion of this deliverable.

---

## 2. Planned Data Analysis (EDA) and Expected Utility

### Critical Business Questions (M1)

- Seasonality of urgent admissions.
- Differences between CCAA (and between departments, if Delivery 3 confirms that breakdown).
- Stationarity of the series.
- Treatment of the 2020–2021 structural break.

### Specific Analyses (M1)

- Seasonal decomposition.
- Augmented Dickey-Fuller (ADF) test for stationarity.
- ACF/PACF to identify the model order.

### How the EDA Guides Feature Engineering

The EDA determines whether the series requires differencing, which exogenous variables (public holidays, seasonal indicator) provide real signal, and confirms whether the breakdown by department (pending validation in Delivery 3) is stable enough to be modelled separately or whether it is preferable to aggregate it at CCAA level.

The EDA questions for Modules 2 and 3 are maintained in the Annex, without being executed in this deliverable.

---

## 3. Type of Model Proposed

| Module | Alternative | Type | Why it is proposed | Main limitation |
|---|---|---|---|---|
| **M1** | Naive baseline + SARIMA/Prophet | Time-series forecasting | The only MVP module; a 4-8-week horizon is consistent with the weekly granularity of the source | Prophet may over-smooth peaks; SARIMA requires stationarity |

The model approach for M2 and M3 (regression on aggregated rates, with SHAP explainability in M3) is preserved in the Annex as a conceptual design, with the methodological corrections required by the review, but is not implemented in this deliverable.

---

## 4. Input Data for the Analysis and the Model (M1)

`gold_demanda_asistencial.csv` (see Delivery 3, section 5.1).

### Variables Discarded and Rationale

| Variable / period | Reason for exclusion |
|---|---|
| Years 2020–2021 | Pandemic structural break |
| Daily occupancy by centre | Not available in public sources (ESCRI is an annual snapshot, not a series) – hence the horizon remains at weeks, never at 24–72 hours |
| Breakdown by department | Included only if Delivery 3 confirms that the source genuinely supports it; otherwise, the model operates at CCAA level |

---

## 5. Output Data and Mode of Consumption (M1)

Output unchanged from the design already validated: weekly projection of `ingresos_urgentes` by CCAA (and department if confirmed), with confidence interval, consumed in the dashboard of Delivery 5.

---

## 6. Validation and Evaluation Strategy (M1)

| Element | M1 |
|---|---|
| **Data split** | Temporal split (`TimeSeriesSplit`) |
| **Primary metric** | MAE, RMSE by horizon |
| **Reference baseline** | Seasonal naive |
| **Acceptance criterion** | ≥15–20% reduction in MAE vs. baseline (4-week horizon) |

### Explicit Verification of Granularity–Horizon–Source-of-Truth Alignment

This is the point that the review asks to be checked strictly: the source (EMH) has weekly granularity; the prediction horizon is 4–8 weeks; and the source of truth against which the model is evaluated (actual `ingresos_urgentes` for that week) exists at that same granularity. At no point in the design is a 24–72-hour prediction promised – that would require a source with daily updates that is not available and, if proposed in the future, would be a distinct module with its own source and its own validation, not a finer reading of this weekly dataset.

### If the Module Does Not Reach the Acceptance Criterion

The descriptive panel (historical ranking with no added predictive component) is documented as a valid result of the TFM.

---

## 7. Risks and Alternatives (M1)

| Risk | Description | Contingency plan |
|---|---|---|
| **Access to EMH microdata** | Subject to special request, not direct download (see Delivery 3, section 2) | Prioritise already available public aggregates if they sustain the required granularity; document the procedure as a calendar risk if not |
| **Breakdown by department not confirmed** | Delivery 3 may conclude that the EMH does not support that level | Model at CCAA level only, documenting the loss of resolution |
| **Heterogeneity of coding across CCAA** | Noise in the series | Additional normalisation in data preparation |

### Part of the Strategy with the Greatest Uncertainty

Whether the EMH genuinely supports the breakdown by department; this determines whether M1 is delivered at week × CCAA × department or simplified to week × CCAA.

---

## Annex – Conceptual Design of Modules 2 and 3 (Not Implemented in This Deliverable)

> The design work already carried out on M2 and M3 is preserved, incorporating the methodological corrections indicated in the review, so that it remains documented should they be activated in the future (see conditions in Delivery 3, section 10.5). **Nothing in this section is implemented, trained or validated in the current MVP.**

### A.1 Business Questions (EDA Not Executed)

**Module 2:**
- Which pathologies concentrate the highest transfer rate relative to their total discharges? Is it stable over time or does it vary by year?
- Does the transfer rate correlate more with clinical severity (mortality, length-of-stay deviation) or with the structural capacity of the CCAA (total beds)? – important in order not to confuse "clinical complexity" with "lack of beds".
- Are there CCAA with systematically high transfer rates regardless of pathology?

**Module 3:**
- Which pathology × age combinations show the highest `tasa_reingreso_30d` according to iCMBD?
- Does length-of-stay deviation correlate with a higher readmission rate?
- Are there systematic differences between CCAA in the readmission rate for the same pathology and age?

### A.2 Proposed Model Type and Required Methodological Corrections

| Module | Alternative | Correction incorporated following the review |
|---|---|---|
| **M2** | Regression (Ridge/Gradient Boosting Regressor) on `tasa_traslado` | The variable `camas_totales_ccaa` is annual structural context; it must **never** be read or communicated as real bed availability for recommending a specific transfer. The score is a planning prioritisation index, not an operational recommendation |
| **M3** | Regression (Gradient Boosting Regressor) on `tasa_reingreso_30d`, with SHAP | Two mandatory corrections before it can be implemented: (1) if the design were to reach individual-episode level, the split between training and test must be made by **real patient identifier**, not by `episodio_id` – splitting by episode does not prevent several episodes of the same person from appearing in both sets, which artificially inflates the metrics; validation must be group-based (`GroupKFold` or equivalent on patient) and preferably also temporal (leaving out the last year). (2) If the readmission rate were ever completed with synthetic data to gain granularity, there is a risk that the model will merely reproduce the rules used to generate that label, obtaining high metrics with no real clinical validity – therefore, as long as the target variable does not come from a real aggregated indicator (iCMBD), M3 is not trained |

An individual binary classifier is ruled out, as already documented, because it would require a per-patient readmission label that no public source offers without resorting to synthetic data.

### A.3 Input Data (Conceptual)

**M2:** `gold_presion_derivacion.csv`. Target variable: `tasa_traslado`. Predictors: `mortalidad_intrahosp_pct`, `desviacion_estancia`, `camas_totales_ccaa` (normalised by the CCAA's total discharges), `anio`.

**M3:** `gold_riesgo_reingreso.csv`. Target variable: `tasa_reingreso_30d`. Predictors: `desviacion_estancia`, `mortalidad_intrahosp_pct`, `pct_ingreso_urgente`, `grupo_edad` (ordinally encoded), `patologia_cie10` (encoded, possibly grouped).

### A.4 Output Data (Conceptual)

**M2:**

| Field | Description | Type |
|---|---|---|
| `patologia_cie10`, `ccaa_codigo`, `anio` | Predicted unit | str/int |
| `score_presion_derivacion` | Index from 0 to 100; higher value = greater structural transfer pressure (planning, not an individual recommendation) | float |
| `factores_principales` | 2–3 variables with the greatest weight in the score | list[str] |

**M3:**

| Field | Description | Type |
|---|---|---|
| `patologia_cie10`, `grupo_edad`, `ccaa_codigo`, `anio` | Predicted unit | str/int |
| `score_riesgo_reingreso` | Index from 0 to 100; higher value = higher relative readmission risk in that segment | float |
| `factores_principales` | 2–3 variables with the greatest weight in the score | list[str] |

**Intended consumption:** ranking/map by CCAA and pathology, never a recommendation about an identified individual patient. The dashboard would explicitly label these panels as "segment-level prioritisation indices, not an assessment of a specific patient".

### A.5 Validation Strategy (Conceptual, Corrected)

| Element | M2 | M3 |
|---|---|---|
| **Data split** | Leave-one-year-out (cross-validation leaving one year out each time, given the small number of available years) | Leave-one-year-out; **if the design were to reach episode level, additionally group-based validation on a real patient identifier, never by `episodio_id`, preferably combined with temporal separation** |
| **Primary metric** | R² and MAE on `tasa_traslado` | R² and MAE on `tasa_reingreso_30d` |
| **Reference baseline** | Historical mean of `tasa_traslado` by pathology (without CCAA) | Historical mean of `tasa_reingreso_30d` by pathology (without CCAA or age) |
| **Acceptance criterion** | Improvement in R² over the baseline | Same criterion as M2 |

### A.6 Specific Risks of M2/M3 (Conceptual)

| Risk | Description | Contingency plan |
|---|---|---|
| **Circularity with a synthetic label (M3)** | If the readmission rate were completed with synthetic data, the model could reproduce the generation rules of that label instead of demonstrating clinical validity | M3 is not trained while the target variable does not come from a real aggregated indicator (iCMBD); if at any point it were explored with synthetic data for exclusively technical reasons, the result would be presented as **technical validation of the prototype**, never as evidence of real care capacity |
| **Confusion between clinical severity and network capacity (M2)** | The model could attribute to "pathology complexity" what is actually a lack of beds in a specific CCAA | Explicit analysis of variance in the EDA to separate both effects before interpreting the score |
| **Information leakage between episodes of the same patient (M3, if episode level is reached)** | Splitting by `episodio_id` does not prevent two episodes of the same person from falling one in training and one in test, inflating the metric | Group-based validation on a real patient identifier, combined with temporal separation |
| **Real granularity of iCMBD** | The readmission indicator may not break down by pathology and CCAA simultaneously | Simplify to the breakdown the indicator does offer, documenting the loss of resolution |
| **Low volume in pathology × CCAA (× age) combinations** | Unstable rates, risk of overfitting | Minimum threshold of discharges per cell; grouping of infrequent pathologies |

### A.7 Condition for Activating This Design

See Delivery 3, section 10.5: it would be implemented only after confirming the real granularity of DRG/iCMBD and with the MVP (M1) already delivered, applying from the outset the validation corrections (real patient identifier, group-based and temporal validation) set out in A.2 and A.5.
