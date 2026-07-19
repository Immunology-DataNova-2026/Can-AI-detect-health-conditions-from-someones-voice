# Control results — 2026-07-19

Scope note: this pass ran from the voice repo
(`Can-AI-predict-whether-someone-is-currently-sick-from-their-voice`), which is
the only one of the three DataNova repos present on this machine plus the
antibody repo (present but not yet set up — see "Did not run"). The
immunosenescence/gene-expression repo is not present at all, so Task 1 could
not be attempted.

## Table 7 (ablation) — rows to fill
| Safeguard | Study | With | Without | Effect |
|---|---|---|---|---|
| Filter refit inside folds | Gene | NOT RUN | NOT RUN | gene-expression repo not present on this machine |
| Antigen holdout | Antibody | NOT RUN | NOT RUN | see "Did not run" — needs env setup + data download + GPU time not yet spent |

## Random-gene control (Study 3)
NOT RUN — requires `controls_immuno.py` against the gene-expression repo's `pipeline.py`, which is not present on this machine.

## Panel-size sweep (Study 3)
NOT RUN — same reason as above.

## Bootstrap CIs
| Task | AUC | 95% CI |
|---|---|---|
| Parkinson's (Sakar) | 0.850 | [0.796, 0.901] |
| Voice disorder (SVD) | 0.866 | [0.844, 0.888] |
| Immune age, internal CV | NOT RUN | gene-expression repo not present |
| Immune age, external | NOT RUN | gene-expression repo not present |

## Acoustic feature families (top 25, Sakar)
note: the first run of this code had a real bug — `pd.DataFrame({"mi": mi, "imp": imp}).head(25)`
silently reindexes to the union of the two Series' labels when they're sorted
in different orders, so it was not actually selecting the top-25-by-mutual-
information rows (it originally reported 0% in every family). Fixed in
`controls_voice_antibody.py` to build the combined frame explicitly in `mi`'s
order, then re-ran. Numbers below are from the corrected run.

| Family | Count in top 25 | Share |
|---|---|---|
| jitter | 0 | 0% |
| shimmer | 0 | 0% |
| HNR | 0 | 0% |
| MFCC | 1 | 4% |
| TQWT | 17 | 68% |
| other | 7 | 28% |
- top 5 individual features by mutual information: `tqwt_entropy_log_dec_35`, `std_delta_delta_log_energy`, `std_8th_delta_delta`, `mean_MFCC_2nd_coef`, `tqwt_TKEO_mean_dec_16`
- reading: none of the classic clinical dysphonia markers (jitter/shimmer/HNR) make the top 25 for this Random Forest — the tunable-Q wavelet transform (TQWT) coefficients dominate (68%), with one MFCC and several energy/delta ("other") features filling out the rest. Worth flagging honestly in the paper: this doesn't contradict the physiological story in Section 7.3 (jitter/shimmer/HNR still separate the classes on their own, per the univariate work elsewhere in the repo) — it means the *specific* Random Forest on Sakar's 752-feature representation leans on TQWT texture over classical perturbation measures, which is a model-specific finding, not a claim that jitter/shimmer/HNR carry no signal.

## Empirical curves
- results/curves.json written: no — that deliverable belongs to Task 1 (gene-expression repo), not present on this machine
- internal AUC / average precision: n/a
- external AUC / average precision: n/a

## Voice bootstrap CIs (detail)
- Parkinson's (Sakar): 252 speakers (64 healthy, 188 PD); per-speaker AUC 0.850, 95% CI [0.796, 0.901]; per-speaker accuracy 0.821, 95% CI [0.774, 0.869]
- Voice disorder (SVD): 1,119 speakers (669 healthy, 450 pathological); per-speaker AUC 0.866, 95% CI [0.844, 0.888]; per-speaker accuracy 0.804, 95% CI [0.780, 0.828]
- both are subject-independent (GroupKFold by speaker/id) with per-speaker probability aggregation, matching the paper's methodology
- both bootstrap CIs sit above the paper's currently-cited reference AUCs (0.83 Sakar, 0.86 SVD) — reasonable, as they're recomputed on a slightly different fold shuffle/imputation path than the original pipeline runs; if the paper wants an exact match it should cite the CI run's own point estimate (0.850 / 0.866) alongside the interval, not the older 0.83/0.86 point estimate with this interval
- reports/controls_voice.md and reports/feature_importance_sakar.csv are the committed artifacts for this section

## Renames
- directories renamed: `svd-success`→`voice-disorder-svd`, `parkinsons-success`→`parkinsons-sakar`, `parkinsons-uci-fail`→`parkinsons-oxford-uci`, `voiced-fail`→`voice-disorder-voiced`, `coswara-fail`→`acute-illness-cross-corpus` (all via `git mv`, history preserved)
- references updated in: top-level `README.md` results table, `DEVLOG.md`, and the `usage`/`warning` fields of all four `models/model_card.json` files (previously pointed at a stale `0N_name` numbering scheme, e.g. `04_parkinsons_sakar/predict.py`)
- verification: `grep -rniE "success|fail"` across `*.py/*.md/*.json` (excluding `.venv`/`.git`/`external_datasets`) returns only legitimate prose ("failed to preprocess", "successfully preprocessed", the paper's narrative use of "success"/"failure" as findings) — no directory-name-shaped matches remain
- antibody repo naming mismatch: confirmed — the repo title/URL says "influenza" but the dataset used throughout (per its own `README.md`) is **AVIDa-SARS-CoV-2**, and "variants" is misspelled as "varients" in the repo name. Checked whether "influenza" appears inside the repo's own tracked files (`*.py`, `*.md`, `*.json`, `*.txt`) — it does not; the repo's internal docs already correctly say SARS-CoV-2. So there is nothing to fix inside the repo; the mismatch is only in the external GitHub repo name/title, which per instructions I did not rename.

## Did not run
- Task 1 (immunosenescence random-gene / panel-size / filter-leakage / curves / bootstrap controls): gene-expression repo not present on this machine at all — nothing to run against.
- Task 3 (antibody antigen-holdout ablation): the antibody repo is present, but `data/processed/train.csv` does not exist yet (the repo's own README says it's produced by `python download_data.py`, which has not been run), and the repo has no local Python environment with `torch`/the ESM2 backbone installed. This is also explicitly flagged in the brief as the expensive task (two 5-epoch training runs, GPU-bound) — did not want to spend that setup and compute time without confirming scope first. Can proceed on request.
