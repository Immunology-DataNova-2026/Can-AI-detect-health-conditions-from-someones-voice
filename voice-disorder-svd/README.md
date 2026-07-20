# Can a model detect an organic voice disorder from sustained vowels?

SVM-RBF on eGeMAPS v02, 1,119 speakers, GroupKFold(5) by speaker, seed 42.

| Level | Accuracy | AUC | 95% CI (AUC) |
|---|---|---|---|
| **Per-speaker** (headline) | 0.804 | **0.866** | [0.844, 0.888] |
| Per-record | 0.710 | 0.764 | — |

Per-speaker averages the predicted probability across a speaker's ~14 vowel
recordings, which is where most of the gain comes from — 0.764 to 0.866.
Sensitivity 0.627, specificity 0.924 — confusion `[[618, 51], [168, 282]]` in
`[[TN, FP], [FN, TP]]` order, positive = pathological.

At the default 0.5 threshold this model is **conservative**: when it flags a
speaker it is usually right, but it misses 168 of 450 pathological speakers.
That is the opposite failure mode to `../parkinsons-sakar/`, and the threshold
would need moving before any screening use.

Other models, per-speaker AUC: Random Forest 0.842 · Logistic Regression 0.823.
Full precision in `models/model_card.json`.
