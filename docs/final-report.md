# NeuroStat: EEG 비지도학습 기반 잠재 패턴 및 이상 신호 탐색

## 1. 연구 배경

뇌파(Electroencephalography, EEG)는 뇌의 전기적 활동을 기록하는 생체신호로, 간질(Seizure), 주기성 방전(Periodic Discharge), 리듬성 이상파(Rhythmic Delta Activity) 등 다양한 신경학적 상태를 반영한다.

기존 EEG 분석 연구는 대부분 전문가가 부여한 라벨을 활용한 지도학습(Classification)에 집중되어 있다. 그러나 실제 의료 데이터에서는 라벨링 비용이 높고, 전문가 간 해석 차이가 존재하며, 알려지지 않은 패턴 또한 포함될 수 있다.

따라서 본 연구에서는 라벨 정보를 직접 학습에 사용하지 않고, EEG 신호 자체의 통계적 특성만을 이용하여 잠재 구조(latent structure)를 탐색하고 이상 신호(outlier)를 탐지하는 비지도학습 접근법을 적용하였다.

---

## 2. 연구 목표

본 연구의 목표는 다음과 같다.

1. EEG 신호로부터 통계적 특징(feature)을 추출한다.
2. 고차원 EEG 데이터를 저차원 공간으로 투영하여 구조를 시각화한다.
3. 군집화(K-Means)를 통해 EEG 잠재 패턴을 탐색한다.
4. 이상치 탐지를 통해 비정상적인 EEG 신호를 식별한다.
5. 전문가 라벨과 비교하여 비지도학습 결과의 의미를 분석한다.

---

## 3. 데이터셋

### HMS Harmful Brain Activity Classification Dataset

본 연구에서는 Kaggle의 HMS Harmful Brain Activity Classification 데이터를 활용하였다.

데이터는 다채널 EEG 신호와 전문가 판독 결과를 포함하고 있으며 다음 6개 클래스로 구성된다.

* Seizure
* GPD
* LPD
* GRDA
* LRDA
* Other

### 데이터 규모

* 원본 EEG 라벨 데이터: 약 106,800개
* EEG 파일 수: 약 17,300개
* EEG 채널 수: 19개

---

## 4. 데이터 전처리

### Feature Extraction

각 EEG 파일의 19개 채널에 대해 다음 통계 특성을 추출하였다.

#### 기본 통계량

* Mean
* Standard Deviation
* Minimum
* Maximum
* Range

#### 분포 특성

* Skewness
* Kurtosis

#### 추가 특성

* Absolute Mean

총 180개의 Feature를 생성하였다.

---

### Artifact 제거

EEG 데이터에는 측정 오류나 전극 문제로 인해 비정상적으로 큰 값이 포함될 수 있다.

이를 제거하기 위해:

* 절대 진폭이 5000 이상인 EEG 제거
* 거의 변동이 없는 Flat Signal 제거

를 수행하였다.

---

## 5. 차원 축소 (PCA)

추출된 Feature Matrix는 180차원의 고차원 데이터로 구성된다.

이를 시각적으로 분석하기 위해 PCA(Principal Component Analysis)를 적용하였다.

### PCA 결과

설명 분산 비율(Explained Variance Ratio)

* PC1: 34.45%
* PC2: 10.57%
* PC3: 10.52%

누적 설명력

* Top 3 PCs: 55.54%
* Top 10 PCs: 약 76%

### 결과 해석

PCA 시각화 결과 EEG 데이터가 하나의 연속적인 공간이 아닌 여러 밀집 영역으로 구성되어 있음을 확인하였다.

이는 EEG 데이터 내부에 서로 다른 잠재 패턴이 존재함을 의미하며, 비지도 군집화 가능성을 시사한다.

[Figure 1 삽입: PCA Scatter Plot]

---

## 6. 군집 분석

### K-Means Clustering

PCA 결과를 바탕으로 K-Means 군집화를 수행하였다.

군집화의 목적은 전문가 라벨을 사용하지 않고 EEG 내부에 존재하는 자연스러운 그룹을 찾는 것이다.

### 분석 결과

군집별 전문가 라벨 분포를 확인한 결과:

* 특정 군집에서는 Seizure 비율이 높게 나타남
* 특정 군집에서는 GPD, LPD가 집중됨
* 일부 군집은 Other가 우세

라는 경향을 확인하였다.

이는 비지도학습만으로도 EEG 병리 패턴이 부분적으로 분리될 수 있음을 보여준다.

[Figure 2 삽입: Cluster Visualization]

[Figure 3 삽입: Cluster별 Expert Consensus 분포]

---

## 7. 이상치 탐지

### 목적

일반적인 EEG 패턴에서 크게 벗어나는 신호를 탐지하기 위해 이상치 탐지를 수행하였다.

### 방법

PCA 공간 및 군집 정보를 기반으로 Outlier Score를 계산하였다.

### 결과

상위 이상치 EEG를 직접 확인한 결과:

* 극단적인 진폭 변화
* 채널 전체 포화(Saturation)
* Flat Signal
* Artifact

등이 다수 발견되었다.

이는 비지도학습 기반 이상치 탐지가 실제 데이터 품질 문제를 효과적으로 식별할 수 있음을 의미한다.

[Figure 4 삽입: Top Outlier EEG Example]

---

## 8. 결과 및 의의

본 연구에서는 EEG 신호로부터 통계적 특징을 추출하고 비지도학습을 적용하여 잠재 구조를 탐색하였다.

주요 결과는 다음과 같다.

* 180개 Feature를 기반으로 EEG를 수치화
* PCA를 통해 데이터 구조 시각화
* K-Means를 통해 잠재 군집 발견
* 전문가 라벨과 군집 간 의미 있는 연관성 확인
* 이상치 탐지를 통해 Artifact EEG 식별

특히 지도학습 없이도 EEG 데이터 내부의 병리학적 구조를 일부 발견할 수 있음을 확인하였다.

---

## 9. 한계점

본 연구는 통계 기반 Feature만을 사용하였다.

따라서:

* 시간적 구조(Time Dependency)
* 주파수 정보(Frequency Information)
* 채널 간 상호작용(Spatial Relationship)

을 충분히 반영하지 못하였다.

또한 이상치 중 일부는 실제 병리 신호가 아닌 측정 Artifact일 가능성이 존재한다.

---

## 10. 향후 연구

향후 연구에서는 다음과 같은 방향으로 확장할 수 있다.

### 1) 주파수 영역 Feature 추가

* Delta
* Theta
* Alpha
* Beta
* Gamma Power

추출을 통한 성능 향상

### 2) Deep Representation Learning

* Autoencoder
* Variational Autoencoder
* Contrastive Learning

기반 잠재 표현 학습

### 3) Diffusion 기반 이상 탐지

NeuroDiffusion 프로젝트와 연계하여

* Reconstruction Error
* Diffusion Score

를 활용한 EEG 이상 탐지 연구 수행

### 4) 임상 해석 가능성 강화

전문가 라벨과 비지도 군집 결과를 비교하여 새로운 EEG 패턴 발견 가능성을 탐색한다.

---

## 11. 결론

본 연구는 EEG 데이터에 대해 비지도학습 기반 분석을 수행하여 데이터 내부의 잠재 구조와 이상 신호를 탐색하였다.

PCA와 K-Means를 통해 EEG 패턴이 자연스럽게 구분됨을 확인하였으며, 이상치 탐지를 통해 실제 Artifact 신호를 식별할 수 있었다.

이는 비지도학습이 EEG 데이터 분석 및 품질 관리에 활용될 수 있음을 보여주는 결과이며, 향후 Deep Learning 기반 Representation Learning 및 Diffusion Model 연구의 기초가 될 수 있다.
