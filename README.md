# Incrementality-Driven Advertising Targeting

An end-to-end causal machine learning case study on the **Criteo Uplift v2.1 randomized experiment**. The project asks a practical advertising question:

> With a fixed budget, which users should receive an ad because the ad is likely to *change* their behavior—not merely because they were already likely to visit?

The analysis progresses from experiment validation and response prediction to T-Learners, an honest causal forest, heterogeneous treatment-effect analysis, business policy evaluation, and uncertainty-aware model selection.

## Executive summary

- Ad assignment has a positive average intent-to-treat effect: approximately **10,342 incremental visits per million assignments** in the full dataset.
- The causal forest isolates a strong top decile with **7.64 percentage points** observed visit uplift, compared with **0.80 points** in its bottom decile.
- At a 30% targeting budget, learned policies generate about **10.0 incremental visits per 1,000 eligible users**, versus **3.45** for random targeting—roughly **2.9× the incremental value** at the same budget.
- The causal forest does **not** reliably beat a strong response model at that budget: the difference is **+0.06 visits per 1,000**, with a 95% paired-bootstrap interval of **−1.04 to +1.17**.
- Validation therefore selects the simpler **response-score policy** for the primary 30% decision. The causal forest remains valuable for discovering high-uplift segments and as a challenger model.

This is a positive business result and a deliberately honest modeling result: personalized targeting clearly beats untargeted allocation, while additional causal-model complexity has not earned a production win over the strongest baseline.

## Why this problem matters

A response model estimates

$$P(Y=1\mid X),$$

which can prioritize users who would visit even without advertising. The decision target is the conditional average treatment effect (CATE):

$$\tau(X)=E[Y(1)-Y(0)\mid X].$$

That distinction separates **high-propensity users** from **persuadable users**. In advertising systems, it affects incremental value, budget efficiency, experiment design, and how model performance should be evaluated.

## Data and estimand

The project uses the unbiased Criteo Uplift v2.1 release: 13,979,592 rows, 12 anonymized pre-treatment features (`f0`–`f11`), randomized treatment assignment, and binary visit/conversion outcomes. See the [official dataset page](https://ailab.criteo.com/criteo-uplift-prediction-dataset/) and the accompanying [AdKDD 2018 paper](https://www.adkdd.org/papers/a-large-scale-benchmark-for-uplift-modeling/2018).

The primary outcome is `visit`. Treatment is **assignment to advertising**, so estimates are intent-to-treat. The post-assignment `exposure` field is not used as a model feature or substituted for randomized treatment.

The data is not committed to this repository. Download and placement instructions are in [`data/README.md`](data/README.md).

## Notebook walkthrough

| Stage | Notebook | Main question |
|---:|---|---|
| 1 | [`01_eda.ipynb`](notebooks/01_eda.ipynb) | Is the experiment valid, balanced, and causally interpretable? |
| 2 | [`02_prediction_baseline.ipynb`](notebooks/02_prediction_baseline.ipynb) | How well can ordinary models predict visits, and does response targeting create lift? |
| 3 | [`03_t_learner.ipynb`](notebooks/03_t_learner.ipynb) | Can separate treated/control outcome models rank incremental response? |
| 4 | [`04_causal_forest.ipynb`](notebooks/04_causal_forest.ipynb) | Does an honest causal forest improve heterogeneous-effect targeting? |
| 5 | [`05_treatment_effect_heterogeneity.ipynb`](notebooks/05_treatment_effect_heterogeneity.ipynb) | Which reproducible user groups have different advertising effects? |
| 6 | [`06_business_experiment.ipynb`](notebooks/06_business_experiment.ipynb) | Which policy creates the most incremental value under a 30% budget? |
| 7 | [`07_proper_evaluation.ipynb`](notebooks/07_proper_evaluation.ipynb) | How should uplift models be evaluated when individual effects are unobservable? |

Every notebook is self-contained, uses deterministic seeds, streams the large CSV instead of loading it all at once, and includes saved outputs plus methodological notes and limitations.

## Key results

### Fixed 30% targeting policy

| Policy | Incremental visits per 1M eligible users | 95% bootstrap CI |
|---|---:|---:|
| Random targeting | 3,450 | [2,655, 4,152] |
| Visit-response targeting | 10,008 | [7,601, 12,377] |
| Causal-forest targeting | 10,063 | [7,780, 12,025] |

The causal policy beats random by **6,613 incremental visits per million** (95% CI: 4,833 to 8,042). Its estimated advantage over response targeting is only **55 per million** (95% CI: −1,043 to 1,172), so that head-to-head result is a statistical and practical tie.

### Treatment-effect heterogeneity

- The validation-defined causal-forest top decile has **7.642 pp** test uplift (95% CI: 6.016 to 9.269).
- The locked interpretable rule `f3 <= 2.167` covers **8.3%** of test users and has **7.699 pp** uplift (95% CI: 6.267 to 9.132).
- The top-versus-bottom model-defined audience contrast is **6.841 pp** (95% CI: 4.985 to 8.698).
- Because all features are anonymized, these are operational score/rule segments—not demographic personas or proof about any individual's counterfactual response.

See [`docs/results.md`](docs/results.md) for the evaluation table, interpretation, and limitations.

## Evaluation design

- A deterministic approximately one-million-row sample is split 60/20/20, stratified by treatment and visit.
- Models use the same 400,000-row subset of the training split for a compute-fair comparison.
- Validation data selects thresholds, subgroup rules, and the preferred policy.
- Randomized test outcomes estimate GATES, Qini/AUUC, fixed-budget Hájek policy value, and paired-bootstrap uncertainty.
- Standard ROC-AUC and PR-AUC are treated only as factual outcome-model diagnostics—not as evidence that a treatment-effect model is correct.

One governance limitation is important: the same test split was inspected across successive project stages. The results are appropriate for exploratory offline analysis, but a publication or production decision should confirm the frozen policy on a new holdout or in a prospective randomized policy experiment.

## Reproduce the analysis

### 1. Create the environment

The committed environment matches the versions used to execute all notebooks (Python 3.10, NumPy 2.2.6, pandas 2.3.2, scikit-learn 1.7.1, LightGBM 4.6.0, and EconML 0.17.0).

```bash
conda env create -f environment.yml
conda activate causal-ml-advertising
python -m ipykernel install --user \
  --name causal-ml-advertising \
  --display-name "Python (causal-ml-advertising)"
jupyter lab
```

Alternatively, install the tested packages into an existing Python 3.10 environment:

```bash
python -m pip install -r requirements.txt
```

### 2. Add the dataset

Place the uncompressed file here:

```text
data/criteo-uplift-v2.1.csv
```

The notebooks search upward from the working directory for this exact path, so they run correctly from inside `notebooks/`.

### 3. Run in order

Open JupyterLab, select the project environment (the existing `convokit` kernel also works in the original development setup), and execute notebooks 01 through 07. Later notebooks rescan the CSV and refit their models so that each stage remains independently reproducible.

## Repository structure

```text
.
├── README.md
├── environment.yml
├── requirements.txt
├── data/
│   └── README.md
├── docs/
│   └── results.md
└── notebooks/
    ├── 01_eda.ipynb
    ├── 02_prediction_baseline.ipynb
    ├── 03_t_learner.ipynb
    ├── 04_causal_forest.ipynb
    ├── 05_treatment_effect_heterogeneity.ipynb
    ├── 06_business_experiment.ipynb
    └── 07_proper_evaluation.ipynb
```

## Main limitations

1. Treatment is assignment, not guaranteed exposure; all causal claims are intent-to-treat.
2. Anonymized features limit product interpretation, fairness review, and transportability.
3. Individual treatment effects are never directly observed; conclusions concern group-average effects and policy value.
4. Results from one historical experiment may not generalize to another campaign, creative, auction, population, or time period.
5. Model-comparison intervals condition on the fitted scores; full pipeline uncertainty would include repeated training.
