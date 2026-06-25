# ECG Rhythm Classification with a 1D CNN — extended coverage via clinically-motivated class merging (single lead)

A single-lead ECG rhythm classifier (Keras 3 / PyTorch) built on the
**Chapman–Shaoxing** database using **lead I only** (≈ smartwatch-grade signal).

This branch extends the [base 5-class classifier](../01_base_five_classes/) to **cover ~98.6 % of the dataset**
by adding atrial flutter (AF) and sinus irregularity (SA) — two rhythms that are **not
reliably separable from their look-alikes on a single lead**. Instead of forcing
impossible distinctions, the model uses two **clinically-motivated merges**:

- **`SRSA`** = sinus rhythm + sinus irregularity (both benign sinus)
- **`AFIBAF`** = atrial fibrillation + atrial flutter (same screening-level action)

> ⚠️ **Research / educational project. Not a medical device and not for clinical use.**

---

## Overview

| | |
|---|---|
| **Task** | 5-class single-lead rhythm classification (with two clinical merges) |
| **Classes** | `SRSA`, `AFIBAF`, `SB` (sinus bradycardia), `SVT` (supraventricular tachycardia), `ST` (sinus tachycardia) |
| **Coverage** | ~98.6 % of the dataset (10 494 records) vs 90.7 % for the base 5 pure classes |
| **Input** | single lead (lead I), 5000 samples (10 s @ 500 Hz) |
| **Model** | 3-block 1D-CNN (first kernel **171**) → global average pooling → softmax |

## Results (held-out test set, n = 1050)

| Class | Precision | Recall | F1 | Support |
|-------|----------:|-------:|----:|--------:|
| SRSA   | 0.985 | 0.906 | 0.944 | 223 |
| AFIBAF | 0.920 | 0.932 | 0.926 | 222 |
| SB     | 0.970 | 0.990 | 0.980 | 389 |
| SVT    | 0.857 | 0.915 | 0.885 | 59  |
| ST     | 0.931 | 0.949 | 0.940 | 157 |
| **Accuracy** | | | **0.950** | 1050 |
| **Macro-F1** | | | **0.935** | 1050 |
| **Weighted-F1** | | | 0.950 | 1050 |

A 5-seed ablation puts the same configuration at **macro-F1 = 93.45 % ± 0.21**, so the
single-run number above is representative, not a lucky seed.

## Key design decisions (and the research behind them)

- **Clinically-motivated class merges, not a hack.** Adding AF and SA as separate
  classes caused a minority-class collapse (their F1 dropped to ~0.2–0.3) because they
  are nearly indistinguishable from AFIB / SR on a single lead. Merging look-alikes that
  share the same clinical management is a standard, defensible choice — it aligns the
  label granularity with what the modality can actually resolve, and restores macro-F1
  from ~73 % (7 raw classes) to ~93.5 %.
- **First-layer kernel = 171, chosen by a multi-seed ablation.** Sweeping the first
  kernel (11 → 181, 5 seeds each) showed a clear optimum around **151–171** for the
  merged classes — *much larger* than the ~51 optimum found for the original 5 clean
  classes. The merged classes absorb temporally-richer rhythms (flutter, sinus
  irregularity) that benefit from a larger receptive field. This is a concrete example
  of an optimal hyper-parameter being **task-dependent**, established by measurement.
- **No class weighting.** An A/B test showed weights did not help here (and slightly hurt
  the over-predicted SVT class) — exposed via the `USE_WEIGHTS` switch.
- **Honest evaluation.** Three-way train/validation/test split; the test set is touched
  exactly once. Per-class precision / recall / F1 + macro-F1 (accuracy alone hides the
  behaviour on the minority classes).

## How to run

```bash
pip install -r ../requirements.txt
```

Download the **ChapmanECG** dataset and place it next to the notebook:
`Diagnostics.xlsx` (file name → rhythm label) and the `ECGData/` folder (per-record CSV
signals). Then open `ECG_1D_CNN_plus_AFIBAF_SRSA.ipynb` and run all cells.

**Data source:** original ChapmanECG collection on figshare —
<https://figshare.com/collections/ChapmanECG/4560497>. A larger, merged
*Chapman-Shaoxing-Ningbo* version is available on PhysioNet
(<https://doi.org/10.13026/wgex-er52>). The raw data is **not** redistributed here.

## Files in this folder

```
ECG_1D_CNN_plus_AFIBAF_SRSA.ipynb
README.md   # this file
```
(Shared `LICENSE` and `requirements.txt` live in the repository root.)

## Roadmap / future work

- **Out-of-distribution handling** (a NOISE / OTHER reject class and/or embedding-based
  detectors) so unseen rhythms and bad signals are flagged, not confidently misclassified.
- **Multi-lead** to push past the single-lead ceiling for AF↔AFIB (atrial flutter shows
  characteristic flutter waves on leads II/V1 that lead I largely misses). Note this is a
  *morphological* limit; the SR↔SA limit is *temporal* and needs rhythm-aware modelling
  (R-R interval features / recurrent or attention layers), not more leads.
- **k-fold cross-validation** for macro-F1 ± std and **external validation** on independent
  single-lead data.

## Dataset citation

Zheng, J., Zhang, J., Danioko, S. et al. *A 12-lead electrocardiogram database for
arrhythmia research covering more than 10,000 patients.* Scientific Data 7, 48 (2020).
<https://doi.org/10.1038/s41597-020-0386-x>

## License

Released under the MIT License — see [LICENSE](../LICENSE).

---

*Author: Marek Malý*
