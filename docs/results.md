# Results and interpretation

## Experimental foundation

The unbiased Criteo v2.1 release contains 13,979,592 users with approximately 85% assigned to treatment. Stage 1 streams the full file, checks allocation and covariate balance, and estimates an average visit intent-to-treat effect of approximately 1.034 percentage points—about 10,342 incremental visits per million assignments.

For Stages 2–7, a deterministic approximately one-million-row sample is split into training, validation, and test sets. The test split contains 199,891 users and has a design-based visit ATE of **1.150 pp** (95% CI: 0.913 to 1.387).

## Targeting results

### At a 10% budget

| Ranking | Incremental visits per 1,000 eligible users |
|---|---:|
| Causal Forest | 7.64 |
| T-Learner | 7.20 |
| Response score | 6.43 |
| Random | 1.15 |

The forest finds the strongest top group, but paired intervals do not establish that it reliably beats the learned baselines.

### At a 30% budget

| Ranking | Incremental visits per 1,000 eligible users |
|---|---:|
| Causal Forest | 10.06 |
| Response score | 10.01 |
| T-Learner | 9.92 |
| Random | 3.45 |

Paired comparisons at 30%:

| Difference | Point estimate per 1,000 | 95% paired-bootstrap CI |
|---|---:|---:|
| Forest − Response | +0.06 | [−1.04, +1.17] |
| Forest − T-Learner | +0.14 | [−1.06, +1.22] |

The correct interpretation is not that the causal forest failed. All learned policies allocate the budget far better than random. The extra complexity simply does not show a reliable incremental gain over the strongest response baseline at the chosen budget.

## Heterogeneity results

| Audience | Share | Observed visit uplift | 95% CI |
|---|---:|---:|---:|
| Forest top decile | 10.0% | 7.642 pp | [6.016, 9.269] |
| Forest bottom decile | 10.0% | 0.801 pp | [−0.093, 1.696] |
| Locked rule `f3 <= 2.167` | 8.3% | 7.699 pp | [6.267, 9.132] |

The validation-defined top-versus-bottom audience difference is **6.841 pp** (95% CI: 4.985 to 8.698). The strongest feature-bin contrast is associated with `f6`: **6.939 pp** (95% CI: 5.750 to 8.127) after Holm multiplicity correction.

These are group-average differences supported by randomized outcomes. They do not reveal any user's unobserved counterfactual, and anonymization prevents meaningful labels such as age, location, or purchase intent.

## Proper model selection

Validation selects **Response score** both by Qini and by policy value at the 30% budget. That choice must remain primary after looking at test results; switching to the forest because its test point estimate is marginally higher would leak test information into model selection.

Test Qini above random, scaled by 1,000:

| Ranking | Qini × 1,000 |
|---|---:|
| Causal Forest | 4.99 |
| Response score | 4.90 |
| T-Learner | 4.52 |

The forest-minus-response Qini difference is **+0.09** (95% paired-bootstrap CI: −0.78 to +0.96), again a tie.

## Business decision

At a fixed 30% budget, use the validated response policy as the primary candidate because it is simpler and statistically tied with the causal forest. Keep the causal forest as:

1. a challenger policy;
2. a tool for high-uplift segment discovery; and
3. a way to generate hypotheses for prospective experiments.

Before production, freeze model versions, eligibility, scores, and thresholds, then randomize eligible users between complete policy engines. Measure incremental visits, conversions, spend, and guardrails by assigned arm.

## Claims the evidence supports

- Advertising assignment has a positive average effect on visits in this experiment.
- Learned targeting substantially improves incremental visit yield over random allocation at the same budget.
- Some model-defined groups have reproducibly larger average effects than others.
- The causal forest and response policy are statistically indistinguishable at the 30% operating point.

## Claims the evidence does not support

- Knowing the true treatment effect for an individual user.
- Claiming that users with negative point predictions are harmed by advertising.
- Treating anonymized feature importance as a causal explanation.
- Assuming these effects transport unchanged to another campaign, creative, auction, or time period.
- Claiming a production-grade untouched test set after repeatedly inspecting the same holdout across project stages.
