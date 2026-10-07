# Delivery 2 – Project Idea Selection and Analysis of Required Data

---

## 1. Selected Idea

**Intelligent Scoring System for the Evaluation of Hospital Discharge and Transfer Criteria**

Premature discharges and avoidable readmissions are among the most costly and persistent problems of the Spanish National Health System (SNS). When a hospital operates under high care pressure, the decision to discharge a patient is conditioned by the urgent need to free up beds, which may lead to unnecessary readmissions with a poorer clinical prognosis and a higher cost to the system. According to INE 2023 data, between 20% and 25% of 30-day readmissions in high-prevalence chronic conditions such as heart failure are potentially avoidable, representing estimated savings of more than €55 million per year in that diagnostic category alone. In parallel, patient transfers between hospitals are frequently carried out reactively: in 2023, 283,792 unplanned transfers were recorded, accounting for 5.83% of all discharges, many of them under emergency conditions that increase clinical risk for the patient and double operating costs. Both problems share a common root: the absence of data-driven tools that support these decisions in an objective, consistent and anticipatory manner.

The project proposes the development of a clinical decision-support system based on Data Science techniques, with two main components. The first is a supervised classification model that estimates each patient's 30-day readmission risk at the point immediately preceding discharge, based on the patient's clinical profile, principal diagnosis, comorbidities, vital signs and operational variables of the episode. The second is a multicriteria hospital transfer algorithm that, in situations of high occupancy, identifies the most suitable receiving centre within the healthcare network by assessing the availability of specialised beds, technological compatibility, estimated transfer time and the patient's diagnostic profile. Both components are fed by anonymised public SNS data and by a synthetic dataset generated from the statistical distributions observed in official sources, following a documented and reproducible methodology. The system does not seek to replace the clinical judgement of healthcare professionals, but to provide a data-based support layer that enables better-founded, more consistent and safer decisions.

The minimum viable product (MVP) to be presented at the end of the course will consist of an interactive dashboard with three integrated functional elements. First, an exploratory and visual analysis of historical SNS data (2014–2023), segmented by pathology, autonomous community and temporal evolution of key indicators such as occupancy, readmissions and transfers. Second, an XGBoost classification model trained on a synthetic dataset that assigns each simulated patient a readmission risk level (high, moderate, low), accompanied by SHAP-based explainability to identify the most determining clinical factors in each case. Third, a simulation of the intelligent transfer module which, given a hospital occupancy scenario, calculates and visualises the score of each candidate centre in the network and proposes the optimal transfer. The MVP will be built entirely on public and synthetic data, without access to identifiable individual clinical data.

---

## 2. Required Data

### Variables or Fields Required

The project requires data organised around four categories, consistent with those identified in the reference clinical literature on readmission prediction and hospital transfer.

**Demographic and administrative variables:** age group in ten-year bands, sex, autonomous community of residence, admission type (urgent or scheduled), admitting department, assigned DRG (Diagnosis-Related Group) and epidemiological week of admission.

**Clinical variables of the episode:** principal diagnosis coded in ICD-10, secondary diagnoses and relevant comorbidities (Charlson index as a derived variable), procedures performed during admission, and key laboratory biomarker results by pathology (troponin in ischaemic heart disease, PCT and lactate in sepsis, BNP in heart failure, CRP in infectious processes).

**Care-process variables:** length of stay in days, difference between actual length of stay and the average stay of the corresponding DRG, number of previous admissions in the last 12 months, time elapsed since the last admission, and pending diagnostic issues at the time of discharge.

**Operational and network variables:** occupancy rate of the department at the time of discharge or admission, level of emergency pressure over the previous 24 hours, occupancy of the centres in the health-area network, and projected demand trend over a 24- and 72-hour horizon.

### Level of Granularity

For the exploratory analysis, the appropriate granularity is at the pathology level (ICD-10 at two digits), broken down by autonomous community and with annual periodicity. For the supervised classification model, individual records from the synthetic dataset will be used, generated from the statistical distributions of the public sources, with one observation per hospitalisation episode.

### Required Historical Depth

A 10-year time series (2014–2023) is necessary to capture seasonal patterns, demand trends by pathology and the impact of extraordinary events such as the pandemic. The 2020–2021 period will be treated as an exogenous variable or excluded from training depending on the initial exploratory analysis, given its atypical nature.

### Approximate Data Volume

The public datasets of the INE and the Ministry of Health contain several thousand aggregated records by pathology, autonomous community and year, sufficient for the exploratory analysis. For the supervised model, a synthetic dataset of between 50,000 and 100,000 records will be generated from the observed statistical distributions, a volume sufficient to train and validate classification models with statistical guarantees.

### Essential versus Desirable Data

**Essential:** principal diagnosis by ICD-10, discharges by pathology and admission modality (urgent/scheduled), average length of stay by DRG, number of readmissions by pathology, number of transfers between centres, hospital occupancy indicators by department and period, and age group and sex as segmentation variables.

**Desirable but not essential:** individual vital signs, laboratory biomarker results and pending diagnostic issues at discharge. These variables, which are not available in public sources at individual level, will be incorporated into the synthetic dataset on the basis of the statistical distributions documented in the reference clinical literature.

---

## 3. Planned Data Sources

### Primary Source 1: Hospital Morbidity Survey (EMH) – INE

- **URL:** https://www.ine.es/dyngs/INEbase/es/operacion.htm?c=Estadistica_C&cid=1254736176778&menu=resultados
- **Access type:** Open and public, with no download restrictions.
- **Format:** CSV and Excel, directly downloadable.
- **Available history:** Complete series 2014–2023.
- **Stability:** Official INE source, annual publication, with high stability and guaranteed maintenance.
- **Main content:** Hospital discharges by principal diagnosis (ICD-10), admission modality (urgent/scheduled), average length of stay, readmissions, transfers between centres and in-hospital mortality, broken down by age, sex and autonomous community. This source has already been worked on and processed previously, so the data are available in downloaded and transformed form.

### Primary Source 2: Statistics of Health Establishments with Inpatient Care (ESCRI) – Ministry of Health

- **URL:** https://www.sanidad.gob.es/estadEstudios/estadisticas/estHospiInternado/inforRecopilaciones/home.htm
- **Access type:** Open and public.
- **Format:** Excel.
- **Available history:** Series 2014–2023.
- **Stability:** Official Ministry of Health source, high stability.
- **Relevant content:** Hospital occupancy indicators, bed turnover rate, emergency pressure, and technological resources by centre and autonomous community. It allows the construction of the operational network variables required for the transfer module. This source has also been worked on and processed previously.

### Complementary Source: Diagnosis-Related Groups (DRG) – Ministry of Health

- **URL:** https://www.sanidad.gob.es/estadEstudios/estadisticas/cmbd.do
- **Access type:** Open and public.
- **Format:** Excel and CSV.
- **Relevant content:** Expected average length of stay by DRG and average cost per episode. It allows the construction of the variable measuring the deviation between actual and expected average stay, one of the most relevant predictors of premature-discharge risk identified in the clinical literature.

### Identified Risks

The main risk is the **limited granularity** of public data: the available sources offer data aggregated by pathology, autonomous community and year, but not individual patient records. This prevents the supervised classification model from being trained directly on real data. The mitigation strategy is the **generation of a synthetic dataset** built from the statistical distributions observed in the public sources, a technique widely used and validated in the literature on machine learning applied to healthcare.

A second risk is the **discontinuity in the time series** caused by the 2020–2021 period, which will be managed as an exogenous variable or through controlled exclusion of the atypical period.

A third, minor risk is the **heterogeneity among autonomous communities** in coding and reporting criteria, which may require a more intensive normalisation process than expected during the data preparation phase.

---

## 4. Privacy and Data Protection Considerations

The project works exclusively with aggregated public data, anonymised at source and published by official bodies (INE and the Ministry of Health). None of the planned sources contains personally identifiable information: EMH and ESCRI data are published at the level of pathology, autonomous community and period, without any attribute that would allow individual patients to be identified.

With regard to the synthetic dataset generated for training the supervised model, the data are entirely artificial and built from documented statistical distributions; therefore, there is no risk of re-identification or of processing personal data in any case.

The project does not require access to clinical records, hospital information systems (HIS/LIS), data on specific patients or any information subject to the General Data Protection Regulation (GDPR) or to the specific health regulations on the protection of clinical data (Law 41/2002 on patient autonomy). Consequently, it is not necessary to establish anonymisation protocols, obtain consents or request special permissions.

From an ethical standpoint, the readmission-risk scoring model will be explicitly evaluated across subgroups defined by age, sex and autonomous community in order to detect and correct potential systematic biases in protected variables, as part of the model validation process.

---

## 5. Initial Feasibility of the Project

**Is it feasible to obtain the required data?** Yes, with a high degree of certainty. The two primary sources are public, stable and accessible without restrictions, and have already been downloaded and processed. There is no access barrier, payment or permission requirement that could compromise data acquisition.

**Does the information have sufficient quality, granularity and historical depth?** Sufficient for the exploratory analysis and for modelling at aggregate level. The individual-level granularity required for the supervised model is addressed through the generation of synthetic data, a methodologically sound strategy appropriate for an academic project of this nature, with direct precedent in the literature on machine learning applied to healthcare.

**Can the idea be developed realistically during the course?** Yes. The scope is well bounded into three specific components: exploratory analysis of public SNS data 2014–2023, construction and training of the XGBoost classification model with SHAP explainability, and simulation of the multicriteria transfer algorithm. These tasks are fully achievable with the tools and time available in the course.

**Which part of the project is the riskiest?** The construction of the synthetic dataset with sufficient clinical realism for the classification model to be representative. To mitigate this risk, the design of the distributions will rely on the statistical patterns observed in the INE's EMH and on the reference clinical literature on hospital readmission prediction, following a documented and reproducible methodology.

**What alternative exists if the primary source fails?** If the EMH or the ESCRI present unexpected problems of access or quality, the immediate alternative is to use the hospital activity datasets of the NHS (United Kingdom), openly available from NHS Digital with a similar structure and greater individual-level granularity, which could be used as an alternative training source while maintaining the same methodological approach.
