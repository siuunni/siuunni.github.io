---
title: "서울시 미세먼지 시공간 예측 (LSTM·G-LSTM)"
date: 2025-12-03
weight: 100
draft: true
featured: true
math: true
image:
  preview_only: true
tags:
  - Spatiotemporal Data
  - Moran's I
  - LSTM
  - G-LSTM
  - Forecasting
---

<div style="display:grid;grid-template-columns:130px 1fr;gap:1rem 1.5rem;margin-bottom:2rem;">

<div style="font-weight:700;">개요</div>
<div>서울시 25개 자치구의 미세먼지(PM10·PM2.5)를 시공간적으로 분석하고, 오염물질의 시간·공간 상관 구조에 맞춰 LSTM·G-LSTM을 적용해 단기 예측한 프로젝트입니다.</div>

<div style="font-weight:700;">과정</div>
<div>2025 하계방학 자료분석 (개인 분석)</div>

<div style="font-weight:700;">데이터</div>
<div>서울 Open Data Plaza 대기오염 — SO₂·NO₂·CO·O₃·PM10·PM2.5, 1시간 단위, 2017–2019, 25개 자치구 측정소 + 기상청 AWS 날씨</div>

<div style="font-weight:700;">기술 스택</div>
<div>Python · pandas · NumPy · PyTorch · scikit-learn · statsmodels · Folium</div>

<div style="font-weight:700;">성과</div>
<div>1일 후 예측 기준 <strong>PM10 MAE 14.90</strong>, <strong>PM2.5 MAE 8.97</strong> 달성</div>

</div>

<!--more-->

---

## 분석 목적

서울시 대기오염(PM10·PM2.5)의 **시공간적 특성을 체계적으로 분석**하고, 2017–2018년 자료를 기반으로 **2019년 1일·2일·3일 후 농도를 예측**하는 단기 예측 모형을 탐색했습니다. 고농도 미세먼지를 조기 탐지해 정책·시민 대응에 기여하는 것이 목표입니다.

---

## 데이터 EDA

### 결측치·이상치 탐색

- 0 또는 음수 관측값, 오염물질별 기준 지표의 2배 이상인 값, 2019년 이후 나타난 `985` 같은 불가능한 값을 모두 **결측치(NA)로 처리**했습니다.
- 각 변수의 결측 발생 일수와 결측 개수가 같아, 동일 일자에 모든 변수가 함께 관측되지 않음을 확인했습니다.

### 상관관계 분석

미세먼지는 다른 오염물질과는 상관이 있지만 **날씨와의 상관은 약했습니다.** 이는 단순 선형 관계로는 설명이 어려워, **날씨를 함께 고려한 비선형 모델이 필요**함을 시사합니다.

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/seoul_corr.png"
       alt="오염물질·날씨 상관관계 히트맵"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    오염물질 간(좌)·날씨 간(중)·오염물질×날씨(우) 상관관계. 미세먼지–날씨 상관이 약해 비선형 모델이 필요합니다.
  </figcaption>
</figure>

### 시간 상관성 — 계절성 & ACF/PACF

PM10·PM2.5 모두 **약 3개월 주기의 계절적 변동**이 나타났고, ACF/PACF에서 **lag 30~35일(특히 33일 전) 시점의 유의한 양의 상관**이 확인됐습니다.

<figure style="margin:1.5rem 0;">
  <div style="display:grid;grid-template-columns:repeat(2,1fr);gap:.75rem;align-items:start;">
    <img src="/uploads/papers/seoul_seasonality.png" alt="계절성 시계열"
         style="display:block;width:100%;height:auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;box-sizing:border-box;">
    <img src="/uploads/papers/seoul_acf_pacf.png" alt="ACF·PACF"
         style="display:block;width:100%;height:auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;box-sizing:border-box;">
  </div>
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    계절성(좌): 약 3개월 주기 변동. ACF/PACF(우): 33일 전 시점의 영향이 뚜렷.
  </figcaption>
</figure>

### 공간 상관성 — Moran's I

거리 기반 공간 가중치($w_{ij}=1$ if $d(i,j)\le d_0$)로 **Moran's I**를 계산해 인접 지역 간 공간 군집성을 측정했습니다.

$$I = \frac{N}{\sum_i \sum_j w_{ij}} \cdot \frac{\sum_i \sum_j w_{ij}(y_i - \bar{y})(y_j - \bar{y})}{\sum_i (y_i - \bar{y})^2}$$

| 오염물질 | 시기 | Moran's I | 해석 |
|:---|:---|:---:|:---|
| **PM10** | 2019 봄 | 0.092 | 전반적으로 낮으나 2019년 봄에 공간 군집성 두드러짐 |
| **PM2.5** | 2017 여름 | 0.143 | 2017년 공간 군집성 → 2018년 이후 약화, 2019년엔 거의 사라짐 |

→ **PM10은 시간 상관이 강하고, PM2.5는 시간+공간 상관이 함께 존재**한다는 핵심 인사이트를 도출했습니다.

---

## 데이터 전처리

- **일평균:** 각 일자의 75% 이상 데이터가 있을 때만 일평균을 계산했습니다.
- **공간 보간:** 날짜별 결측을 **IDW(역거리가중)** 로 인접 3~5개 측정소의 거리 기반 가중평균으로 채웠습니다. 풍향(deg)은 벡터(cos·sin)로 변환 후 보간해 각도로 복원했습니다.
- **시간 보간:** 남은 결측은 선형 보간, 앞뒤 값이 없으면 ffill/bfill로 처리했습니다.
- **파생·날씨 변수:** 계절을 1·2·3·4로 인코딩하고, 기상청 AWS의 기온·풍속·풍향·강수량·기압·습도를 추가해 비선형 예측력을 높였습니다.

---

## 모델 — 오염물질 특성에 맞춘 선택

EDA에서 드러난 시공간 상관 구조에 따라 **오염물질별로 다른 모델**을 적용했습니다.

| | PM10 | PM2.5 |
|:---|:---|:---|
| **상관 구조** | 시간 상관 강함, 공간 군집성 약함 | 시간 + 공간 상관 모두 강함 |
| **모델** | **LSTM** (시계열 패턴) | **G-LSTM** (시공간 구조 반영) |

**G-LSTM의 핵심 — 학습되는 공간 인접 행렬.** 일반 LSTM의 게이트에 **파라미터화된 인접 행렬 $W_{adj}$** 를 곱해, 측정소 간 공간 상호작용을 데이터로부터 직접 학습하게 했습니다.

$$C_t = I_t \odot U_t + F_t \odot (C_{t-1} W_{adj})$$

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/seoul_glstm.png"
       alt="G-LSTM 구조"
       style="width:80%;display:block;margin:0 auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    G-LSTM 셀 구조 — 인접 행렬 $W_{adj}$ 로 측정소 간 공간 의존성을 학습
  </figcaption>
</figure>

---

## 실험 결과

### PM10 (LSTM) · PM2.5 (G-LSTM)

| 모델 | 1일차 MAE | 1일차 RMSE | 2일차 MAE | 3일차 MAE |
|:---|:---:|:---:|:---:|:---:|
| **PM10 (LSTM)** | **14.90** | 32.14 | 16.71 | 16.76 |
| **PM2.5 (G-LSTM)** | **8.97** | 14.13 | 10.25 | 10.47 |

예측이 잘 된 지역은 실제 농도를 잘 따라갔고, 급격한 고농도 스파이크가 잦은 일부 지역에서는 오차가 컸습니다.

<figure style="margin:1.5rem 0;">
  <div style="display:grid;grid-template-columns:repeat(2,1fr);gap:.75rem;align-items:start;">
    <img src="/uploads/papers/seoul_lstm_result.png" alt="LSTM PM10 예측"
         style="display:block;width:100%;height:auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;box-sizing:border-box;">
    <img src="/uploads/papers/seoul_glstm_result.png" alt="G-LSTM PM2.5 예측"
         style="display:block;width:100%;height:auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;box-sizing:border-box;">
  </div>
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    실제(True) vs 예측(Predict). 좌: PM10(LSTM), 우: PM2.5(G-LSTM)
  </figcaption>
</figure>

### 공간 영향력 시각화

G-LSTM이 학습한 인접 행렬을 지도에 그린 결과, **교통량이 많은 강남·강북 지역을 중심으로 주변 지역에 영향력**이 퍼지는 구조를 확인했습니다.

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/seoul_influence.png"
       alt="G-LSTM 공간 영향력 지도"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    주변 지역에 영향을 주는 정도(좌)와 받는 정도(우). 강남·강북 중심으로 영향력이 확산됩니다.
  </figcaption>
</figure>

---

## 결론

- 서울시 미세먼지의 **시공간 특성을 분석**하고, 2017–2018 기반 2019년 단기(1·2·3일) 예측을 수행했습니다.
- EDA·공간통계로 **PM10은 시간 상관, PM2.5는 시간+공간 상관**이 함께 존재함을 확인하고, 이에 맞춰 **PM10 → LSTM, PM2.5 → G-LSTM** 을 적용했습니다.
- 예측 결과 1일 후 기준 **PM10 MAE 14.90, PM2.5 MAE 8.97** 로, 고농도 미세먼지 조기 탐지 가능성을 입증했습니다.
