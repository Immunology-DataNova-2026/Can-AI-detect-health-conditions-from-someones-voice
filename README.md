# Can AI predict whether someone is currently sick from their voice?

| Experiment | Accuracy | AUC | Evaluated at |
|---|---|---|---|
| acute-illness-cross-corpus | 0.53 | 0.54 | cross-dataset (leave-one-dataset-out) |
| voice-disorder-voiced | 0.63 | 0.65 | per-record (corpus has no repeat speakers to group) |
| parkinsons-oxford-uci | 0.72 | 0.71 | per-record, split by subject |
| parkinsons-sakar | 0.821 | 0.850 | per-speaker |
| voice-disorder-svd | 0.804 | 0.866 | per-speaker |

The last column matters. These rows are not all the same kind of number: the two
strong results aggregate each speaker's recordings into one decision, while the
others are scored per recording. Quoting them side by side without saying so is
what made the model cards disagree with the manuscript in the first place — see
`CONTROLS_RESULTS.md`.

Full-precision values, confusion matrices, and bootstrap CIs live in each task's
`models/model_card.json`.
