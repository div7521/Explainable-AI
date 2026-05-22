# Explainable AI for Bearing Fault Diagnosis

**Divyanshi Gupta | 2022B3A41257P | BITS Pilani**  
LOP under Professor Madhurjya Dev Choudhury | May 2026

---

## What this project does

Bearing faults cause 40–50% of industrial motor failures. Deep learning models can detect them with high accuracy — but nobody trusts a black box in a factory. This project builds a full XAI pipeline that not only classifies bearing faults but explains exactly why it made each decision, in terms that an engineer can verify against physical reality.

The work covers:
- Converting raw vibration signals into STFT spectrograms and Cyclic Spectral Coherence (CSC) maps
- Training ResNet50 via transfer learning to classify 10 fault classes
- Applying Grad-CAM to show which part of the spectrogram the CNN attended to
- Running SHAP on handcrafted statistical features via XGBoost
- Comparing STFT vs CSC as CNN inputs — for both accuracy and physical interpretability

---

## Dataset

**CWRU Bearing Benchmark Dataset** — Case Western Reserve University  
https://www.kaggle.com/datasets/brjapon/cwru-bearing-datasets

Bearing: SKF 6205-2RS JEM · Sampling rate: 12,000 Hz · Motor speed: 1772 RPM

| Class | File | Fault | Defect |
|-------|------|-------|--------|
| 0 | Time_Normal_1_098.mat | Healthy | — |
| 1 | B007_1_123.mat | Ball | 0.007 in |
| 2 | B014_1_190.mat | Ball | 0.014 in |
| 3 | B021_1_227.mat | Ball | 0.021 in |
| 4 | IR007_1_110.mat | Inner Race | 0.007 in |
| 5 | IR014_1_175.mat | Inner Race | 0.014 in |
| 6 | IR021_1_214.mat | Inner Race | 0.021 in |
| 7 | OR007_6_1_136.mat | Outer Race | 0.007 in |
| 8 | OR014_6_1_202.mat | Outer Race | 0.014 in |
| 9 | OR021_6_1_239.mat | Outer Race | 0.021 in |

---

## Notebook — gradcam-shap.ipynb

The notebook runs end to end on Kaggle. Add the CWRU dataset before starting.

### Pipeline order

**1. Load signals**  
Each `.mat` file is loaded via `scipy.io.loadmat()`. The Drive End accelerometer signal (`DE_time`) is extracted and flattened to a 1D array of ~487,000 samples.

**2. Segment**  
Sliding window over the 1D signal. Two window sizes depending on representation:
- STFT → 2048 samples (0.17 sec), 50% overlap
- CSC → 12000 samples (1.0 sec), 50% overlap

**3. STFT spectrograms**  
`scipy.signal.stft` → magnitude → dB scale → normalize → resize to 224×224 → stack 3 channels → feed to ResNet50.

**4. Cyclic Spectral Coherence maps**  
STFT → instantaneous power `|Zxx|²` → FFT along time axis → normalize to coherence [0,1] → crop to cyclic 0–300 Hz / spectral 0–3000 Hz → resize 224×224 → stack 3 channels.  
CSC makes bearing fault frequencies explicit on the horizontal axis. A fault at BPFO shows up as a sharp vertical line at exactly 107 Hz — the CNN doesn't need to infer it.

**5. ResNet50 — two-phase transfer learning**  
Phase 1: freeze all ResNet layers, train only top head (lr=1e-3, 10 epochs)  
Phase 2: unfreeze last 30 layers of ResNet, fine-tune (lr=1e-5, 10 epochs)

Custom top: `GlobalAveragePooling2D → Dense(256, relu) → Dropout(0.4) → Dense(10, softmax)`

**6. Grad-CAM**  
Target layer: `conv5_block3_out` (last conv layer, output 7×7×2048).  
`GradientTape` records gradients of class score w.r.t. feature maps → global average pool → weighted sum → ReLU → upsample 7×7 to 224×224 → overlay on input.  
Cyan dashed lines mark theoretical fault frequencies. Alignment = the CNN learned real bearing physics.

**7. SHAP**  
Separate branch using pre-extracted statistical features from `feature_time_48k_2048_load_1.csv`.  
Features: max, min, mean, sd, rms, skewness, kurtosis, crest factor, form factor.  
Model: XGBoost (n_estimators=200, max_depth=5).  
Explainer: `shap.TreeExplainer` → SHAP values shape (n_samples, 9, 10).  
Plots: beeswarm per class, global bar chart, force plot for single predictions.

**8. STFT vs CSC comparison**  
Runs both models side by side and prints a summary table with:
- Overall accuracy
- Macro F1-score
- Grad-CAM interpretability score (fault freq activation / background activation ratio)

---

## Bearing fault frequencies at 1772 RPM

Computed from SKF 6205-2RS geometry: N=9, d=0.3126 in, D=1.748 in, α=0°

| | Hz | 2× | 3× |
|-|----|----|-----|
| BPFO | 107.36 | 214.72 | 322.08 |
| BPFI | 162.18 | 324.36 | 486.54 |
| BSF | 141.17 | 282.34 | 423.51 |
| FTF | 11.93 | 23.86 | 35.79 |
| Shaft | 29.53 | 59.06 | 88.59 |

Overlaid as cyan dashed lines on every Grad-CAM and CSC plot. If the red activation blob lands on the cyan line — the model is detecting real fault physics, not noise.

---

## Results

| Metric | STFT | CSC |
|--------|------|-----|
| Overall Accuracy | [run Cell 7] | [run Cell 7] |
| Macro F1 | [run Cell 7] | [run Cell 7] |
| Grad-CAM Interpretability Score | [run Cell 7] | [run Cell 7] |
| XGBoost Accuracy (SHAP) | 95.87% | — |
| Top SHAP feature | kurtosis | — |

---

## Key findings so far

- **Kurtosis** is the dominant feature for ball and inner race faults because those faults produce sharp, impulsive shocks — exactly what kurtosis measures
- **RMS and standard deviation** dominate for outer race faults because outer race defects produce sustained, regular energy increases rather than sharp spikes
- **Mean** carries near-zero SHAP values across all classes — vibration signals are zero-mean by nature
- **Grad-CAM on STFT** shows broad activation bands. **Grad-CAM on CSC** shows sharper vertical ridges that align more precisely with theoretical fault frequencies

---

## Files in this repo

| File | Description |
|------|-------------|
| `gradcam-shap.ipynb` | Main notebook — full pipeline |
| `stft_vs_csc_comparison.py` | Comparison script — run after training both models |
| `README.md` | This file |

---

## Dependencies

All pre-installed on Kaggle.

```
tensorflow, scipy, numpy, scikit-learn, xgboost, shap, matplotlib, seaborn, opencv-python, pandas
```

---

## Notes

- Do not restart the Kaggle kernel between training and the comparison script — `model_stft`, `model_csc`, `history_stft`, `history_csc` must all stay in memory
- CSC produces fewer training samples than STFT (longer window → fewer segments) — expected tradeoff, not a bug
- The Grad-CAM interpretability score is a custom metric written for this project, not from any library
- MATLAB was used only for initial raw signal visualization — all processing and modelling is in Python
