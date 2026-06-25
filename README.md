# ECG Rhythm Classification from a Single Lead using shallow 1D CNN

A single-lead ECG rhythm classifier (Keras 3 / PyTorch) trained on the public
**Chapman–Shaoxing** database, using **lead I only** to emulate smartwatch-grade
single-lead recordings (lead I ≈ potential between the right and left arm).

> ⚠️ **Research / educational project. Not a medical device and not for clinical use.**

This repository is **one project told in two parts** — a base classifier, and an
extension that pushes its coverage as far as a single lead allows.

## The story

### Part 1 — [base: 5 clean rhythm classes](01_base_five_classes/)

A 1D-CNN classifying five rhythms (sinus rhythm, atrial fibrillation, sinus
bradycardia, supraventricular tachycardia, sinus tachycardia) at **macro-F1 ≈ 95.4 %**
on a single lead, built with deliberately honest methodology: an untouched test set,
per-class precision/recall/F1 (not just accuracy), and a multi-seed kernel ablation.

### Part 2 — [extended: +2 rhythms via clinically-motivated merging](02_extended_merged_classes/)

Adding **atrial flutter** and **sinus irregularity** caused a minority-class collapse —
on a single lead they are essentially indistinguishable from atrial fibrillation and
normal sinus rhythm. Rather than forcing impossible distinctions, the look-alikes were
**merged into clinically-equivalent classes** (`SRSA`, `AFIBAF`). This restores
**macro-F1 ≈ 93.5 %** while raising data coverage from **90.7 % to 98.6 %**, and shows
that the optimal convolution kernel is **task-dependent** (it shifted from ~51 to ~171
for the merged task).

| Part                  | Classes                   | Data coverage | Macro-F1 |
| --------------------- | ------------------------- | -------------:| --------:|
| 1 — base              | SR, AFIB, SB, SVT, ST     | 90.7 %        | ~95.4 %  |
| 2 — extended (merged) | SRSA, AFIBAF, SB, SVT, ST | 98.6 %        | ~93.5 %  |

## What this project demonstrates

- **Honest evaluation** — no data leakage (three-way split, test set touched once);
  macro-F1 and per-class metrics, not just accuracy.
- **Decisions driven by measurement** — multi-seed kernel ablations and with/without
  class-weighting A/B tests, rather than guesswork.
- **Engineering + domain judgement** — recognising when two classes are *not* separable
  on the modality, and merging them on clinical grounds instead of chasing a metric.

## Repository structure

```
01_base_five_classes/        # Part 1 — 5 clean classes  (notebook + README)
02_extended_merged_classes/  # Part 2 — extended, merged classes (notebook + README)
requirements.txt
LICENSE
README.md
```

Each part has its **own README** with the detailed results and rationale.

## How to run

```bash
pip install -r requirements.txt
```

The raw **ChapmanECG** data is **not** redistributed. Download it from figshare
(<https://figshare.com/collections/ChapmanECG/4560497>) and place `Diagnostics.xlsx`
and the `ECGData/` folder next to the notebook you want to run. Then open the notebook
in either part and run all cells.

## Dataset citation

Zheng, J., Zhang, J., Danioko, S. et al. *A 12-lead electrocardiogram database for
arrhythmia research covering more than 10,000 patients.* Scientific Data 7, 48 (2020).
<https://doi.org/10.1038/s41597-020-0386-x>

## License

Released under the MIT License — see [LICENSE](LICENSE).

---

*Author: Marek Malý · maly1972@seznam.cz*
