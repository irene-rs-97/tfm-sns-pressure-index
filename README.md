# Structural Healthcare Pressure Index for the Spanish National Health System (SNS)
**Master’s Thesis · Master in Data Science & AI (Evolve)**  
**Author:** Irene Rodríguez Sánchez · Academic Year 2025–2026  

**Structural pressure index for the Spanish National Health System (SNS), developed using public datasets from INE and the Ministry of Health. Includes ETL pipelines, data modeling, and an interactive dashboard for healthcare capacity planning.**

---

## The Problem

Regional medical directorates and bed management units within the SNS currently compare healthcare pressure across autonomous communities and pathologies in a reactive manner, without a unified panel grounded in real data. This project validates eight public datasets from INE and the Ministry of Health and confirms a key finding that shapes the entire design: **none of the sources simultaneously link diagnosis, territory, and time**.

Based on this limitation, the scope is narrowed to a single MVP:  
**a structural pressure index by pathology × autonomous community for 2023**,  
complemented by a descriptive national trend block covering **2014–2023**.

The project follows the **CRISP-DM** methodology, with a strong emphasis on methodological honesty: every limitation of the sources is documented as a result rather than minimized, and the final evaluation relies on **group-based cross-validation** (instead of simple K-Fold) precisely to avoid overstating what the model can demonstrate.

---

## Main Result

| | Simple K-Fold | GroupKFold by pathology | **GroupKFold by autonomous community** |
|---|---|---|---|
| Model improvement over baseline | +8.6% | +5.4% (negative R²) | **+0.7%** |

The index separates **pathologies** robustly (driven by their national clinical profile), but the correct validation —forcing the model to generalize to unseen autonomous communities— shows that the **territorial** contribution is real but much more modest than suggested by naïve validation.  
This correction, not the optimistic initial figure, is the central result of the evaluation.

---

## Repository Structure
├── README.md
└── docs/
├── assets/
│   └── 05_mockup_frontal.png       # Front-end mockup used in the final presentation
├── entregas/                       # Project evolution throughout the course
│   ├── 01_ideas_producto.md        # Delivery 1 - Initial product ideas
│   ├── 02_datos_necesarios.md      # Delivery 2 - Selected idea and required data
│   ├── 03_modelo_datos.md          # Delivery 3 - Data model design and gold layer
│   ├── 04_analisis_modelado.md     # Delivery 4 - Analysis and modeling strategy
│   └── 05_diseno_frontal.md        # Delivery 5 - Front-end and UX design
└── notebooks/                      # Google Colab notebooks
├── 1_TFM_Evolve
├── 1_TFM_Evolve_punto_2.5.ipynb
└── 2_TFM_Evolve_punto_2.6.ipynb


> Deliveries 3, 4, and 5 reflect the project’s evolution during the course (including corrections applied after feedback in each phase) and are already updated to represent the final design: a single MVP built entirely on real data, without modules dependent on synthetic data.

---

## Data Sources

Eight open public files, unrestricted, corresponding to the 2023 cycle except for two national historical series (2014–2023):

| File | Institution | Content |
|---|---|---|
| `Alta_Diagnostico_Provincia.xlsx` | INE - Hospital Morbidity Survey | Discharges by diagnosis × autonomous community/province |
| `Alta_Motivo_ingreso.xlsx` | INE - EMH | Transfers and deaths by diagnosis (national) |
| `Alta_Urgencia_ingreso.xlsx` | INE - EMH | % urgent admissions by diagnosis (national) |
| `Estancia_media.xlsx` | INE - EMH | Average length of stay by diagnosis (national) |
| `Altas_Diagnostico_principal.xlsx` | INE - EMH | Discharges by diagnosis × age (not used in gold layer) |
| `Tablas_CCAA_2023.xlsx` | Ministry of Health - ESCRI | Hospitals, beds, staff, emergencies by autonomous community |
| `Actividad_Evolucion_2014-2023.xlsx` | Ministry of Health - ESCRI | National hospital activity time series |
| `Dotacion_Evolucion_2014-2023.xlsx` | Ministry of Health - ESCRI | National bed capacity time series |

---

## Demo

The front-end mockup (`docs/assets/05_mockup_frontal.png`) corresponds to the project’s final presentation.  
The interactive dashboard (self-contained HTML, no backend) that consumes the precomputed index will be added to the repository soon.

---

## Methodological Principles

- **No synthetic data.** The entire project is built and validated using 100% real public sources; modules requiring synthetic data (e.g., readmission risk, multi-year referral pressure) are discarded and documented as conceptual designs not implemented.
- **Source limitations treated as results.** The inability to cross diagnosis + territory + time in a single dataset determines the project scope from the data understanding phase.
- **Honest validation over favorable validation.** The final evaluation uses GroupKFold by autonomous community because it is the scheme capable of disproving the most optimistic result, not the one that best confirms it.

---

## Main Limitations

- The index does not generalize strongly to unseen autonomous communities (+0.7% improvement under correct validation).
- The national trend block is descriptive: 10 annual points do not allow statistically reliable forecasting.
- No causal relationship is established between structural variables and healthcare pressure—only association.
- No 30-day readmission or multi-year referral pressure: no public source offers that granularity without synthetic data.

---

## Target Audience

Regional medical directorates, department heads, and bed management units within the SNS, as a support tool for **structural planning** — never for individual clinical decision-making on specific patients.

---

## Pending Uploads

- [ ] Full TFM report (PDF)  
- [ ] Remaining Colab notebooks  
- [ ] Interactive dashboard (`.html`) translated into English


