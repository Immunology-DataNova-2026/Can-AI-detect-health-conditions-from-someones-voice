# Discrepancies between committed artifacts and re-runs

Every row below came from an executed re-run using the committed configuration —
same seeds (42 voice/gene, 0 antibody), same hyperparameters, same CV scheme.
Nothing was retrained, retuned, or reseeded. Where a re-run disagrees with a
committed value, the disagreement is recorded rather than corrected in place.

| Artifact | Committed | Re-run | Difference | Likely cause |
|---|---|---|---|---|
| `parkinsons-sakar/models/model_card.json` → `honest_auc_speaker_independent` | 0.83 | 0.8386758066903492 per-record / **0.8499418218085106 per-speaker** | +0.0087 vs per-record, **+0.0199 vs per-speaker** | rounding **and** aggregation level — see note 1 |
| `voice-disorder-svd/models/model_card.json` → `honest_auc_speaker_independent` | 0.86 | **0.8662813486131873 per-speaker** (0.7644593326894301 per-record) | +0.0063 | rounding to 2 dp — see note 2 |
| both voice model cards → accuracy | *field absent* | SVD 0.8042895442359249 per-speaker; Sakar 0.8214285714285714 per-speaker | n/a | the cards never carried an accuracy field, so the READMEs' "0.80" has no committed artifact behind it — see note 3 |

## Note 1 — Sakar is the substantive one
The card's 0.83 does not match *either* re-run level. It is closest to the
README's 0.827, which is a **per-record** figure, while the manuscript quotes the
**per-speaker** number. So two things are stacked: the value is rounded to 2 dp,
and it is recorded at a different aggregation level than the text uses.

This matters beyond cosmetics because the bootstrap CI is computed on the
per-speaker metric: reporting "AUC 0.83, 95% CI [0.796, 0.901]" pairs a
per-record point estimate with a per-speaker interval. The point estimate sits
almost on the interval's lower bound, which reads as a much weaker result than
the data supports. Task 5 updates the card to carry both levels explicitly.

## Note 2 — SVD is only rounding, but rounds the wrong way
0.8662813486131873 rounds to 0.87, not 0.86, so the committed 0.86 appears to be
a floor/truncation rather than a round. The effect is small and conservative
(it understates the result), but it is the reason `matches_model_card` is
`false` for SVD as well as Sakar at a ±0.005 tolerance.

## Note 3 — a gap the paper currently admits in print
Neither voice model card recorded a confusion matrix, which is why the manuscript
states it cannot give sensitivity/specificity for SVD or Sakar. Those matrices
are now in `metrics_full_svd.json` / `metrics_full_sakar.json`, and the derived
operating points are:

| Task (per-speaker) | Sensitivity | Specificity | Confusion `[[TN,FP],[FN,TP]]` |
|---|---|---|---|
| Voice disorder (SVD) | 0.6266666666666667 | 0.9237668161434978 | `[[618, 51], [168, 282]]` |
| Parkinson's (Sakar) | 0.9148936170212766 | 0.546875 | `[[35, 29], [16, 172]]` |

Worth stating plainly in the paper: the two models sit at **opposite operating
points** at the default 0.5 threshold. SVD is conservative (specificity 0.92,
sensitivity 0.63 — it misses 168 of 450 pathological speakers). Sakar is the
reverse (sensitivity 0.91, specificity 0.55 — it flags 29 of 64 healthy speakers).
Neither is wrong, but "AUC 0.85–0.87" alone conceals that one model under-calls
disease and the other over-calls it, and for a screening claim the operating
point is the part that matters.

## Reproducibility check (not a discrepancy)
Both voice tasks reproduced **exactly** across two independent runs in this
session — SVD per-speaker AUC 0.8662813486131873 and Sakar 0.8499418218085106
matched the earlier controls run to all printed digits. The pipelines are
deterministic under the committed seeds; the disagreements above are with the
*stored* values, not between runs.

## Items checked and found consistent
- immuno `metrics.json` (internal AUC 0.9322344322344323, external 0.8106060606060606) matched the re-run to full precision.
- immuno `results/curves.json` AUC/AP values matched `metrics.json`.
- antibody `test_eval_results.csv` per-antigen rows were not recomputed row-by-row; the pooled test curve in `curves_all.json` is a fresh inference pass over the same checkpoint. Pooled AUROC is reported there for comparison against the per-antigen table.
