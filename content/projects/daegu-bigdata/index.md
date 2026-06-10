---
title: "대구 빅데이터 분석 경진대회: IT/SW 기업 성장성 판단"
date: 2025-12-04
weight: 90
authors:
  - me
  - 이진
  - 고연희
featured: true
image:
  preview_only: true
tags:
  - Data Analysis
  - MissForest
  - PCA
  - Factor Analysis
  - Random Forest
---

<div style="display:grid;grid-template-columns:130px 1fr;gap:1rem 1.5rem;margin-bottom:2rem;">

<div style="font-weight:700;">개요</div>
<div>기업의 성장 잠재력을 정량적으로 파악해 전략적 기업 대출(여신) 기준을 제안한 빅데이터 분석 프로젝트입니다.</div>

<div style="font-weight:700;">대회</div>
<div>제5회 대구 빅데이터 분석 경진대회 — <strong>우수상 수상</strong></div>

<div style="font-weight:700;">데이터</div>
<div>IT/SW기업 생태계 실태조사, 대구·경북 ITSW기업, 현대카드·대구BC카드 매출, 시장금리(KOSIS)</div>

<div style="font-weight:700;">기술 스택</div>
<div>Python · R · pandas · NumPy · scikit-learn · statsmodels · missingpy</div>

<div style="font-weight:700;">성과</div>
<div><strong>Random Forest 회귀 RMSE 0.0269</strong> 달성</div>

</div>

<!--more-->

---

## 문제 정의

대구·경북 지역 예금은행의 기업 대출은 특정 서비스업에 집중되어, 성장 잠재력이 높은 IT/SW·벤처 기업에 자금이 충분히 공급되지 못하는 **자금 배분 비효율** 문제가 있었습니다.

기존 여신 심사는 최근 3개년 재무제표·담보 중심이라 **미래 성장 잠재력을 반영하지 못합니다.** 본 프로젝트는 신용 서류에 드러나지 않는 요인까지 반영해 **성장 잠재력 기반의 새로운 여신 기준**을 제안합니다.

---

## 접근 방법

1. **카드 매출 분석** — 현대카드·대구BC카드 데이터로 IT/비IT 산업의 매출액 증감률을 시장금리와 함께 분석해 산업 성장성을 파악합니다.
2. **성장 업종 추출** — 한국 산업 평균 증감률보다 높은 업종(정보통신업, 전문·과학·기술 서비스업)을 선별합니다.
3. **성장 잠재력 예측 모델** — 기업 데이터를 통합해 *매출액 대비 연구개발비 투자금액* 파생변수를 종속변수로, 성장성 관련 요인을 독립변수로 하는 회귀 모델을 구축합니다.

---

## 카드 데이터 분석 — 산업별 매출 성장성

5년간(2017–2021) 카드 매출을 월별로 집계해 IT/비IT 산업의 매출액 증감률을 시장금리 추세와 비교했습니다. 팬데믹 이후 두 산업 모두 매출이 감소했으나, **비IT 산업(관광·여가)의 감소폭이 더 컸습니다.**

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/daegu_sales_growth.png"
       alt="IT산업과 비IT산업의 매출액 증감률 및 시장금리 추세"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    IT산업·비IT산업의 매출액 증감률과 시장금리 추세 (2017–2021)
  </figcaption>
</figure>

---

## 데이터 전처리 & 파생변수

- **데이터 통합:** 사업자등록번호를 기준으로 여러 기업 데이터를 병합하고, 범주형·논리형 자료를 정수형으로 처리했습니다.
- **결측치 대치:** 무응답·결측이 많은 변수를 **MissForest**(Random Forest 기반 반복 대치)로 보완했습니다.
- **종속변수 설계:** 기업가치평가의 가중평균자본비용(WACC) 개념을 차용해, *매출액 대비 연구개발비 투자금액*의 연도별 변화율에 가중치(0.4 / 0.6)를 부여한 성장성 지표를 만들었습니다.
- **독립변수:** 기업부설 연구소 유무, 특허 등록·출원 건수, SW 융합 기술 분야별 시장 전망 등 신용 서류에 없는 변수를 활용했습니다.

---

## 차원 축소 & 적합성 검정

변수 구조를 단순화하기 위해 **주성분 분석(PCA)** 과 **요인 분석**을 수행했습니다. 누적 분산 설명력이 약 99%에 도달하는 지점을 참고해 차원을 결정했습니다.

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/daegu_pca_scree.png"
       alt="PCA scree plot과 누적 분산 설명력"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    PCA scree plot(좌)과 누적 분산 설명력(우)
  </figcaption>
</figure>

요인 분석 적합성은 **Bartlett 검정**(p-value ≈ 0.0)과 **KMO 검정**(≈ 0.66)으로 확인해, 요인 분석이 적합하다는 결론을 얻었습니다. 요인 적재량을 기준으로 특허 관련, 데이터 산업, 기기·차량 산업, 연구소, 의료 산업 등 **5개의 해석 가능한 feature**를 도출했습니다.

---

## 회귀 모델

도출한 5개 feature를 독립변수로, *매출액 대비 연구개발비 투자금액*을 종속변수로 하여 여러 회귀 방법을 비교했습니다. **Random Forest 회귀**가 가장 우수했습니다.

| n_estimators | max_features | max_depth | RMSE |
|:---:|:---:|:---:|:---:|
| 10 | 2 | 10 | 0.0292 |
| **10** | **2** | **30** | **0.0269** |
| 1000 | 2 | 50 | 0.0306 |

최적 모델은 검증 데이터에서 **RMSE 약 0.0269** 로 가장 낮은 오차를 보였습니다.

---

## 기대 효과

- **객관적 성장성 평가:** 재무 상태가 약하지만 성장 잠재력이 높은 기업을 정량 지표로 발굴해, 은행이 투명·안전하게 자금을 공급할 수 있습니다.
- **지역 경제 활성화:** R&D 프로젝트의 잠재 가치를 반영한 기업 맞춤형 대출로 산업 발전과 혁신을 촉진합니다.
- **윈-윈 구조:** 기업은 자금을 확보해 성장하고, 은행은 신규 고객 확보·신용평가 개선·장기 투자 수익을 얻습니다.

---


- [Presentation (PDF)](/uploads/awards/daegu-bigdata-ppt.pdf)
- [Award Certificate (PDF)](/uploads/awards/daegu-bigdata-kim-sieun.pdf)
- [Related Article](https://www.idaegu.co.kr/news/articleView.html?idxno=435460)
