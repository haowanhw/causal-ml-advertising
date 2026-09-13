# Notebook guide

Run the notebooks in numeric order for the complete project narrative. Each notebook is also self-contained: it recreates the deterministic sample, split, and models needed for that stage.

| Notebook | Focus | What to pay attention to |
|---|---|---|
| `01_eda.ipynb` | Experiment understanding | Randomization, ITT estimand, assignment versus exposure |
| `02_prediction_baseline.ipynb` | Response prediction | Rare-outcome metrics, calibration, response versus incrementality |
| `03_t_learner.ipynb` | First CATE model | Two response surfaces, GATES, policy value |
| `04_causal_forest.ipynb` | Causal forest | Honesty, cross-fitting, uncertainty, fair baselines |
| `05_treatment_effect_heterogeneity.ipynb` | Segment discovery | Discovery/confirmation separation, multiplicity, interpretable rule |
| `06_business_experiment.ipynb` | Decision policy | Fixed budget, paired uncertainty, economics, prospective test design |
| `07_proper_evaluation.ipynb` | Uplift evaluation | Qini/AUUC, Hájek versus IPW, model selection discipline |

Saved cell outputs are intentional: they let a reviewer inspect the full analysis on GitHub without downloading the 3 GiB dataset or rerunning expensive models.
