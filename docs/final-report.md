# 프로젝트 최종보고 | Unsupervised EEG Pattern Discovery and Early Warning Framework

---

# 0. 연구 배경

EEG(Electroencephalography)는 뇌의 전기적 활동을 기록하는 대표적인 생체 신호로, 뇌전증(Seizure), GPD, GRDA, LRDA 등 다양한 신경학적 이상 패턴을 분석하는 데 활용된다.

최근 EEG 분석 분야에서는 딥러닝 기반 분류 모델이 높은 성능을 보이고 있으나, 대부분 지도학습(Supervised Learning)에 의존하고 있다. 이러한 접근은 충분한 라벨 데이터가 필요하며, 새로운 EEG 패턴이나 미확인 이상 신호를 발견하는 데에는 한계가 존재한다.

반면 비지도학습(Unsupervised Learning)은 라벨 없이 EEG의 잠재 구조를 탐색하고 이상 패턴을 탐지할 수 있다는 장점을 가진다.

본 연구에서는 EEG 시계열 데이터를 통계적 특징으로 변환한 후, PCA, K-Means Clustering, Isolation Forest를 활용하여 EEG의 잠재 구조를 분석하고 이상 신호를 탐지하는 경량 분석 프레임워크를 구축하였다.

---

# 1. 프로젝트 개요

본 프로젝트의 목표는 EEG 데이터 내 숨겨진 패턴과 구조를 발견하고, 비정상적인 EEG 신호를 자동 탐지할 수 있는 비지도학습 기반 분석 시스템을 구축하는 것이다.

구체적으로 다음 목표를 설정하였다.

### 연구 목표

1. EEG 잠재 구조 발견
2. 이상 패턴 탐지
3. 해석 가능한 EEG 분석
4. 경량 EEG 분석 시스템 구축

---

# 2. 데이터셋

## Dataset

* Source: HMS Harmful Brain Activity Classification
* Format: EEG Parquet Files
* Sampling Rate: 200 Hz
* Total EEG Files: 약 17,300개

---

## EEG 채널 구성

본 연구에서는 EEG 19채널과 EKG 채널을 포함한 데이터를 활용하였다.

```text
Fp1, F3, C3, P3, F7, T3, T5, O1,
Fz, Cz, Pz,
Fp2, F4, C4, P4, F8, T4, T6, O2,
EKG
```

---

# 3. 데이터 전처리 및 Feature Engineering

EEG 원시 시계열 데이터를 직접 분석하는 대신, 각 EEG 파일을 통계적 특징 벡터로 변환하였다.

---

## 추출 Feature

각 채널에 대해 다음 특징을 계산하였다.

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

## 초기 이상 패턴 탐지

초기 Feature Matrix를 기반으로 Isolation Forest를 적용하여 이상 EEG 후보를 탐색하였다.

분석 결과, 이상치로 탐지된 EEG 대부분이 예상했던 발작(Seizure) 또는 병적 EEG 패턴이 아니라 비정상적으로 큰 진폭을 가지는 신호로 나타났다.

상위 이상치 후보들의 원시 파형을 직접 확인한 결과 다음과 같은 특징이 반복적으로 관찰되었다.

- 수만 단위의 비정상적인 진폭
- 순간적인 수직 점프(vertical jump)
- 장시간 지속되는 포화(saturation) 구간
- EEG 생리학적으로 설명하기 어려운 급격한 신호 변화

대표 사례는 아래와 같다.

<img width="570" height="433" alt="image" src="https://github.com/user-attachments/assets/bbc163f4-ebe0-4055-b336-5bafd2f32221" />

위 그림에서는 정상적인 EEG 진폭 범위를 크게 벗어나는 신호 포화 현상이 관찰되며, 이러한 패턴은 실제 뇌 활동보다는 측정 오류 또는 센서 Artifact에 가까운 형태를 보였다.

따라서 초기 Isolation Forest 결과는 EEG 이상 신호뿐 아니라 데이터 품질 문제 역시 탐지하고 있음을 확인하였다.

---

## Artifact 제거

초기 분석 과정에서 Isolation Forest가 탐지한 이상치 대부분이 실제 뇌파 이상 신호가 아닌 측정 Artifact임을 확인하였다.

대표적인 Artifact 특징은 다음과 같다.

* 비정상적으로 큰 진폭
* 수직 점프 형태의 파형
* 채널 전체에 걸친 순간적인 포화 현상

이를 제거하기 위해 다음 조건을 적용하였다.

```python
abs(signal) > 5000
```

따라서 해당 EEG는 분석 대상에서 제외하였다.

---

## 최종 Feature Matrix

```text
EEG 수 : 8,739

Feature 수 : 180
```

---

## 이상 패턴 재탐지

최종 Feature Matrix를 이용하여 Isolation Forest로 다시 이상치를 확인하였다. 

그 결과 초기 분석에서 관찰되었던 비생리학적 포화 신호 대신,
주기성과 진폭 변화를 가지는 EEG 패턴들이 이상치로 탐지되었다.

<img width="997" height="372" alt="image" src="https://github.com/user-attachments/assets/13e0fffb-4262-4c9e-b707-f34a61828f51" />

이러한 신호는 실제 EEG의 통계적 특성으로 인해
정상 패턴과 구분된 것으로 해석할 수 있으며,
이후 Cluster 분석 및 Expert Label 비교 분석의 대상으로 활용하였다.

---

# 4. 차원 축소 (PCA)

고차원 EEG Feature Space를 시각적으로 분석하기 위해 PCA를 수행하였다.

---

<img width="989" height="690" alt="image" src="https://github.com/user-attachments/assets/c184fbeb-cf38-4604-a50d-4601424756b0" />

---

## 결과

```text
PC1 = 34.4%
PC2 = 10.6%
PC3 = 10.5%
```

```text
누적 설명력

PC1 + PC2 = 45.0%
PC1 + PC2 + PC3 = 55.5%
```

---

## 해석

PCA 결과 EEG 데이터는 하나의 균일한 집합이 아니라 여러 개의 잠재 구조를 형성하고 있음을 확인하였다.

이는 EEG 내부에 서로 다른 통계적 특성을 가진 패턴 그룹이 존재함을 의미한다.

---

# 5. 클러스터링 분석

EEG의 잠재 구조를 탐색하기 위해 K-Means Clustering을 수행하였다.

---

[INSERT FIGURE 2. K-Means Clustering Result]

---

## 결과

최종적으로 3개의 주요 클러스터가 형성되었다.

| Cluster   | Count |
| --------- | ----: |
| Cluster 1 | 5,678 |
| Cluster 0 | 2,523 |
| Cluster 2 |   538 |

---

## 해석

Cluster 1은 대부분의 EEG를 포함하는 일반적인 그룹으로 나타났다.

반면 Cluster 2는 상대적으로 적은 EEG만 포함하며 다른 그룹과 명확하게 구분되는 특성을 보였다.

---

# 6. 이상 패턴 탐지

Isolation Forest를 활용하여 이상 EEG 탐지를 수행하였다.

---

## 모델 설정

```python
contamination = 0.05
```

---

## 결과

| Class   | Count |
| ------- | ----: |
| Normal  | 8,302 |
| Anomaly |   437 |

---

[INSERT FIGURE 3. Anomaly Score Distribution]

---

## 해석

전체 EEG 중 약 5%가 이상 패턴으로 탐지되었다.

초기에는 대부분의 이상치가 Artifact로 확인되었으며, Artifact 제거 이후 실제 EEG 기반 이상 패턴 탐지가 가능해졌다.

---

# 7. 이상 패턴 정량 분석

Anomaly EEG와 Normal EEG의 Feature 평균을 비교하였다.

---

[INSERT FIGURE 4. Top Feature Difference]

---

## 주요 결과

Anomaly EEG는 다음 특징을 보였다.

* Range 증가
* Standard Deviation 증가
* Maximum Value 증가

---

## 해석

이상 EEG는 정상 EEG보다 더 큰 진폭과 변동성을 가지는 경향을 보였다.

특히 특정 채널이 아닌 전 채널에서 유사한 경향이 관찰되었다.

---

# 8. 클러스터 해석

각 Cluster의 평균 Feature를 비교하였다.

---

## Cluster 1

특징

* 낮은 Range
* 낮은 Standard Deviation
* Anomaly 비율 0.6%

정의

```text
저진폭 · 저변동성 EEG 그룹
```

---

## Cluster 0

특징

* 중간 수준 Range
* 중간 수준 Standard Deviation
* Anomaly 비율 2.3%

정의

```text
중간 활동성 EEG 그룹
```

---

## Cluster 2

특징

* 매우 높은 Range
* 매우 높은 Standard Deviation
* Anomaly 비율 64.1%

정의

```text
고진폭 · 고변동성 EEG 그룹
```

---

[INSERT FIGURE 5. Cluster Feature Comparison]

---

# 9. Expert Label 비교 분석

비지도학습 결과와 전문가 라벨을 비교하였다.

---

## Seizure 비율 분석

| Label   | Anomaly 비율 |
| ------- | ---------: |
| Seizure |       9.9% |
| Other   |       5.4% |
| GPD     |       5.3% |
| GRDA    |       3.3% |
| LRDA    |       3.4% |
| LPD     |       2.6% |

---

## 결과

Seizure EEG는 다른 EEG 유형보다 높은 비율로 Anomaly로 탐지되었다.

이는 Isolation Forest가 비지도 환경에서도 발작 관련 EEG 패턴을 상대적으로 특이하게 인식함을 의미한다.

---

[INSERT FIGURE 6. Expert Label vs Anomaly]

---

# 10. EEG 위험도 기반 조기 감지 모델

Isolation Forest Score를 활용하여 EEG 위험도를 정의하였다.

---

## 위험도 기준

| Risk Level | Condition        |
| ---------- | ---------------- |
| Normal     | score > 0.10     |
| Warning    | 0 ≤ score ≤ 0.10 |
| High Risk  | score < 0        |

---

## 결과

| Risk Level | Count |
| ---------- | ----: |
| Normal     | 7,027 |
| Warning    | 1,275 |
| High Risk  |   437 |

---

[INSERT FIGURE 7. Risk Level Distribution]

---

## Seizure 위험도 분석

| Risk Level | Ratio |
| ---------- | ----: |
| High Risk  |  9.9% |
| Warning    | 23.2% |
| Normal     | 66.9% |

---

## 해석

Seizure EEG의 약 33%가 Warning 이상으로 탐지되었다.

이는 Anomaly Score가 EEG 이상 신호 조기 감지를 위한 위험도 지표로 활용될 가능성을 보여준다.

---

# 11. 한계 분석

본 연구는 EEG 잠재 구조 탐색 및 이상 탐지에 성공하였으나 몇 가지 한계를 가진다.

* 통계 Feature 기반 분석으로 인해 시간적 패턴 정보 손실
* Isolation Forest Threshold 설정의 경험적 요소 존재
* 실제 임상 환경에서의 검증 부족
* EEG 세그먼트 단위 분석 미수행

---

# 12. 최종 결론

본 연구에서는 EEG 데이터를 통계적 Feature로 변환한 후 PCA, K-Means Clustering, Isolation Forest를 활용하여 비지도학습 기반 EEG 분석 프레임워크를 구축하였다.

분석 결과 EEG는 여러 개의 잠재 구조를 형성하고 있었으며, 특정 클러스터는 높은 진폭과 변동성을 가지는 이상 EEG 그룹으로 확인되었다.

또한 Isolation Forest는 Seizure EEG를 상대적으로 높은 비율로 탐지하였으며, 이를 기반으로 EEG 위험도 평가 및 조기 경고 시스템의 가능성을 확인하였다.

---

# 13. 프로젝트 의의

본 연구는 EEG 데이터를 대상으로 비지도학습 기반 잠재 구조 탐색, 이상 패턴 탐지, 위험도 평가를 통합적으로 수행하였다는 점에서 의의를 가진다.

특히 단순 시각화 수준을 넘어 Expert Label과의 비교 분석을 통해 비지도학습 결과의 해석 가능성을 제시하였다.

---

# 14. 기존 연구와의 차별성

기존 EEG 연구는 대부분 지도학습 기반 분류 정확도 향상에 초점을 맞춘다.

반면 본 연구는 라벨 없이 EEG의 잠재 구조를 발견하고 이상 패턴을 탐지하는 데 집중하였다.

또한 PCA, K-Means, Isolation Forest를 연계하여 EEG 구조 탐색부터 위험도 평가까지 하나의 분석 프레임워크로 통합하였다는 점에서 차별성을 가진다.

---

# 15. 추후 발전 방향

## 15.1 Temporal Feature 추가

현재 통계 Feature 중심 분석에서 나아가 FFT, Band Power, Spectral Entropy 등 주파수 특징을 추가할 수 있다.

---

## 15.2 Deep Learning 기반 이상 탐지

Autoencoder 및 Diffusion Model을 활용한 EEG 이상 탐지 연구로 확장 가능하다.

---

## 15.3 실시간 EEG 위험도 시스템

실시간 EEG 입력에 대해 위험도를 평가하고 이상 신호를 조기에 경고하는 시스템으로 발전시킬 수 있다.
