# OpenADMET CYP Inhibition Blind Challenge 2026 — method reports

Method reports for the three submission tracks.

**Author:** Jamal Shamsara

| Track | Report | Submitted file | Blind score |
|---|---|---|---|
| Direct inhibition (regression) | [**regression.md**](regression.md) | `25_0_per_endpoint_best_regression_submission.csv` | MA-ST-RAE **0.4749** (**0.4593** on the full 750-compound reveal) |
| Time-dependent inhibition (classification) | [**classification.md**](classification.md) | `10_25_majority_vote_classification_submission.csv` | MA-MCC **0.3846** |
| CYP3A4 structure prediction (pose) | [**pose.md**](pose.md) | `24_9_manual_structures.zip` | pending |

Each report describes the **single best-scoring submission** for its track, not the full
exploration behind it. Where a number is quoted it is the organisers' own blind score unless
it is explicitly labelled as internal.

## Common ground

All three tracks were approached from one principle, which the challenge's design makes
unavoidable: **a local score is a hypothesis about the blind score, and most of them are
wrong.** The published training data and the blinded test set differ in label distribution,
assay condition and chemical space, so a model selected purely on cross-validation tends to
be mis-centred on the test set even when it ranks compounds correctly. Every selection
decision reported here was therefore either (a) made on a locally validated protocol whose
agreement with blind placement had itself been measured, or (b) made on already-scored blind
results and labelled as such.

Nothing in these reports used the blinded labels. Where a submission reuses a column from an
earlier submission of ours, that is stated and the earlier blind score is given, because the
choice was informed by it.

## Software

RDKit, scikit-learn, PyTorch, Chemprop (D-MPNN), XGBoost / HistGradientBoosting, OpenEye
(Spruce, POSIT, Omega, QUACPAC), AutoDock Vina 1.2.7 + Meeko, xTB (GFN2), PoseBusters, and
the open-source [`mlmolprop`](https://github.com/shamsaraj/mlmolprop) package for the shared
descriptor, split and metric machinery.

## Contact

Jamal Shamsara
