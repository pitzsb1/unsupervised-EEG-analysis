# 프로젝트 최종보고 | NeuroStat: Unsupervised EEG Pattern Discovery and Early Warning Framework

## 0. 연구 배경

EEG(Electroencephalography)는 뇌의 전기적 활동을 기록하는 대표적인 생체 신호로, 뇌전증(Seizure), GPD, GRDA, LRDA 등 다양한 신경학적 이상 패턴을 분석하는 데 활용된다.

최근 EEG 분석 분야에서는 딥러닝 기반 분류 모델이 높은 성능을 보이고 있으나, 대부분 지도학습(Supervised Learning)에 의존하고 있다. 이러한 접근은 충분한 라벨 데이터가 필요하며, 새로운 EEG 패턴이나 미확인 이상 신호를 발견하는 데에는 한계가 존재한다.

반면 비지도학습(Unsupervised Learning)은 라벨 없이 EEG의 잠재 구조를 탐색하고 이상 패턴을 탐지할 수 있다는 장점을 가진다.

본 연구에서는 EEG 시계열 데이터를 통계적 특징으로 변환한 후 PCA, K-Means Clustering, Isolation Forest를 활용하여 EEG의 잠재 구조를 분석하고 이상 신호를 탐지하는 경량 분석 프레임워크를 구축하였다.

---

## 1. 프로젝트 개요

### 연구 목표

1. EEG 잠재 구조 발견
2. 이상 패턴 탐지
3. 해석 가능한 EEG 분석
4. EEG 위험도 기반 조기 경고 프레임워크 구축

---

## 2. 데이터셋

### Dataset

* Source: HMS Harmful Brain Activity Classification
* Format: EEG Parquet Files
* Sampling Rate: 200Hz
* Total EEG Files: 약 17,300개

### EEG 채널 구성

```text
Fp1, F3, C3, P3, F7, T3, T5, O1,
Fz, Cz, Pz,
Fp2, F4, C4, P4, F8, T4, T6, O2,
EKG
```

---

## 3. 초기 EEG 탐색 및 Feature Engineering

EEG 원시 시계열 데이터를 직접 분석하는 대신 통계적 Feature로 변환하였다.

### 추출 Feature

각 채널별:

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

### 프로젝트 분석 프레임워크

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/7a3e85ab-8791-4570-97ce-d516d1e15b92" />

---

## 4. 초기 EEG 구조 탐색

### PCA

고차원 EEG Feature Space를 시각적으로 분석하기 위해 PCA를 수행하였다.

<img width="989" height="690" alt="image" src="https://github.com/user-attachments/assets/388223c4-847d-4684-9aaa-f7883fa782ff" />

#### 결과

```text
PC1 = 34.4%
PC2 = 10.6%
PC3 = 10.5%

누적 설명력

PC1 + PC2 = 45.0%
PC1 + PC2 + PC3 = 55.5%
```

#### 해석

PCA 결과 EEG 데이터는 하나의 균일한 집합이 아니라 여러 개의 잠재 구조를 형성하고 있음을 확인하였다.

---

### Initial Isolation Forest

<img width="807" height="603" alt="image" src="https://github.com/user-attachments/assets/1b89930f-ca9d-4b4a-983f-4fc243d4fa15" />

---

## 5. Artifact 발견 및 제거

초기 Isolation Forest 분석 결과, 상위 이상치 후보들이 실제 발작 EEG가 아닌 비정상적인 진폭 포화 신호임을 확인하였다.

대표적인 특징은 다음과 같다.

* 수만 단위 진폭
* 수직 점프(vertical jump)
* 장시간 포화(saturation)
* 비생리학적 급격한 변화

<img width="570" height="433" alt="image" src="https://github.com/user-attachments/assets/0d28485c-f0cd-48e2-8015-dc3488dc3c02" />

위와 같은 패턴은 실제 EEG보다는 센서 오류 또는 측정 과정에서 발생한 Artifact에 가까운 형태였다.

따라서 Isolation Forest는 EEG 이상 신호뿐 아니라 데이터 품질 문제 역시 탐지하고 있음을 확인하였다.

---

### Artifact 제거

Artifact 제거 기준:

```python
abs(signal) > 5000
```

조건을 만족하는 EEG를 제거하였다.

---

## 6. 정제 데이터 재분석

### Clean Feature Matrix

```text
EEG 수 : 8,739

Feature 수 : 180
```

---

### PCA 재분석

Artifact 제거 후 PCA를 다시 수행하였다.

<img width="989" height="690" alt="image" src="https://github.com/user-attachments/assets/35b85db2-5334-4873-b0f3-fbd7e76bf7b5" />

---

### Isolation Forest 재분석

정제된 Feature Matrix를 대상으로 Isolation Forest를 다시 수행하였다.

<img width="997" height="372" alt="image" src="https://github.com/user-attachments/assets/d6a3f465-cc62-4d52-886f-1360ebb2f42f" />

초기 분석에서 관찰되었던 비생리학적 포화 신호 대신 주기성과 진폭 변화를 가지는 EEG 패턴이 이상치로 탐지되었다.

이는 데이터 정제가 실제 EEG 기반 이상 패턴 탐지에 중요한 영향을 미쳤음을 보여준다.

---

### EKG 영향 분석

초기 분석 과정에서 EKG 채널은 EEG 채널보다 큰 진폭을 가지는 경우가 많았다.

그러나 Artifact 제거 이후에도 Isolation Forest가 의미 있는 EEG 이상 패턴을 탐지하는 것을 확인하였다.

따라서 최종 이상 탐지 결과는 EKG 단일 채널에 의해 지배되지 않았음을 확인하였다.

---

## 7. 이상 패턴 정량 분석

### Anomaly vs Normal 비교

Isolation Forest 결과를 이용하여 EEG를 Anomaly와 Normal 그룹으로 분리하였다.

<img width="877" height="545" alt="image" src="https://github.com/user-attachments/assets/0d949942-e055-4d36-91d9-f4463de0732f" />

### 주요 결과

Anomaly EEG는 다음 특징을 보였다.

* Range(진폭) 증가
* Standard Deviation(변동성) 증가
* Maximum Value(최댓값) 증가

### 해석

이상 EEG는 정상 EEG보다 더 큰 진폭과 변동성을 가지는 경향을 보였다.

특히 특정 채널이 아닌 전 채널에서 유사한 패턴이 관찰되었다.

---

## 8. 클러스터 해석

### Cluster 결과

| Cluster   | Count |
| --------- | ----: |
| Cluster 1 |  5678 |
| Cluster 0 |  2523 |
| Cluster 2 |   538 |

---

### Cluster 특징 분석

#### Cluster 1

* 저진폭
* 저변동성
* Anomaly 비율 약 0.6%

#### Cluster 0

* 중간 수준 진폭
* 중간 수준 변동성
* Anomaly 비율 약 2.3%

#### Cluster 2

* 매우 높은 Range
* 매우 높은 Standard Deviation
* Anomaly 비율 약 64.1%

```text
고진폭 · 고변동성 EEG 그룹
```

<img width="859" height="508" alt="image" src="https://github.com/user-attachments/assets/d5aca0b2-5fd3-4ac9-a5d8-d81e46ce54a2" />

### 해석

Cluster 2는 Isolation Forest가 탐지한 이상 EEG가 집중적으로 포함된 그룹으로 나타났다.

이는 특정 EEG 패턴 그룹이 이상 신호와 밀접하게 연관될 수 있음을 시사한다.

---

## 9. Expert Label 기반 검증

비지도학습 결과를 전문가 라벨과 비교하여 검증하였다.

### Anomaly 비율 비교

| Label   | Anomaly 비율 |
| ------- | ---------: |
| Seizure |       9.9% |
| Other   |       5.4% |
| GPD     |       5.3% |
| LRDA    |       3.4% |
| GRDA    |       3.3% |
| LPD     |       2.6% |

### 해석

Seizure EEG는 다른 EEG 유형보다 높은 비율로 이상치로 탐지되었다.

이는 Isolation Forest가 비지도 환경에서도 발작 관련 EEG 패턴을 상대적으로 특이하게 인식하고 있음을 의미한다.

---

## 10. EEG 위험도 기반 조기 경고 프레임워크

### 위험도 정의

| Risk Level | Condition        |
| ---------- | ---------------- |
| Normal     | score > 0.10     |
| Warning    | 0 ≤ score ≤ 0.10 |
| High Risk  | score < 0        |

---

### 결과

| Risk Level | Count |
| ---------- | ----: |
| Normal     |  7027 |
| Warning    |  1275 |
| High Risk  |   437 |

---

### Seizure 위험도 분석

| Risk Level | Ratio |
| ---------- | ----: |
| High Risk  |  9.9% |
| Warning    | 23.2% |
| Normal     | 66.9% |

---

### 해석

Seizure EEG의 약 33%가 Warning 이상으로 탐지되었다.

이는 Anomaly Score가 EEG 이상 신호 조기 감지를 위한 위험도 지표로 활용될 가능성을 보여준다.

---

## 11. 한계 분석

본 연구는 EEG 데이터의 잠재 구조 탐색과 이상 패턴 탐지를 목표로 수행되었으며, 의미 있는 결과를 도출하였다. 그러나 다음과 같은 한계점이 존재한다.

---

### 통계 Feature 기반 분석의 한계

본 연구에서는 EEG 시계열 데이터를 Mean, Standard Deviation, Range, Skewness 등의 통계 Feature로 변환하여 분석하였다.

이러한 접근은 계산 효율성이 높고 해석이 용이하다는 장점이 있으나, EEG의 시간적 변화(Temporal Dynamics)와 주파수 특성을 충분히 반영하지 못한다는 한계가 있다.

따라서 동일한 통계값을 가지더라도 서로 다른 파형 구조를 갖는 EEG를 구분하지 못할 가능성이 존재한다.

---

### Frequency Domain 정보 미활용

EEG 분석에서는 Delta, Theta, Alpha, Beta, Gamma 대역의 주파수 특성이 중요한 의미를 가진다.

그러나 본 연구에서는 시간 영역(Time Domain) 기반 Feature만 사용하였기 때문에 주파수 영역에서 나타나는 이상 패턴은 충분히 반영되지 못하였다.

향후 FFT(Fast Fourier Transform) 및 Band Power 기반 Feature를 추가한다면 보다 정교한 EEG 구조 분석이 가능할 것으로 기대된다.

---

## 12. 최종 결론

본 연구에서는 EEG 데이터를 통계적 Feature로 변환한 후 비지도학습 기반 EEG 분석 프레임워크를 구축하였다.

특히 초기 이상치 탐색 과정에서 데이터 내 Artifact를 발견하였으며, 파형 검토를 통해 Artifact 제거 기준을 수립하였다.

이후 정제된 데이터에 대해 재분석을 수행한 결과 EEG의 잠재 구조를 확인하고 고위험 EEG 그룹을 탐지할 수 있었다.

또한 Expert Label 비교를 통해 비지도학습 결과의 해석 가능성을 검증하였으며, 최종적으로 EEG 위험도 기반 조기 경고 프레임워크를 제안하였다.

---

## 13. 프로젝트 의의

* EEG 잠재 구조 탐색
* 비지도 이상 탐지
* Expert Label 기반 검증
* EEG 위험도 평가
* Early Warning Framework 제안

을 하나의 통합 분석 파이프라인으로 수행하였다는 점에서 의의를 가진다.

---

## 14. 기존 연구와의 차별성

기존 EEG 연구는 대부분 지도학습 기반 분류 정확도 향상에 초점을 맞춘다.

반면 본 연구는 라벨 없이 EEG의 잠재 구조를 발견하고 이상 패턴을 탐지하는 데 집중하였다.

또한 Artifact 발견 → 데이터 정제 → 재분석 → 위험도 평가까지 수행하였다는 점에서 차별성을 가진다.

---

## 15. 추후 발전 방향

프로젝트의 한계를 개선하기 위해 개선해야 할 부분을 정리하였다.

---

### Frequency 기반 EEG Feature 확장

본 연구는 시간 영역(Time Domain) Feature를 중심으로 진행되었다.

향후에는 FFT(Fast Fourier Transform)를 활용하여 다음과 같은 주파수 기반 Feature를 추가할 수 있다.

* Delta / Theta / Alpha / Beta / Gamma Band Power
* Relative Band Power
* Spectral Entropy
* Spectral Edge Frequency

이를 통해 EEG의 주파수 특성을 반영한 보다 정교한 구조 분석이 가능할 것으로 기대된다.

---

### 실시간 Early Warning 시스템 개발

본 연구에서는 Anomaly Score를 기반으로 위험도를 정의하였다.

향후에는 실시간 EEG 스트리밍 데이터를 입력받아 위험도를 지속적으로 계산하는 Online Monitoring System으로 확장할 수 있다.

예를 들어,

```text
EEG 입력 → Feature 추출 → Anomaly Score 계산 → 위험도 평가 → 경고 발생
```

과 같은 파이프라인을 구축함으로써 실시간 발작 전조 탐지(Early Warning System)에 활용할 수 있을 것으로 기대된다.
