# Can a model detect Parkinson's disease from a sustained vowel?

Random Forest, 252 speakers, GroupKFold(5) by speaker id, seed 42.

| Level | Accuracy | AUC | 95% CI (AUC) |
|---|---|---|---|
| **Per-speaker** (headline) | 0.821 | **0.850** | [0.796, 0.901] |
| Per-record | 0.816 | 0.839 | — |

Per-speaker averages the predicted probability across a speaker's recordings.
Sensitivity 0.915, specificity 0.547 — confusion `[[35, 29], [16, 172]]` in
`[[TN, FP], [FN, TP]]` order, positive = Parkinson's.

At the default 0.5 threshold this model **over-calls** disease: it catches 172 of
188 Parkinson's speakers but flags 29 of 64 healthy ones. Reasonable for triage,
not for confirmation.

Other models, per-record AUC: SVM-RBF 0.804 · Logistic Regression 0.803.
Full precision in `models/model_card.json`.
