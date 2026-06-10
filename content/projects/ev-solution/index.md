---
title: "전기차 주행가능거리 기반 충전소 추천 시스템"
date: 2025-12-05
weight: 60
featured: true
math: true
authors:
  - me
  - 장현우
  - 김도현
  - 곽성찬
image:
  preview_only: true
tags:
  - EV
  - BMS
  - LSTM
  - Range Estimation
  - Recommendation
---

<div style="display:grid;grid-template-columns:130px 1fr;gap:1rem 1.5rem;margin-bottom:2rem;">

<div style="font-weight:700;">개요</div>
<div>전기차의 실시간 <strong>BMS(배터리 관리 시스템)</strong> 데이터로 주행가능거리를 예측하고, 실제 주행 경로 위에서 도달 가능한 충전소를 실시간 추천하는 시스템입니다.</div>

<div style="font-weight:700;">기간</div>
<div>알파프로젝트 — 팀 N4 (2024.3 – 2024.6)</div>

<div style="font-weight:700;">담당 역할</div>
<div><strong>주행가능거리 예측 모델링 · 경로 추천 알고리즘 구현 · GUI 개발</strong></div>

<div style="font-weight:700;">데이터</div>
<div>전기차(BMW i3) BMS 주행 로그 — 배터리 전압·전류, SoC, 속도, 거리, 온도 등</div>

<div style="font-weight:700;">기술 스택</div>
<div>Python · pandas · NumPy · LSTM(TensorFlow/Keras) · Selenium · BeautifulSoup · Naver Map API · Arduino·ESP(IoT)</div>

<div style="font-weight:700;">성과</div>
<div>주행가능거리 예측 <strong>MAE 0.03 · MSE 0.005</strong> 달성</div>

</div>

<!--more-->

---

## 문제 정의

전기차 운전자는 "지금 배터리로 어디까지 갈 수 있고, 그 안에 충전소가 있는가"를 늘 신경 써야 합니다. 여기엔 두 가지 어려움이 있었습니다.

- **배터리 잔량만으로는 주행가능거리 예측이 어렵다.** 동일한 SoC라도 주행 환경·운전 습관·도로 조건에 따라 실제 갈 수 있는 거리가 크게 달라집니다.
- **방향 정보가 없으면 엉뚱한 충전소를 추천한다.** 실시간 GPS·경로 정보가 없으면, 주행 방향과 반대편에 있는 충전소를 추천하게 됩니다.

그래서 **① BMS 데이터로 주행가능거리를 예측 → ② 실제 주행 경로 위에서 도달 가능한 충전소를 추천**하는 흐름을 설계했습니다.

---

## 1. 주행가능거리 — 1차 계산 

BMS에서 들어오는 **배터리 전압 및 전류**로 실시간 전력을 구하고, 누적 에너지 소비와 회생제동(에너지 회수)까지 반영해 주행가능거리를 계산했습니다. 배터리 전류가 음수면 회생제동으로 에너지를 회수하는 상태로 간주했습니다.

$$P_{[kW]} = \frac{V_{\text{batt}} \times I_{\text{batt}}}{1000}, \qquad \text{소비율}_{[kWh/km]} = \frac{E_{[kWh]}}{\text{거리}_{[km]}}$$

$$\text{주행가능거리} = \frac{\dfrac{SoC}{100}\times C_{\text{batt}} + (E_{\text{회수}} \cdot \mathbb{1}[P<0])}{\max(|P|,\,0.1)}$$

(BMW i3 배터리 용량 $C_{\text{batt}} = 42.2\,\text{kWh}$ 기준)

**한계 진단 — SoC 상관분석.** 계산한 주행가능거리가 타당한지 SoC를 비롯한 변수들과의 상관관계로 검증했습니다. 그 결과 **주행가능거리와 SoC의 상관(≈0.35)이 낮았고**, 아래 산점도처럼 같은 배터리 잔량(SoC)에서도 추정 주행거리가 크게 퍼져 **단순 계산식만으로는 한계**가 있음을 확인했습니다.

<figure style="margin:1.5rem 0;">
  <div style="display:grid;grid-template-columns:repeat(2,1fr);gap:.75rem;align-items:start;">
    <img src="/uploads/papers/ev_soc_corr_1.png"
         alt="변수들 간 Heat map"
         style="display:block;width:100%;height:auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;box-sizing:border-box;">
    <img src="/uploads/papers/ev_soc_corr_2.png"
         alt="배터리 잔량(SoC)과 추정 주행거리의 상관계수"
         style="display:block;width:100%;height:auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;box-sizing:border-box;">
  </div>
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    배터리 잔량(SoC)과 추정 주행거리의 상관계수가 0.35로 두 변수간에 선형적인 특징이 매우 약하다는 것을 알 수 있습니다. 
  </figcaption>
</figure>

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/ev_soc_scatter.png"
       alt="배터리 잔량(SoC)과 추정 주행거리의 괴리"
       style="width:80%;display:block;margin:0 auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    배터리 SoC(%) vs 추정 주행거리(km). 같은 잔량에서도 주행거리가 넓게 퍼져 있어, 잔량만으로는 정확한 예측이 어렵다는 것을 보여줍니다.
  </figcaption>
</figure>


---

## 2. 주행가능거리 — 동적 소비율 보정 + LSTM 예측

단순 계산식의 한계를 넘기 위해, 주행 데이터의 변동성과 비선형 패턴을 반영하는 두 단계를 추가했습니다.

- **동적 소비율 보정:** 주행 중 변동성을 반영하도록 **가중 이동 평균 소비율(MAAEC)** 과 **보정식(AAEC)** 을 설계해, 순간적인 가속·회생제동에 휘둘리지 않는 안정적인 소비율을 산출했습니다.
- **LSTM 시계열 예측:** 보정된 소비율 위에 **LSTM 기반 시계열 모델**을 결합해, 주행 환경·운전 습관에 따른 **비선형 주행 패턴**을 학습했습니다.

→ 그 결과 주행가능거리 예측에서 **MAE 0.03, MSE 0.005** 의 높은 성능을 달성해, 1차 계산식의 한계를 크게 개선했습니다.

---

## 3. 경로 기반 충전소 추천

방향을 무시한 추천 문제를 해결하기 위해, **실제 주행 경로**를 반영한 추천 알고리즘을 구현했습니다.

1. **네이버 길찾기 API** 로 현재 위치에서 후보 충전소까지의 실제 주행 경로(도달 거리)를 계산
2. 예측된 **주행가능거리와 연동**해, 도달 가능한 충전소만 필터링
3. 그 안에서 **최소 거리** 충전소를 추천하고, 네이버 지도를 실시간 크롤링한 충전소 정보(개방·충전 속도·이용 가능 대수 등)와 함께 **GUI로 시각화**

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/ev_recommend_gui.png"
       alt="실시간 전기차 충전소 추천 GUI"
       style="width:80%;display:block;margin:0 auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    실시간 충전소 추천 GUI — 현재 위치와 도달 가능한 충전소 위치를 지도에 함께 표시
  </figcaption>
</figure>

> **데이터 수집:** 충전소 현황은 GPS 위치를 기준으로 **네이버 지도를 Selenium·BeautifulSoup으로 크롤링**해 이름·거리·개방 여부·충전 속도·이용 가능 대수를 수집하고 DataHub에 저장했습니다.

---

## 4. 사용자 대시보드 & 데모 시스템

운전자가 한눈에 보도록 **실시간 대시보드**(주행가능거리·배터리 잔량·배터리 온도·주변 충전소 목록)를 구성했습니다. 실제 전기차 대신 **Arduino·ESP 기반 모형 전기차**로 BMS-유사 데이터를 클라우드로 전송해 전체 흐름을 시연했습니다.

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/ev_dashboard_1.png"
       alt="전기차 충전 최적화 대시보드"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
   <img src="/uploads/papers/ev_dashboard_2.png"
       alt="전기차 충전소 추천 대시보드"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    EV charging optimization 및 충전소 추천 대시보드 — 주행가능거리,배터리 잔량,온도,주변 충전소 현황을 실시간 표시
  </figcaption>
</figure>



---

## 핵심 요약

- **예측 모델링:** 단순 계산식의 한계(SoC 상관 ≈0.35)를 진단하고, 동적 소비율 보정(MAAEC·AAEC) + LSTM 시계열 모델로 **MAE 0.03 · MSE 0.005** 달성.
- **경로 추천 알고리즘:** 네이버 길찾기 API로 실제 주행 경로를 계산해, 예측 주행가능거리와 연동한 **방향까지 고려한 충전소 추천**을 구현.
- **End-to-end:** 데이터 수집(크롤링) → 주행가능거리 예측 → 경로 기반 추천 → 대시보드·모형 전기차 데모까지 전 과정을 완성.

---

## 링크

- [GitHub — N4 AIoT Project](https://github.com/statjhw/N4-AIot-Project)
