# Delivery 3 – Data Model Design and Gold Layer of the Project

> **Revision note (response to feedback received):** the feedback indicates that several gold tables required levels of granularity that real public sources do not guarantee (weekly demand by department, individual episodes, biomarkers, 30-day readmission); that filling those gaps with synthetic data makes it possible to demonstrate the pipeline but not to sustain clinical conclusions or a real decision-support system; and that the three modules still maintained an excessively broad scope. Accordingly, this document reduces the deliverable to **a single MVP built entirely on real data** (Module 1 – demand forecasting), incorporates an explicit phase of **validation of real fields and frequencies in the EMH and ESCRI** before closing the design of the gold layer, and reclassifies Modules 2 and 3 as **conceptual design, not implemented**, conditional on that validation confirming that a real (non-synthetic) source exists capable of supporting them. The data architecture (layers, format, cleaning principles) is retained because it was not what was called into question; what is adjusted is the product promise, so that it matches the real evidence available.

---

## 1. Summary of the Idea and Project Data

### Problem Addressed

Avoidable readmissions, premature discharges and reactive transfers between centres are costly, structural problems of the Spanish healthcare system, aggravated by unanticipated care pressure. The complete project (long-term vision) aims to address the three facets of the problem – how much demand will arrive, to which centre a patient should be transferred when the network is saturated, and which pathologies/profiles concentrate readmission risk – with real, aggregated evidence. **The TFM (Master's Thesis), however, does not promise all three components at once**: it delivers one of them in full and validated on real data, and documents the remainder as a conceptual design pending source validation.

### Why the MVP Focuses on a Single Module

Selecting a single module responds directly to two problems identified in the review:

1. **Risk of unguaranteed granularity.** Promising three modules requires assuming, without having verified it, that the EMH offers a reconstructible discharge date at weekly level *and* breakdown by department, that ESCRI provides useful bed context, and that iCMBD breaks down readmission by pathology and autonomous community simultaneously. If any of these assumptions is not confirmed, the only way out is to synthesise the data – and synthetic data cannot sustain a clinical conclusion or a real decision-support system, only a demonstration of the pipeline.
2. **Excessive scope for the available time and data.** Three modules, each with its own source validation, exploratory data analysis (EDA), model and dashboard panel, dilute effort and increase the likelihood that none will be fully validated.

The MVP selection criterion is therefore: **the module whose real source is already confirmed at the required granularity, without depending on special access procedures or on indicators whose exact breakdown has yet to be confirmed.**

### Solution to Be Built

| Module | What it would address | Granularity | Primary source | Status in this deliverable |
|---|---|---|---|---|
| **1. Care demand forecasting** | Projection of urgent/scheduled admissions, 4–8 weeks ahead, by autonomous community (CCAA) (and by department if the validation in section 2 confirms it) | Week × (department) × CCAA | EMH, ESCRI | **MVP – in development, real data** |
| **2. Transfer prioritisation index** | Identify which pathology × CCAA combinations concentrate the greatest structural transfer pressure | Pathology (ICD-10, 2 digits) × CCAA × year | EMH, ESCRI, DRG | **Conceptual design – not implemented, real source pending validation** |
| **3. Readmission risk index at discharge** | Stratify which pathology × age group × CCAA combinations concentrate the highest 30-day readmission risk | Pathology × age group × CCAA × year | EMH, DRG, iCMBD | **Conceptual design – not implemented, real source pending validation** |

The design of Modules 2 and 3 is preserved in full in section 10 (formerly sections 5.2/5.3) because the feasibility analysis already carried out has value and may be activated if source validation supports it – but it is not part of what this TFM promises to deliver as a functional system.

### Data Sources and Type of Information Provided (MVP)

| Source | Type of information | Use in the project |
|---|---|---|
| Hospital Morbidity Survey (EMH) – INE, series 2014–2023 | Discharges with actual discharge date, ICD-10 diagnosis, admission modality, age, sex, CCAA | Primary MVP source: reconstruction of the weekly admissions series |
| Statistics of Specialised Care Centres (formerly ESCRI) – Ministry of Health, series 2014–2023 | Annual snapshot of installed/operating beds, technological resources and staff by centre | Static annual structural context for the MVP (network capacity) |

DRG and iCMBD are documented as potential sources for a future extension (section 10), but are not used in the current MVP.

---

## 2. Source Validation: What Granularity They Actually Support

In the previous version, this section was a description of the expected granularity. The feedback calls for something stricter: **to confirm before proceeding, not to assume**, exactly which fields and frequencies exist in the EMH and ESCRI. It is divided into (2.1) the pending validation checklist – the priority task requested – and (2.2) what is already documented about publication periodicity, which remains valid but does not replace field-level validation.

### 2.1 Pending Validation Checklist (priority action before closing the gold design)

Before the MVP gold layer can be considered closed, the following items remain to be confirmed and documented in the project log:

- [ ] Does the EMH microdata allow the actual discharge date to be reconstructed with sufficient precision to aggregate to **epidemiological week**, or is the real temporal granularity available coarser than assumed?
- [ ] Does the EMH break data down by **hospital department**, or is that breakdown not guaranteed, so that the MVP must remain at week × CCAA (without department) until it can be confirmed?
- [ ] Which ESCRI fields are genuinely available at CCAA/year level without significant gaps (bed resources, staff), and which have incomplete coverage by centre or year?
- [ ] Confirm the channel and actual timeframe for access to the EMH microdata (INE User Service Area): if the procedure takes longer than is acceptable for the TFM calendar, document the impact and the alternative (already available public aggregates vs. requested microdata).
- [ ] If the breakdown by department is not confirmed, formally document the reduction of the MVP scope to week × CCAA, recording that this field will not be generated synthetically in order to maintain the original promise.

**Principle governing this validation:** if a field or frequency cannot be confirmed against the real source, the MVP is simplified (less breakdown, shorter horizon, fewer modules) rather than filled with synthetic data. An invented data point is not a lower granularity; it is a different source that cannot sustain the same conclusions.

### 2.2 Known Publication Periodicity (context; does not replace 2.1)

| Source | Publication periodicity | Actual periodicity of the data | Consequence for the project |
|---|---|---|---|
| **EMH (INE)** | Annual (12–18 months' lag) | Microdata with discharge date (precision to be confirmed, see 2.1) | Weekly MVP series, conditional on confirmation of the checklist |
| **ESCRI / SIAE (Ministry of Health)** | Annual | Snapshot at 31 December, not an occupancy series | Static annual structural context; never real-time bed availability |

**Known point of attention:** the EMH microdata page states that, although aggregated results are freely downloadable, individual microdata files are provided only under special conditions through the INE User Service Area, and not by direct download without a procedure. This does not invalidate the MVP (if the required weekly breakdown were also available in public aggregates, that route would be prioritised), but it constrains the calendar if the exact microdata is indispensable; it is documented as a risk in section 9.

---

## 3. Chosen Storage Technology or Format

**Parquet** is retained for the `processed` layer and **CSV** for `raw` and `gold`. No relational database is used: the MVP source is a downloadable file or query with no streaming updates, and the volume (thousands of aggregated rows, not massive microdata) does not require transactionality or concurrent queries.

---

## 4. Data Layer Structure

```
data/
├── raw/          # Original data as obtained from each source
├── processed/    # Clean, typed and partially transformed data
└── gold/         # Final MVP dataset, ready for the model and dashboard
```

| Layer | Expected content | Granularity |
|---|---|---|
| `raw/` | EMH microdata/aggregates by year, annual ESCRI files | One annual delivery per source |
| `processed/` | Clean Parquet files: corrected types, treated nulls, removed duplicates, `snake_case`, ISO 8601 dates | EMH aggregated to week × CCAA (× department if confirmed in 2.1) |
| `gold/` | `gold_demanda_asistencial.csv` – the only MVP dataset | See section 5 |

The conceptual design of Modules 2 and 3 (section 10) does not generate files in `gold/` while it remains inactive; if their source is validated in the future, they will be added as additional datasets without modifying this structure.

---

## 5. Definition of the Gold Layer (MVP)

### 5.1 `gold_demanda_asistencial.csv`

**Granularity:** epidemiological week × CCAA (× department, conditional on confirmation of checklist 2.1).
**Primary key:** `(anio, semana_epidemiologica, ccaa_codigo[, servicio])`.
**Main fields:** `ingresos_urgentes` (target), `ingresos_programados`, `ratio_urgentes_programados`, `indicador_estacional`, `festivo_semana`, `camas_totales_ccaa`, `tendencia_ingresos_4s`.
**Downstream use:** SARIMA/Prophet forecasting; demand dashboard (Deliveries 4 and 5).

**Scope note:** if the validation in section 2.1 does not confirm a breakdown by department, this dataset is delivered at week × CCAA granularity only, without the `servicio` field, with the reason documented. That level of detail is not synthesised.

---

## 6. Data Dictionary (MVP)

| Field | Description | Type | Source | Mandatory | Remarks |
|---|---|---|---|---|---|
| `anio` | Year of the record | int | EMH | Yes | – |
| `semana_epidemiologica` | ISO week of the year | int | EMH | Yes | – |
| `ccaa_codigo` | INE code of the CCAA | str | EMH | Yes | – |
| `servicio` | Hospital department | str | EMH | Conditional | Only if checklist 2.1 confirms the real breakdown |
| `ingresos_urgentes` | Number of urgent admissions in the week | int | EMH | Yes | Target variable |
| `ingresos_programados` | Number of scheduled admissions in the week | int | EMH | Yes | – |
| `camas_totales_ccaa` | Structural capacity of the CCAA (annual context, not real-time occupancy) | int | ESCRI | Yes | – |

The field dictionary for Modules 2 and 3 is kept in section 10, with the same label of "conceptual design, not implemented".

---

## 7. Expected Data Quality Issues

- **Confirmation of real granularity (EMH, ESCRI):** the main risk identified in the review; managed through the checklist in section 2.1 before closing the design, not during implementation.
- **Diagnosis → department mapping:** requires documented mapping rules that may introduce ambiguity for non-clinical codes; applies only if the breakdown by department is confirmed.
- **Access to EMH microdata:** subject to a special request (see section 2.2); calendar risk if the procedure is delayed.
- **2020–2021 structural break:** excluded or treated as an exogenous variable.

---

## 8. Planned Cleaning and Transformation Decisions

### Treatment of Nulls

- Weeks without records are treated as explicit missing values, not as zero, so as not to distort seasonality.
- If a field from checklist 2.1 is not confirmed, it is removed from the design rather than imputed or generated synthetically.

### Derived Variables to Be Built

| Derived variable | Calculation |
|---|---|
| `ratio_urgentes_programados` | `ingresos_urgentes / ingresos_programados` |
| `tendencia_ingresos_4s` | 4-week moving average of `ingresos_urgentes` |

### Data to Be Discarded

- Years 2020–2021.
- Any field whose real granularity is not confirmed in the checklist of section 2.1 (rather than replacing it with an estimate or synthetic data).

---

## 9. Data Model Risks

### Which part is clearest?

That the EMH and ESCRI are real, public sources with a historical series sufficient for a demand MVP; and the principle of not using synthetic data under any circumstances.

### Which part generates the most uncertainty?

Precisely the point raised by the review: whether weekly granularity by department is genuinely guaranteed by the EMH, or whether the project must settle for week × CCAA. This uncertainty is resolved in the validation phase (section 2.1), before building the complete pipeline, not afterwards.

### Alternative if the breakdown by department is not confirmed

The MVP is delivered at week × CCAA granularity, without a `servicio` field, documenting the loss of resolution as a conscious decision and not as a hidden defect.

### Alternative if access to EMH microdata is delayed

Prioritise the already available public aggregates (if they sustain the required granularity) over the microdata request, documenting any loss of resolution this entails.

---

## 10. Annex – Conceptual Design of Modules 2 and 3 (Not Implemented in This Deliverable)

> Everything that follows is the design work already carried out on Modules 2 (transfer pressure) and 3 (readmission risk). It is retained because it has analytical value, but it is **not part of the delivered MVP**: its gold datasets are not built, its models are not trained and it does not appear in the dashboard of this deliverable. It would be activated only if, in a future phase, it is confirmed that DRG and iCMBD offer the required real granularity – never by completing that granularity with synthetic data.

### 10.1 Approach

| Module | What it would address | Granularity | Primary source |
|---|---|---|---|
| **2. Transfer prioritisation index** | Identify which pathology × CCAA combinations concentrate the greatest structural transfer pressure, to support network planning decisions (not the individual transfer of a patient in real time) | Pathology (ICD-10, 2 digits) × CCAA × year | EMH, ESCRI, DRG |
| **3. Readmission risk index at discharge** | Stratify which pathology × age group × CCAA combinations concentrate the highest 30-day readmission risk, to support the review of discharge protocols in those segments | Pathology × age group × CCAA × year | EMH, DRG, iCMBD |

Neither would produce a recommendation for a specific patient at a specific moment (that would require real-time bed occupancy and individual traceability between admissions, which no public source offers). They would instead produce a risk/prioritisation index per segment (pathology × CCAA × age).

### 10.2 Additional Sources They Would Require

| Source | Type of information | Intended use |
|---|---|---|
| Diagnosis-Related Groups (DRG) – SNS | Average length of stay and reference cost by diagnosis | Length-of-stay deviation variable (severity/complexity) |
| iCMBD (indicators and analysis axes of the CMBD, Minimum Basic Data Set) – Ministry of Health | Official aggregated indicators: readmission rate, mortality, average length of stay, by CCAA/hospital/diagnosis | Real target variable of M3, if its breakdown reaches pathology × CCAA |

**Open point of attention, unresolved:** it remains to be confirmed whether iCMBD breaks down the readmission rate by pathology × CCAA simultaneously, or only along one of the two axes, and whether access is through interactive query or bulk download. Until this is confirmed, M3 does not progress beyond conceptual design.

### 10.3 Proposed Gold Datasets (Conceptual)

**`gold_presion_derivacion.csv` (M2)** – key `(anio, ccaa_codigo, patologia_cie10)`:

| Field | Type | Description | Source |
|---|---|---|---|
| `anio` | int | Year of the record | EMH |
| `ccaa_codigo` | str | INE code of the CCAA | EMH |
| `patologia_cie10` | str | Diagnostic group (ICD-10, 2 digits) | EMH |
| `altas_totales` | int | Number of discharges for that pathology/CCAA/year | EMH |
| `traslados_totales` | int | Number of transfers to another centre | EMH |
| `tasa_traslado` | float | `traslados_totales / altas_totales` | Derived |
| `mortalidad_intrahosp_pct` | float | % of discharges due to death | EMH |
| `estancia_media_dias` | float | Actual average length of stay | EMH |
| `estancia_media_grd_esperada` | float | Expected length of stay according to DRG | DRG |
| `desviacion_estancia` | float | `estancia_media_dias - estancia_media_grd_esperada` | Derived |
| `camas_totales_ccaa` | int | Structural capacity of the CCAA – **annual context, never real bed availability for deciding a transfer** | ESCRI |
| `score_presion_derivacion` | float | Output variable: combined index | Derived (model) |

**`gold_riesgo_reingreso.csv` (M3)** – key `(anio, ccaa_codigo, patologia_cie10, grupo_edad)`:

| Field | Type | Description | Source |
|---|---|---|---|
| `anio` | int | Year of the record | EMH / iCMBD |
| `ccaa_codigo` | str | INE code of the CCAA | EMH |
| `patologia_cie10` | str | Diagnostic group | EMH |
| `grupo_edad` | str | Ten-year age band | EMH |
| `altas_totales` | int | Number of discharges in the segment | EMH |
| `tasa_reingreso_30d` | float | Target variable: official 30-day readmission rate – **must come from an already aggregated, real indicator (iCMBD); if that granularity is not confirmed, the module cannot be sustained with synthetic data** | iCMBD |
| `estancia_media_dias` | float | Actual average length of stay of the segment | EMH |
| `desviacion_estancia` | float | Actual stay vs. expected DRG stay | DRG (derived) |
| `mortalidad_intrahosp_pct` | float | % of discharges due to death in the segment | EMH |
| `pct_ingreso_urgente` | float | % of urgent admissions in the segment | EMH |
| `camas_totales_ccaa` | int | Annual structural context | ESCRI |
| `score_riesgo_reingreso` | float | Output variable: risk predicted/explained by the model | Derived (model) |

### 10.4 Why the Real-Time / Individual Version Is Ruled Out (If They Are Ever Activated)

| Element a real-time/individual system would require | Why it is not feasible with public sources | Solution that would be adopted |
|---|---|---|
| Bed occupancy by centre, updated several times a day | ESCRI is an annual snapshot, not an occupancy series | Annual structural capacity as context, never as a real-time availability variable |
| Patient identifier to link admission and readmission | The EMH's statistical unit is the discharge, not the patient; there is no public traceability between discharges of the same person | Use the `tasa_reingreso_30d` from iCMBD, already calculated by the Ministry with internal traceability that the public does not have; if individual-episode granularity were ever explored, a real patient identifier – not an `episodio_id` – would be indispensable |
| Individual biomarkers and vital signs at the time of discharge | Not published at patient level in any public source | Not used; they would be replaced by aggregated severity proxies (mortality, length-of-stay deviation) at segment level |

### 10.5 Condition for Activating This Design

Modules 2 and 3 would move from "conceptual design" to "in development" only if: (a) it is confirmed that DRG and iCMBD offer the required real granularity, following the same type of checklist as in section 2.1; and (b) the MVP (Module 1) has already been delivered and validated, so as not to repeat the scope problem identified in the review. Under no circumstances would a granularity gap be filled with synthetic data in order to force their activation.
