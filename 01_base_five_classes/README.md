# ECG Rhythm Classification with a 1D CNN (single lead)

A single-lead ECG rhythm classifier built with a 1D convolutional neural network
(Keras 3 / PyTorch backend). The model is trained on the **Chapman–Shaoxing** 12-lead
ECG database using **lead I only**, to emulate the kind of single-lead signal produced
by smartwatch-grade devices (lead I ≈ potential between the right and left arm).

> ⚠️ **Research / educational project. Not a medical device and not for clinical use.**

---

## Overview

| | |
|---|---|
| **Task** | 5-class ECG rhythm classification |
| **Classes** | `SR` sinus rhythm, `AFIB` atrial fibrillation, `SB` sinus bradycardia, `SVT` supraventricular tachycardia, `ST` sinus tachycardia |
| **Input** | single lead (lead I), 5000 samples (10 s @ 500 Hz) |
| **Model** | 3-block 1D-CNN → global average pooling → softmax |
| **Framework** | Keras 3 with the PyTorch backend |

## Pipeline

1. **Load** single-lead signals for the 5 target rhythms.
2. **Scale** each record independently (per-record z-score).
3. **Split** into train / validation / test = **80 / 10 / 10**, *stratified* by class.
4. **Train** the 1D-CNN with **EarlyStopping monitored on the validation set**
   (the test set is never used for any training decision).
5. **Evaluate** on the held-out test set with per-class precision / recall / F1,
   macro-F1 and the confusion matrix.

## Results (held-out test set, n = 965)

| Class | Precision | Recall | F1 | Support |
|-------|----------:|-------:|----:|--------:|
| SR    | 0.961 | 0.951 | 0.956 | 182 |
| AFIB  | 0.945 | 0.966 | 0.956 | 178 |
| SB    | 0.979 | 0.974 | 0.977 | 389 |
| SVT   | 0.905 | 0.966 | 0.934 | 59  |
| ST    | 0.961 | 0.936 | 0.948 | 157 |
| **Accuracy** | | | **0.962** | 965 |
| **Macro avg** | 0.950 | 0.959 | **0.954** | 965 |
| **Weighted avg** | 0.962 | 0.962 | 0.962 | 965 |

**Macro-F1 ≈ 95.4 %** on a single lead. For context, this is in the range of
published multi-class results on this dataset that typically use all 12 leads.

## Key design decisions (why it is set up this way)

- **Three-way split + EarlyStopping on validation.** The test set is touched exactly
  once, at the very end, so the reported numbers are an honest, independent estimate
  (no information leak from model selection).
- **Per-class precision / recall / F1, not just accuracy.** With imbalanced classes,
  accuracy is misleading; macro-F1 weights every class equally and reveals where the
  model is actually weakest (here: SVT precision — false alarms on the smallest class).
- **Class weighting evaluated, not assumed.** An A/B test (with vs without
  `class_weight`) showed no measurable benefit for this 5-class setup, so it is off by
  default but exposed through the `USE_WEIGHTS` switch for future use.
- **Single lead by design.** Lead I is the lead that best matches consumer wearable
  ECG, which is the long-term deployment target.

## How to run

```bash
pip install -r ../requirements.txt
```

Download the **ChapmanECG** dataset and place it next to the notebook:

- `Diagnostics.xlsx` — the diagnosis table (file name → rhythm label)
- `ECGData/` — the folder of per-record CSV signal files

Then open `ECG_1D_CNN.ipynb` and run all cells (Kernel → Restart & Run All).

**Data source:** original ChapmanECG collection on figshare —
<https://figshare.com/collections/ChapmanECG/4560497>.
A larger, merged *Chapman-Shaoxing-Ningbo* version is available on PhysioNet
(<https://doi.org/10.13026/wgex-er52>). The raw data is **not** redistributed here.

## Files in this folder

```
ECG_1D_CNN.ipynb
README.md   # this file
```
(Shared `LICENSE` and `requirements.txt` live in the repository root.)

## Roadmap / future work

- **Out-of-distribution handling.** A softmax classifier always forces an input into
  one of the 5 classes. Planned: a `NOISE` / `OTHER` reject class and/or an
  embedding-based detector (Mahalanobis / kNN) so unseen rhythms and bad signals are
  flagged instead of confidently misclassified.
- **k-fold cross-validation** to report macro-F1 ± standard deviation.
- **External validation** on independent single-lead data
  (PhysioNet/CinC 2017 AliveCor, paired Apple Watch datasets).
- **Retrain on the larger Chapman-Shaoxing-Ningbo** database for better coverage of
  the minority classes.

## Dataset citation

Zheng, J., Zhang, J., Danioko, S. et al. *A 12-lead electrocardiogram database for
arrhythmia research covering more than 10,000 patients.* Scientific Data 7, 48 (2020).
<https://doi.org/10.1038/s41597-020-0386-x>

## License

Released under the MIT License — see [LICENSE](../LICENSE).

---

*Author: <YOUR NAME>*
