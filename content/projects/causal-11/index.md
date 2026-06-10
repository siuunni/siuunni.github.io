---
title: "가짜연구소 인과추론 팀 11기 — Python DiD 코드집 배포"
date: 2025-12-01
weight: 120
featured: true
image:
  preview_only: true
tags:
  - Causal Inference
  - DiD
  - Event Study
  - DoubleML
---

<div style="display:grid;grid-template-columns:130px 1fr;gap:1rem 1.5rem;margin-bottom:2rem;">

<div style="font-weight:700;">개요</div>
<div>흩어져 있던 다양한 DiD(이중차분) 방법론을 Python 코드로 통합·정리해, 누구나 바로 따라 쓸 수 있는 인과추론 코드집으로 만들어 배포한 프로젝트입니다.</div>

<div style="font-weight:700;">활동</div>
<div>가짜연구소 11기 인과추론 팀 (오픈소스 코드집 제작·배포)</div>

<div style="font-weight:700;">담당 역할</div>
<div><strong>DiD 파트 코드 제작 (기여도 85%) · 팀원 코드 리뷰 (15%)</strong></div>

<div style="font-weight:700;">기술 스택</div>
<div>Python · pandas · NumPy · statsmodels · linearmodels · DoubleML · LightGBM · Git/GitHub</div>

<div style="font-weight:700;">기대 효과</div>
<div>R 중심이던 인과추론 학습을 <strong>Python으로 확장</strong>하고, 여러 DiD 방법을 한곳에 통합해 재현성·학습 효율 향상</div>

</div>

<!--more-->

---

## 무엇을, 왜 했나

DiD(Difference-in-Differences)는 정책·처치의 효과를 추정하는 대표적인 인과추론 방법이지만, 막상 공부하려 하면 **자료가 대부분 R 중심**이라 Python 사용자에게는 진입 장벽이 높았습니다. 게다가 Basic DiD부터 최신 Staggered DiD·머신러닝 DiD까지 방법론이 흩어져 있어, 한눈에 비교하며 배우기 어려웠습니다.

그래서 **다양한 DiD 방법론을 Python으로 일관되게 구현하고, 한 저장소에 통합해 배포**하기로 했습니다. 저는 그중 **DiD 파트 코드 제작을 주도**했습니다.

---

## 작업 내용

- **데이터 준비:** 지역 단위 패널데이터를 전처리하고, 처치 전후 구간을 구분해 모든 방법론이 공통으로 쓸 수 있는 형태로 정리했습니다.
- **DiD 방법론 구현·비교:** 아래 방법들을 같은 데이터 위에서 구현해 결과를 비교했습니다.
  - **Basic DiD** — 가장 기본적인 2×2 이중차분
  - **회귀 기반 DiD** — 회귀식으로 표현한 DiD
  - **TWFE** — 양방향 고정효과 모형
  - **DRDiD** — 이중 강건(Doubly Robust) DiD
  - **Staggered DiD** — 처치 시점이 집단마다 다른 경우를 보정
  - **Event Study** — 처치 전후 시점별 동적 효과 추정
  - **DoubleML** — 머신러닝으로 교란을 통제하는 이중 머신러닝
  - **LightGBM 기반 ML DiD** — 부스팅 모델을 결합한 DiD
- **코드 리뷰·배포:** DiD 파트 코드를 제작하고 팀원 코드를 리뷰한 뒤, Python 인과추론 코드집으로 정리해 GitHub에 공개 배포했습니다.

---

## 핵심 요약

- **Python으로의 확장:** R 위주이던 인과추론 학습 자료를 Python으로 옮겨, Python 사용자의 진입 장벽을 낮췄습니다.
- **방법론 통합:** Basic·TWFE·Staggered·DoubleML·ML DiD까지 8가지 방법을 한 저장소에 모아, 재현성과 학습 효율을 높였습니다.
- **주도적 기여:** DiD 파트 코드 제작(85%)을 주도하고 팀원 코드를 리뷰해, 배포 가능한 오픈소스 코드집으로 완성했습니다.

---


- [GitHub — awesome-causal-inference-python](https://github.com/CausalInferenceLab/awesome-causal-inference-python)
- [Book — awesome-causal-inference-python](https://causalinferencelab.github.io/awesome-causal-inference-python/main/intro.html)
- [Certification](/static/uploads/awards/awsome_cacual_cerfi.pdf)
