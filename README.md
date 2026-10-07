##  제조 산업 | Cost-Sensitive Predictive Maintenance

### 센서 데이터 기반 설비 고장 예측 및 유지보수 의사결정 시스템

> **Business Goal:** 설비 고장을 단순히 예측하는 것을 넘어, 고장 미탐지에 따른 생산 중단 비용과 불필요한 점검 비용을 함께 고려하여 실제 유지보수 의사결정을 지원하는 Predictive Maintenance 시스템을 설계했습니다.

### Problem

전체 10,000건의 설비 데이터 중 실제 고장은 약 **3.4%**에 불과해 심각한 클래스 불균형이 존재합니다.

이러한 환경에서는 단순 Accuracy가 높더라도 실제 고장을 놓치는 문제가 발생할 수 있습니다.

특히 제조 현장에서는

- **False Negative:** 실제 고장을 놓쳐 생산라인 중단 발생
- **False Positive:** 정상 설비를 불필요하게 점검

이라는 서로 다른 비용 구조가 존재합니다.

따라서 본 프로젝트에서는 단순 정확도 최대화가 아니라 **고장 탐지 성능과 유지보수 비용 간의 trade-off 최적화**를 핵심 문제로 정의했습니다.

---

### Approach

#### 1. Leakage-Safe Data Pipeline

- 식별자 변수 `UDI`, `Product ID` 제거
- 실제 예측 시점에서 사용할 수 없는 개별 고장 유형 변수 `TWF`, `HDF`, `PWF`, `OSF`, `RNF` 제외
- Train/Test Split 이후 전처리를 수행하도록 Pipeline 구성
- StandardScaler와 PCA가 Test 데이터를 학습하지 않도록 Data Leakage 방지

#### 2. Feature Engineering

높은 상관성을 가진

- Air Temperature
- Process Temperature

센서를 표준화한 뒤 PCA를 적용하여 하나의 온도 주성분으로 변환했습니다.

PCA 또한 Train 데이터에서만 학습하도록 구성했습니다.

#### 3. Imbalanced Learning Experiment

단순히 SMOTE를 적용하는 대신 여러 불균형 처리 전략을 동일한 Test Set에서 비교했습니다.

- Baseline
- Class Weight
- SMOTE

이를 통해 데이터 증강 자체를 목적이 아니라 **검증해야 할 모델링 전략**으로 다뤘습니다.

#### 4. Model Benchmarking

다음 모델을 동일한 평가 환경에서 비교했습니다.

- Logistic Regression
- Random Forest
- Gradient Boosting
- LightGBM

클래스 불균형 문제를 고려하여 Accuracy뿐 아니라

- Recall
- Precision
- F1-score
- ROC-AUC
- **PR-AUC**

를 함께 평가했습니다.

특히 희소한 고장 데이터를 평가하기 위해 **PR-AUC를 주요 모델 선정 기준으로 사용했습니다.**

---

### Cost-Sensitive Threshold Optimization

일반적인 분류 모델의 기본 Threshold인 `0.5`를 그대로 사용하지 않았습니다.

제조 현장의 의사결정 구조를 반영하기 위해 다음과 같이 비용 함수를 정의했습니다.

\[
Expected\ Cost
=
FN \times Cost_{failure}
+
FP \times Cost_{inspection}
\]

예시 시나리오:

- 고장 미탐지(False Negative): **$10,000**
- 불필요한 점검(False Positive): **
