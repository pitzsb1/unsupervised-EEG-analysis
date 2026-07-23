# NeuroStat: Unsupervised EEG Pattern Discovery and Early Warning Framework

## 0. Background

Electroencephalography (EEG) is a representative biosignal that records the brain's electrical activity and is widely used to analyze various neurological abnormalities, including **Seizures, GPD (Generalized Periodic Discharges), GRDA (Generalized Rhythmic Delta Activity), and LRDA (Lateralized Rhythmic Delta Activity).**

Recently, deep learning–based EEG classification models have achieved remarkable performance. However, most existing approaches rely on **supervised learning**, requiring large amounts of labeled data. This dependency limits their ability to discover previously unseen EEG patterns or identify unknown abnormalities.

In contrast, **unsupervised learning** enables the exploration of latent EEG structures and anomaly detection without requiring labeled data.

In this project, raw EEG time-series signals were transformed into statistical features, followed by **Principal Component Analysis (PCA), K-Means Clustering, and Isolation Forest** to analyze latent EEG structures and detect anomalous patterns through a lightweight analysis framework.

---

# 1. Project Overview

## Objectives

* Discover latent EEG structures
* Detect anomalous EEG patterns
* Provide interpretable EEG analysis
* Build an EEG risk-based early warning framework

---

# 2. Dataset

## Dataset Information

* **Source:** HMS Harmful Brain Activity Classification
* **Format:** EEG Parquet Files
* **Sampling Rate:** 200 Hz
* **Total EEG Files:** Approximately 17,300

## EEG Channels

```text
Fp1, F3, C3, P3, F7, T3, T5, O1,
Fz, Cz, Pz,
Fp2, F4, C4, P4, F8, T4, T6, O2,
EKG
```

---

# 3. Initial EEG Exploration & Feature Engineering

Instead of directly analyzing raw EEG time-series data, statistical features were extracted from each EEG recording.

## Extracted Features

For each channel:

* Mean
* Standard Deviation
* Minimum
* Maximum
* Range
* Absolute Mean
* Skewness
* Kurtosis
* Flat Signal Indicator

---

## Analysis Framework

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/97d65ac6-b552-4db5-b53f-2aced4099ef6" />

---

# 4. Initial EEG Structure Exploration

## Principal Component Analysis (PCA)

PCA was performed to visualize the high-dimensional EEG feature space.

<img width="989" height="690" alt="image" src="https://github.com/user-attachments/assets/f8937456-88ad-4f3c-ba22-deea89ad9b71" />

### Results

```text
PC1 = 34.4%
PC2 = 10.6%
PC3 = 10.5%

Cumulative Explained Variance

PC1 + PC2 = 45.0%
PC1 + PC2 + PC3 = 55.5%
```

### Interpretation

The PCA results indicate that EEG recordings do not form a single homogeneous distribution but instead consist of multiple latent structures.

---

## Initial Isolation Forest

<img width="807" height="603" alt="image" src="https://github.com/user-attachments/assets/a2aea80a-0df5-4f83-a5f4-e9d91a14e488" />

---

# 5. Artifact Detection and Removal

Initial Isolation Forest analysis revealed that the highest-ranked anomalies were not pathological EEG patterns but abnormal amplitude saturation signals.

Typical characteristics included:

* Extremely large amplitudes (tens of thousands)
* Vertical jumps
* Long-term saturation
* Non-physiological abrupt signal changes

<img width="570" height="433" alt="image" src="https://github.com/user-attachments/assets/16bc5863-7119-4e16-a874-18109900360c" />

These patterns resembled sensor failures or recording artifacts rather than genuine EEG activity.

This observation suggests that the Isolation Forest detected not only abnormal EEG signals but also data quality issues.

---

## Artifact Removal

Artifact removal criterion:

```python
abs(signal) > 5000
```

EEG recordings satisfying this condition were removed.

---

# 6. Reanalysis Using Cleaned Data

## Clean Feature Matrix

```text
Number of EEGs: 8,739

Number of Features: 180
```

---

## PCA After Cleaning

PCA was performed again after artifact removal.

<img width="989" height="690" alt="image" src="https://github.com/user-attachments/assets/25cb4bd1-e863-41e0-ab74-715996b28bf8" />

---

## Isolation Forest After Cleaning

Isolation Forest was re-applied to the cleaned feature matrix.

<img width="997" height="372" alt="image" src="https://github.com/user-attachments/assets/db9bc348-26a3-4273-9dfd-d112334a0ef3" />

Instead of detecting non-physiological saturated signals, the model identified EEG patterns exhibiting meaningful periodicity and amplitude variations.

This demonstrates that proper data cleaning significantly improves EEG anomaly detection.

---

## EKG Influence Analysis

During the initial analysis, the EKG channel often exhibited larger amplitudes than EEG channels.

However, after artifact removal, Isolation Forest continued to identify meaningful EEG anomalies, indicating that the final anomaly detection results were not dominated by the EKG channel alone.

---

# 7. Quantitative Analysis of Anomalies

## Anomaly vs. Normal Comparison

EEG recordings were divided into **Anomaly** and **Normal** groups based on Isolation Forest predictions.

<img width="877" height="545" alt="image" src="https://github.com/user-attachments/assets/8e78acd4-8bfe-4970-b94d-0e30d95eacb2" />

## Key Findings

Compared with normal EEGs, anomalous EEGs exhibited:

* Larger signal ranges
* Higher standard deviations
* Higher maximum amplitudes

### Interpretation

Anomalous EEG recordings generally showed greater amplitudes and variability than normal EEGs.

Moreover, these characteristics appeared consistently across multiple channels rather than being limited to a specific electrode.

---

# 8. Cluster Interpretation

## Cluster Distribution

| Cluster   | Count |
| --------- | ----: |
| Cluster 1 | 5,678 |
| Cluster 0 | 2,523 |
| Cluster 2 |   538 |

---

## Cluster Characteristics

### Cluster 1

* Low amplitude
* Low variability
* Approximately 0.6% anomaly rate

### Cluster 0

* Moderate amplitude
* Moderate variability
* Approximately 2.3% anomaly rate

### Cluster 2

* Very large signal range
* Very high standard deviation
* Approximately 64.1% anomaly rate

```text
High-Amplitude / High-Variability EEG Group
```

<img width="859" height="508" alt="image" src="https://github.com/user-attachments/assets/41d33fb7-60a0-4c4f-b091-e23d70932843" />

### Interpretation

Cluster 2 contained the majority of EEG recordings detected as anomalous by Isolation Forest.

This suggests that specific EEG pattern groups are strongly associated with abnormal brain activity.

---

# 9. Validation Using Expert Labels

The unsupervised learning results were validated by comparing them with expert annotations.

## Anomaly Ratio by Label

| Label   | Anomaly Rate |
| ------- | -----------: |
| Seizure |         9.9% |
| Other   |         5.4% |
| GPD     |         5.3% |
| LRDA    |         3.4% |
| GRDA    |         3.3% |
| LPD     |         2.6% |

### Interpretation

Seizure EEG recordings were detected as anomalies at a considerably higher rate than other EEG categories.

This indicates that Isolation Forest can identify seizure-related EEG characteristics even without supervision.

---

# 10. EEG Risk-Based Early Warning Framework

## Risk Definition

| Risk Level | Condition        |
| ---------- | ---------------- |
| Normal     | Score > 0.10     |
| Warning    | 0 ≤ Score ≤ 0.10 |
| High Risk  | Score < 0        |

---

## Results

| Risk Level | Count |
| ---------- | ----: |
| Normal     | 7,027 |
| Warning    | 1,275 |
| High Risk  |   437 |

---

## Seizure Risk Distribution

| Risk Level | Ratio |
| ---------- | ----: |
| High Risk  |  9.9% |
| Warning    | 23.2% |
| Normal     | 66.9% |

### Interpretation

Approximately **33% of seizure EEG recordings** were classified as either **Warning** or **High Risk**.

This suggests that the anomaly score has potential as an early indicator of abnormal EEG activity.

---

# 11. Limitations

Although this study successfully explored latent EEG structures and detected anomalous patterns, several limitations remain.

## Limitations of Statistical Features

This study relied on statistical features such as mean, standard deviation, range, and skewness.

While these features are computationally efficient and interpretable, they do not fully capture the temporal dynamics of EEG signals.

Consequently, EEG recordings with similar statistical characteristics but different waveform structures may not be distinguishable.

---

## Lack of Frequency-Domain Features

Frequency-domain information plays an essential role in EEG analysis.

However, only time-domain features were used in this study, limiting the detection of abnormalities expressed through frequency characteristics.

Future work will incorporate FFT-based features such as:

* Delta / Theta / Alpha / Beta / Gamma Band Power
* Relative Band Power
* Spectral Entropy
* Spectral Edge Frequency

to enable more comprehensive EEG analysis.

---

# 12. Conclusion

This study developed an unsupervised EEG analysis framework by transforming EEG time-series data into statistical features.

During the initial anomaly analysis, recording artifacts were identified and systematically removed based on waveform inspection.

Subsequent analyses on the cleaned dataset successfully revealed latent EEG structures and identified high-risk EEG groups.

Furthermore, comparison with expert annotations demonstrated the interpretability of the unsupervised results, leading to the proposal of an EEG risk-based early warning framework.

---

# 13. Contributions

This work integrates the following components into a unified EEG analysis pipeline:

* Latent EEG structure discovery
* Unsupervised anomaly detection
* Validation using expert labels
* EEG risk assessment
* Early warning framework

---

# 14. Novelty

Most existing EEG studies focus primarily on improving classification accuracy through supervised learning.

In contrast, this study emphasizes discovering latent EEG structures and detecting anomalies **without relying on labels**.

Furthermore, the complete pipeline—from **artifact detection** to **data cleaning**, **reanalysis**, and **risk assessment**—distinguishes this work from conventional approaches.

---

# 15. Future Work

## Frequency-Based EEG Features

Future work will extend the current time-domain features by incorporating frequency-domain information using FFT.

Potential additions include:

* Delta / Theta / Alpha / Beta / Gamma Band Power
* Relative Band Power
* Spectral Entropy
* Spectral Edge Frequency

These features are expected to provide a richer representation of EEG signals and improve latent structure analysis.

---

## Real-Time Early Warning System

The proposed framework currently estimates EEG risk using anomaly scores computed from offline data.

Future work aims to extend this framework into an **online monitoring system** capable of processing real-time EEG streams.

A potential pipeline is illustrated below:

```text
EEG Input
    ↓
Feature Extraction
    ↓
Anomaly Score Estimation
    ↓
Risk Assessment
    ↓
Early Warning
```

Such a system could support continuous monitoring and early seizure warning in real-world clinical environments.

---
