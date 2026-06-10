---
title: "배터리 전압·온도 시계열 기반 이상 셀 탐지"
date: 2025-12-12
weight: 10
featured: true
math: true
image:
  preview_only: true
tags:
  - Manufacturing AI
  - Battery
  - Time Series
  - Conformal Prediction
  - FPCA
---

<div style="display:grid;grid-template-columns:130px 1fr;gap:1rem 1.5rem;margin-bottom:2rem;">

<div style="font-weight:700;">개요</div>
<div>배터리팩 충전 과정의 <strong>전압 및 온도</strong> 시계열을 함수형 데이터로 변환하고, 데이터 특성에 맞춰 FPCA와vd-FPCA로 차원을 줄인 뒤, GMM 기반 등각 예측(Conformal Prediction)으로 통계적으로 보장된 이상 탐지 기준을 설계한 프로젝트입니다.</div>

<div style="font-weight:700;">데이터</div>
<div>KAMP 「전기차 배터리 충전 실험 데이터(품질보증)」 — 셀 176개(16 모듈 × 11 셀) 전압, 온도 측정점 32개(모듈당 2개). 학습은 정상만, 테스트는 정상,이상 혼합</div>

<div style="font-weight:700;">기술 스택</div>
<div>Python,R,scikit-fda, scikit-learn,</div>

<div style="font-weight:700;">성과</div>
<div>전압·온도 이상을 <strong>위양성 없이 전수 식별</strong>(실제,시뮬레이션 데이터 모두 F1 ≈ 1), 유의수준 $\alpha=0.01$ 에서 통계적으로 보장된 커버리지 달성</div>

</div>

<!--more-->

---

## 문제 정의

제조 현장의 이상 탐지는 두 가지 제약이 있습니다. **① 이상 데이터를 얻기 어렵고, ② 라벨을 얻기는 더 어렵습니다.** 이 배터리팩 데이터도 같은 상황이었습니다.

- **학습 데이터:** 전부 **정상** 상태의 충전 과정만 수집 — 176개 셀 전압 + 32개 온도
- **테스트 데이터:** 정상·이상이 섞여 있지만, 이상 샘플엔 **이상 발생 시점과 "이상이다"라는 이진 라벨만** 제공. 이상의 **구체적 유형(온도/전압)은 알려주지 않음**
- **판정 규칙:** 전압이든 온도든 한쪽이라도 이상이면 그 배터리팩 전체를 이상으로 분류

즉 정상 데이터만으로 정상의 분포를 학습하고, 새 샘플이 그 분포를 벗어나는지를 판단하는 **단일 클래스(novelty) 탐지** 문제로 접근했습니다. 그리고 "몇 % 신뢰수준에서 이상이다"라고 말할 수 있도록, 통계적 보장이 있는 판정 기준이 필요했습니다.

---

## 배터리팩 계층 구조

배터리팩은 계층적으로 구성됩니다. 팩 1개 = **모듈 16개**, 모듈 1개 = **셀 11개**, 따라서 팩 전체는 **176개 셀**입니다. 온도는 모듈마다 센서 2개가 달려 총 32개 측정점을 갖습니다. 이 **셀–모듈–팩 계층**을 그대로 분석 단위에 반영했습니다(온도는 모듈당 2개 센서를 평균내 모듈 대표 온도로 사용).


<figure style="margin:1.5rem auto;max-width:560px;">
  <img src="/uploads/papers/battery_pack.png"
       alt="전기 자동차용 배터리 팩-모듈-셀[사진 출처: 삼성 SDI]"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    배터리 팩은 여러 개의 모듈로 구성되며, 각 모듈은 다수의 셀로 이루어진 계층적 구조를 가졌습니다.
  </figcaption>
</figure>

---

## 데이터 전처리

함수형 데이터 분석은 모든 곡선이 같은 충전 사이클을 담고 있어야 합니다. 그래서 전처리의 핵심은 실제 충전 구간만 정확히 절단하는 것이었습니다.

**① 기본 절단.** 각 곡선에서 전압 및 온도의 **최솟값 인덱스를 충전 시작점, 최댓값 인덱스를 충전 완료점**으로 잡아 완전한 충전 사이클을 추출했습니다.

**② 변화점 탐지로 정밀 절단.** 그런데 일부 데이터는 충전 초반에 **최저 전압이 한동안 평평하게 유지**되는 구간이 있었습니다. 단순히 최솟값의 첫 인덱스를 시작점으로 잡으면, 충전 전 평형 상태까지 끌려 들어가는 문제가 생겼습니다. 그래서 이런 경우에만 선택적으로 **변화점 탐지(ruptures 라이브러리)** 를 적용해, 감지된 변화점 중 **가장 이른 시점**을 실제 충전 시작점으로 삼았습니다. 모든 데이터에 적용하면 계산 비용이 크기 때문에, **문제가 되는 곡선에만 선택 적용**해 효율적으로 처리했습니다.

**③ 길이 통일.** 곡선마다 측정 길이가 달라, 모두 **1초 단위 격자로 선형보간**하고, 가장 긴 곡선을 기준으로 짧은 곡선은 마지막 값을 연장해 길이를 맞췄습니다. 다만 **온도는 센서별 변화 양상이 너무 달라** 같은 방식으로 늘리면 본래 패턴이 손상되고 탐지 성능이 떨어졌습니다. 그래서 온도에는 **가변 도메인 함수형 주성분 분석(vd-FPCA)** 을 적용했습니다. vd-FPCA는 길이가 다른 곡선을 공통 격자로 억지로 맞추지 않고, **도메인 길이를 변수로 한 삼변량 평활(penalized thin-plate spline)로 공분산을 추정하고 길이에 조건부로 고유분해**하는 방법입니다([Johns et al., 2019](https://doi.org/10.1080/10618600.2019.1604373)). 덕분에 측정 길이가 제각각인 온도 곡선의 고유한 패턴을 보존하면서 차원을 줄일 수 있었습니다.

---

## 함수형 분석

전압과 온도는 데이터 성격이 달랐습니다. 전압은 곡선 길이가 비교적 일정해 일반 FPCA를 적용했고, 온도는 센서마다 측정 길이가 제각각이라 길이 차이를 다룰 수 있는 vd-FPCA를 적용했습니다.

### 전압 — FPCA + Mahalanobis 거리

먼저 데이터를 **팩 단위 5:5로 Train / Calibration 으로 분할**한 뒤, 모든 전압 곡선을 함수형 데이터로 변환했습니다. Train 세트에서 **FPCA**를 학습해 고유함수(eigenfunction) 집합을 추정하고, Calibration,Test 곡선은 같은 함수공간으로 **정사영**해 2차원 FPCA 점수 공간에 표현했습니다.

이어 Train 세트에서 모듈 안 셀들의 전압 벡터로 평균 벡터와 공분산 행렬을 추정했습니다. 팩 $i$, 모듈 $j$, 셀 $k$의 전압 벡터를 $y_{ijk}$ 라 하면 모듈 단위 평균은

$$\bar{y}_{ij} = \frac{1}{K} \sum_{k=1}^{K} y_{ijk},$$

Train 세트 공분산 행렬 $\Sigma$ 는

$$\Sigma = \frac{1}{|D_{\mathrm{tr}}|}\sum_{i \in D_{\mathrm{tr}}} \sum_{j=1}^{M}\frac{1}{K} \sum_{k=1}^{K}(y_{ijk} - \bar{y}_{ij})(y_{ijk} - \bar{y}_{ij})^{\top}.$$

이 평균과 공분산 통해 각 셀이 모듈 분포에서 얼마나 벗어났는지를 **Mahalanobis 거리**로 측정해, 셀 단위 이상 점수를 정의했습니다. 셀–모듈 계층을 반영하므로, 팩이 이상으로 판정됐을 때 **어느 셀이 원인인지까지 해석**할 수 있습니다.

### 온도 — vd-FPCA

온도 곡선엔 vd-FPCA를 적용한 후 GMM을 통해 정상 집단과 이상 집단을 군집화했습니다. 주성분 공간에서 온도 이상 데이터는 정상 군집과 멀리 떨어진 $(-25,\,26)$ 부근에 **고립되어 나타났습니다.** 이런 뚜렷한 분리는 **vd-FPCA가 길이가 다른 온도 곡선의 정상·이상 패턴 차이를 효과적으로 포착**했음을 보여줍니다.

<figure style="margin:1.5rem auto;max-width:560px;">
  <img src="/uploads/papers/battery_temp.png"
       alt="온도 vd-FPCA 주성분 점수 공간"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    온도 vd-FPCA 점수 공간. 정상(train·cal·test_temp_ok)은 한 군집을 이루고, 온도 이상(test_temp_ng)은 $(-25,26)$ 부근에 고립됩니다.
  </figcaption>
</figure>

---

## 등각 예측

차원을 줄인 점수 공간 위에서, **등각 예측(Conformal Prediction)** 으로 이상을 판정했습니다.

**왜 등각 예측인가.** 이 데이터는 이상 샘플이 적고 라벨도 부족합니다. 등각 예측은 점 예측 대신 예측 집합(prediction set)을 내놓는데, 교환가능성(exchangeability)이라는 최소한의 가정만으로 불확실성을 정량화합니다. 핵심은 **모델의 유효성과 무관하게 예측 집합이 참값을 포함할 확률이 설정한 수준 이상으로 유지된다**는 점입니다. 덕분에 분포 가정을 강하게 두기 어렵고 데이터 수가 적은 이번 같은 상황에서도 유효하게 작동하고, "$\alpha$ 수준에서 정상을 이상으로 잘못 볼 확률"을 통제할 수 있습니다.

적합성 점수와 예측 집합 구성 방식이 다른 **세 가지 임계값 설정 방법**을 모두 적용해 강건성을 검증했습니다.

- **정상 테스트 데이터** — 세 방법 모두에서 예측 영역 **내부**에 위치해 정상으로 올바르게 분류
- **이상 데이터** — 세 방법 모두에서 예측 영역을 **명확히 벗어나** 이상으로 정확히 식별. 특히 한 이상 샘플은 정상 군집에서 상당히 멀리 떨어져, 높은 신뢰도로 탐지됨
- **위양성(false positive) 0건** — 세 방법 모두에서 완벽한 분류

<figure style="margin:1.5rem auto;max-width:600px;">
  <img src="/uploads/papers/battery_voltage.png"
       alt="전압 적합성 점수 분포와 1% 임계값"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    전압 적합성 점수 분포. 정상(ok)은 낮은 값에 모이고 이상(test_ng)은 큰 값으로 퍼지며, $\alpha=0.01$ 임계값(26.65)을 기준으로 이상이 분리됩니다.
  </figcaption>
</figure>

→ 전압·온도 양쪽 모두에서 $\alpha=0.01$ 수준의 **통계적으로 보장된 커버리지를 유지하면서**, 실제·시뮬레이션 데이터 모두 **F1 ≈ 1** 의 높은 정확도를 달성했습니다.

---

## 시뮬레이션 검증

실제 이상 데이터가 적어, 평균 이동·최댓값/최솟값 이동·시간가변 이동 등 다양한 이상 시나리오를 시뮬레이션으로 생성해 방법론의 강건성을 검증했습니다. 대부분의 시나리오에서 **Accuracy·F1·MCC가 0.94~0.99 수준**으로 안정적이었습니다.

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/battery_sim.png"
       alt="시뮬레이션 데이터 성능 요약 표"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    시뮬레이션 이상 시나리오별 성능 요약 (Accuracy·Precision·Recall·F1·MCC).
  </figcaption>
</figure>

---

## 핵심 요약

- **라벨 없는 제조 데이터에 맞춘 설계:** 정상만으로 분포를 학습하는 novelty 탐지로 접근하고, 셀–모듈–팩 계층 구조를 분석 단위에 반영했습니다.
- **전압·온도를 특성에 맞게:** 길이가 일정한 전압엔 FPCA + Mahalanobis 거리를, 길이가 제각각인 온도엔 vd-FPCA를 적용해 각 신호의 패턴을 살렸습니다.
- **통계적으로 보장된 판정:** GMM 기반 등각 예측으로 세 가지 임계값 방법 모두에서 **위양성 없이** 이상을 전수 식별(F1 ≈ 1)했고, $\alpha=0.01$ 의 커버리지 보장을 확보했습니다.

---

## 링크

- [Award Certificate (PDF)](/uploads/awards/kdata-kim-sieun.pdf)
- [Related Article](https://n.news.naver.com/article/030/0003382790?sid=105)

## 참고문헌

- Johns, J. T., Crainiceanu, C., Zipunnikov, V., & Gellar, J. (2019). [Variable-Domain Functional Principal Component Analysis](https://doi.org/10.1080/10618600.2019.1604373). *Journal of Computational and Graphical Statistics*, 28(4), 993–1006.
