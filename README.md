#  통신사 고객 이탈 예측 시스템

## 1. 프로젝트 개요

### 1.1. 개발 배경 및 필요성

구독형 비즈니스 모델(통신, OTT 등)에서 신규 고객 유치 비용은 기존 고객 유지 비용보다 약 **5~25배 더 높습니다.** 따라서 기업의 수익성을 위해 기존 고객의 **이탈(Churn)을 사전에 방지**하는 것이 필수적입니다.

본 프로젝트는 머신러닝을 활용해 이탈 가능성이 높은 고객을 사전에 선별하고, 그 원인을 분석하여 선제적인 마케팅 대응을 가능하게 하는 시스템을 구축하고자 했습니다.

### 1.2. 개발 목표

* **예측 모델 구축:** 고객 데이터를 분석하여 높은 정확도와 **재현율(Recall)**을 가진 이탈 예측 모델 개발.
* **설명 가능한 AI (XAI):** 단순 예측을 넘어, 해당 고객이 **'왜' 이탈하려고 하는지** 주요 위험/방어 요인을 시각화.
* **웹 서비스 구현:** 비개발자(마케터 등)도 쉽게 사용할 수 있도록 **FastAPI와 직관적인 Web UI(슬라이더 등)** 연동.

---

## 2. 사용 기술 및 개발 환경

| 구분 | 상세 기술 | 비고 |
| :--- | :--- | :--- |
| **Language** | Python 3.9 | 데이터 분석 및 백엔드 개발 |
| **Model & Data** | Scikit-learn, Pandas, Imbalanced-learn | 모델 학습, 전처리, SMOTE |
| **Backend** | FastAPI, Uvicorn | RESTful API 서버 구축, Pydantic 데이터 검증 |
| **Frontend** | HTML5, Bootstrap 5, jQuery (AJAX) | 반응형 UI, 비동기 통신 |
| **Environment** | VS Code, Anaconda | 개발 IDE 및 가상환경 |

---

## 3. 데이터 분석 및 전처리 (Data & Preprocessing)

### 3.1. 데이터셋 개요

* **출처:** Kaggle Telco Customer Churn Dataset
* **규모:** 약 7,000건의 고객 데이터, 20개의 특징(Feature)
* **주요 특징:** 가입 기간(Tenure), 월 요금(MonthlyCharges), 계약 형태(Contract), 인터넷 서비스 종류 등.

### 3.2. 데이터 전처리 및 특징 공학 (Feature Engineering)

모델의 성능 향상을 위해 다음과 같은 고도화된 전처리 과정을 **Scikit-learn Pipeline**으로 구축했습니다.

* **결측치 및 이상치 처리:** `TotalCharges`의 빈 값을 수치형으로 변환 및 보간 처리.
* **수치형 데이터 구간화 (Binning):** `tenure`(가입 기간) 등 선형적이지 않은 데이터의 특성을 반영하기 위해 `KBinsDiscretizer`를 사용하여 데이터를 5개 구간으로 범주화하여 정확도 향상.
* **데이터 불균형 해소 (SMOTE):** 이탈(Yes) 데이터가 적은 불균형(Imbalance) 문제를 해결하기 위해, 학습 데이터에 한해 **SMOTE** (Synthetic Minority Over-sampling Technique) 기법을 적용. 이를 통해 소수 클래스인 '이탈 고객'에 대한 **재현율(Recall)**을 대폭 개선함.
* **스케일링 및 인코딩:** `StandardScaler`와 `OneHotEncoder`를 적용하여 수치형/범주형 데이터를 모델에 최적화.

---

## 4. 모델링 및 성능 평가

### 4.1. 비교 모델 선정

최적의 모델을 찾기 위해 다음과 같은 시나리오로 모델을 학습하고 비교했습니다.

* **Model A:** Logistic Regression (기본)
* **Model B:** Logistic Regression + 구간화(Binning) + SMOTE 적용
* **Model C:** Random Forest (비선형 트리 모델)

### 4.2. 성능 평가 결과

불균형 데이터셋임을 고려하여 단순 정확도(Accuracy)보다는 **Recall(재현율)**과 F1-Score, AUC를 핵심 지표로 선정했습니다.

* **최종 선정 모델:** **Model B (Logistic Regression + Binning + SMOTE)**
* **선정 사유:**
    1. Random Forest와 대등한 성능을 보이면서도, **'예측 요인(Feature Importance)'을 해석하기가 용이함.**
    2. SMOTE 적용 후, 실제 이탈자를 찾아내는 능력인 **Recall(재현율)이 0.XX에서 0.XX로 크게 향상됨.**
 
---

## 5. 시스템 구현 (System Implementation)



### 5.1. 백엔드 API (FastAPI)

* **모델 로딩:** 학습된 Pipeline 객체를 `.pkl` 파일로 직렬화하여 서버 기동 시 로드.
* **`/predict` 엔드포인트:** 프론트엔드로부터 JSON 데이터를 받아 예측 수행.
* **요인 분석 알고리즘:** 모델의 가중치(Coefficient)와 입력값을 연산하여, 이탈에 기여한 **상위 3개 위험 요인(Risk Factors)**과 **방어 요인(Safe Factors)**을 추출하여 반환.
* **CORS 설정:** 로컬 및 외부 웹 환경에서의 접근 허용.

### 5.2. 프론트엔드 UI (Web)

* **UX 중심 설계:** 수치 입력 오류를 방지하기 위해 `tenure`, `MonthlyCharges` 등을 **Range Slider**로 구현하여 직관적인 조작 제공.
* **실시간성:** **AJAX 비동기 통신**을 통해 페이지 새로고침 없이 즉각적인 예측 결과 확인 가능.
* **동적 시각화:** 예측 결과(이탈/유지)에 따라 UI 색상이 변경되며(Red/Green), 주요 원인 분석 리포트를 동적으로 생성하여 표시.

---

## 6. 결론 및 기대 효과

### 6.1. 프로젝트 성과

본 프로젝트를 통해 단순히 머신러닝 모델을 만드는 것을 넘어, **데이터 전처리 → 모델 학습 → API 배포 → 웹 서비스 연동**으로 이어지는 **AI 풀스택(Full-stack) 파이프라인**을 성공적으로 구축했습니다. 특히 **SMOTE와 구간화**를 통해 불균형 데이터 문제를 해결하고 모델 성능을 실용적인 수준으로 끌어올렸습니다.

### 6.2. 비즈니스 기대 효과

이 시스템을 도입할 경우, 마케터는 복잡한 데이터 분석 없이 웹 접속만으로 이탈 위험 고객을 식별할 수 있습니다. 특히 **"왜 이탈 위험이 높은지"**에 대한 근거(예: "월별 계약이라서 위험함")를 제공받음으로써, 해당 고객에게 맞춤형 프로모션(예: 연간 계약 전환 할인)을 제안하여 이탈률을 효과적으로 낮출 수 있을 것으로 기대됩니다.

### 6.3. 향후 발전 방향

* **XGBoost/LightGBM 도입:** 더 높은 예측 성능을 위해 부스팅 계열 모델 도입 검토.
* **하이퍼파라미터 튜닝 자동화:** `GridSearchCV` 등을 활용하여 최적의 파라미터를 주기적으로 갱신하는 **MLOps 체계 구축**.

---
시연 영상 링크 : https://youtube.com/shorts/YIB0EfdCbVs?feature=share
