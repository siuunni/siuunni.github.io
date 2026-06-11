---
title: "가짜연구소 인과추론 팀 10기 — 실무 인과추론 기법 학습·적용"
date: 2025-12-02
weight: 110
featured: true
authors:
  - me

image:
  preview_only: true
tags:
  - Causal Inference
  - Uplift Tree
  - CEVAE
  - 2SLS
  - Meta Learner
---

<div style="display:grid;grid-template-columns:130px 1fr;gap:1rem 1.5rem;margin-bottom:2rem;">

<div style="font-weight:700;">개요</div>
<div>『실무로 배우는 인과추론 with Python』을 함께 공부하며, 인과추론 핵심 기법을 Synthetic 데이터와 실무형 예제로 직접 구현·비교한 스터디 프로젝트입니다.</div>

<div style="font-weight:700;">활동</div>
<div>가짜연구소 10기 인과추론 팀 (오픈소스 협업 스터디)</div>
<div style="font-weight:700;">기간</div>
<div>2025.03~2025.08</div>

<div style="font-weight:700;">담당 역할</div>
<div><strong>대안적 실험설계 파트 발표 · 분석 코드 제작 및 업로드</strong> (기여도 15%)</div>

<div style="font-weight:700;">기술 스택</div>
<div>Python · pandas · NumPy · scikit-learn · CausalML · Git/GitHub</div>

<div style="font-weight:700;">성과</div>
<div><strong>가짜연구소 10기 Project of the Season 수상</strong></div>

</div>

<!--more-->

---

## 무엇을, 왜 했나

상관관계만으로는 "이 처치가 정말 결과를 바꿨는가"를 답할 수 없습니다. 인과추론은 바로 그 질문에 답하기 위한 방법론으로, 마케팅·의료·정책 등 실무에서 의사결정의 근거가 됩니다. 다만 이론서만으로는 실제 데이터에 적용했을 때의 감을 잡기 어렵기 때문에, **이론을 공부하는 데 그치지 않고 직접 데이터에 적용해 실용성을 확인**하는 것을 목표로 삼았습니다.

저는 책의 **대안적 실험설계** 파트를 맡아 발표했고, 주요 기법들의 분석 코드를 직접 작성해 공유했습니다.

---

## 다룬 기법

처치효과를 추정하는 서로 다른 접근들을 하나씩 구현하며, 어떤 상황에 어떤 방법이 맞는지를 비교했습니다.

- **Synthetic 데이터 실습:** 정답(true effect)을 아는 가상 데이터를 만들어, 각 기법이 실제 인과효과를 얼마나 잘 복원하는지 검증했습니다.
- **Uplift Tree:** 처치에 대한 **고객 반응 차이**를 세분화해, 처치가 효과적인 집단과 그렇지 않은 집단을 구분했습니다.
- **CEVAE:** 관측되지 않은 **잠재 교란 변수**를 보정하는 인과 추론 모델을 실습했습니다.
- **도구변수(IV)·2SLS:** 내생성 문제가 있을 때 도구변수와 2단계 최소제곱법으로 인과효과를 추정했습니다.
- **Meta-learner 비교:** S·T·X-learner 등 메타러너 계열의 처치효과 추정 성능을 비교했습니다.
- **DragonNet vs Meta-learner:** 신경망 기반 DragonNet과 메타러너의 추정 성능을 정량적으로 비교 분석했습니다.

---

## 핵심 요약

- **이론 → 적용:** 책으로 배운 인과추론 기법을 Synthetic·실무형 데이터에 직접 적용해 실용성을 확인했습니다.
- **다양한 기법 비교:** Uplift Tree·CEVAE·IV/2SLS·Meta-learner·DragonNet을 한 흐름 안에서 비교해, 상황별 적합한 방법을 정리했습니다.
- **오픈소스 기여:** 대안적 실험설계 파트 발표와 분석 코드 업로드로 팀 결과물에 기여했고, **10기 Project of the Season**을 수상했습니다.

---

- [GitHub — causal-inference-with-causalML](https://github.com/CausalInferenceLab/causal-inference-with-causalML)
