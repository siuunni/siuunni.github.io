---
title: "텍스트 기반 이미지 색채화 품질 개선"
date: 2025-12-08
weight: 50
featured: true
math: true
authors:
  - me
  - 김세희
image:
  preview_only: true
tags:
  - Generative AI
  - Diffusion
  - Computer Vision
  - SAM
  - OpenCLIP
---

<div style="display:grid;grid-template-columns:130px 1fr;gap:1rem 1.5rem;margin-bottom:2rem;">

<div style="font-weight:700;">개요</div>
<div>텍스트 설명을 따라 흑백 이미지를 색채화하는 생성형 AI 프로젝트입니다. 여러 공개 색채화 모델을 비교하고, 텍스트 의미를 반영하는 추론 파이프라인을 구축했습니다.</div>

<div style="font-weight:700;">대회</div>
<div>2025 INHA AI Challenge, 대학원생 부문 3위</div>

<div style="font-weight:700;">과제</div>
<div>텍스트(캡션) 조건부 흑백→컬러 이미지 생성, 텍스트 의미 일치도 기반 Score 평가</div>

<div style="font-weight:700;">기술 스택</div>
<div>Python, PyTorch, Diffusers, SAM(Segment Anything), OpenCLIP, NumPy</div>

<div style="font-weight:700;">성과</div>
<div>최종 <strong>Score 0.70</strong> 달성 (베이스라인 대비 약 17~32% 향상)</div>

</div>

<!--more-->

---

## 문제 정의

텍스트 설명을 입력으로 받아 흑백 이미지를 색채화하되, 결과 색상이 텍스트 의미와 일치해야 하는 과제입니다. "빨간 셔츠를 입은 사람"이라는 설명이 있으면 실제로 그 셔츠가 빨갛게 칠해져야 점수를 받습니다.

핵심은 어떤 모델을 쓰느냐보다, 어떤 모델과 추론 전략의 조합이 텍스트 의미를 더 잘 반영하느냐를 실험으로 찾는 것이었습니다.

---

## 1. 공개 모델 비교

먼저 색채화 분야의 주요 사전학습 모델들을 같은 데이터와 환경에서 비교했습니다.

| 모델 | Score |
|:---|:---:|
| Unicolor | 0.56 |
| ControlNet | 0.53 |
| ControlColor | 0.57 |
| COCO-LC | 0.60 |
| **L-CAD (튜닝 후 최종)** | **0.70** |

같은 조건에서 L-CAD가 가장 좋았고, 이를 메인 모델로 삼아 최적화에 집중했습니다.

---

## 2. 추론 파라미터 튜닝

초기 실험에서 두 가지 문제가 있었습니다. 같은 입력인데도 랜덤 시드에 따라 색이 달라졌고, 텍스트 의미와 맞지 않는 색이 나오기도 했습니다.

디퓨전 모델의 결과는 모델 구조만큼이나 추론 설정에 좌우됩니다. 그래서 주요 추론 파라미터를 조정하며 결과의 안정성과 의미 일관성을 확인했습니다.

- **DDIM Steps** — 샘플링 단계 수에 따른 품질, 안정성 변화
- **Guidance Scale** — 텍스트 조건을 얼마나 강하게 반영할지
- **Strength** — 원본 구조를 얼마나 보존할지
- **Self-Attention Guidance(SAG)** — 생성 결과의 일관성 보정

여러 차례 실험을 거쳐, 성능을 떨어뜨리지 않으면서 결과 변동성을 줄이는 설정을 찾았습니다.

---

## 3. 객체 단위 앙상블

단일 모델의 한계를 확인하기 위해, 객체별로 더 나은 모델 결과를 골라 합치는 앙상블을 설계했습니다.

1. **SAM(Segment Anything)** 으로 이미지 안의 객체를 분리
2. 각 객체에 대해 COCO-LC와 L-CAD 결과를 모두 생성
3. **OpenCLIP** 이미지–텍스트 유사도로 객체마다 텍스트 의미에 더 맞는 결과를 선택해 합성

이 방식은 COCO-LC 단일 모델(0.60) 대비 0.686까지 점수를 올렸지만, 최적화한 L-CAD 단일 모델(0.70)에는 못 미쳤습니다. 객체 경계를 합성하는 과정에서 생기는 부자연스러움이 앙상블의 이점을 상쇄한 것으로 보입니다.

결국 앙상블보다 L-CAD 단일 모델의 추론 최적화가 더 효과적이어서, 이를 최종 제출안으로 택했습니다.

<figure style="margin:1.5rem 0;">
  <div style="display:grid;grid-template-columns:repeat(3,1fr);gap:.75rem;align-items:start;">
    <img src="/uploads/papers/colorization_input.png" alt="흑백 입력 이미지"
         style="display:block;width:100%;height:auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;box-sizing:border-box;">
    <img src="/uploads/papers/colorization_lcad_2.png" alt="L-CAD 색채화 결과 1"
         style="display:block;width:100%;height:auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;box-sizing:border-box;">
    <img src="/uploads/papers/colorization_lcad_1.png" alt="L-CAD 색채화 결과 2"
         style="display:block;width:100%;height:auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;box-sizing:border-box;">
  </div>
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    최종 L-CAD 색채화 결과. 좌측 흑백 입력을 텍스트 의미에 맞게 자연스럽게 색채화한 모습입니다.
  </figcaption>
</figure>

---

## 결과

| 단계 | Score | 비고 |
|:---|:---:|:---|
| 최고 베이스라인 (COCO-LC) | 0.60 | 공개 모델 비교 1위 |
| SAM + OpenCLIP 객체 앙상블 | 0.686 | 단일 모델보다 낮음 |
| **L-CAD 단일 최적화 (최종)** | **0.70** | 최종 채택 |

- 최저 베이스라인(ControlNet 0.53) 대비 약 32%, COCO-LC(0.60) 대비 약 17% 향상
- 2025 INHA AI Challenge 대학원생 부문 3위

---

## 핵심 요약

- **모델 비교로 메인 선정:** Unicolor, ControlNet, ControlColor, COCO-LC, L-CAD를 같은 조건에서 비교해 L-CAD를 메인 모델로 정했습니다.
- **추론 최적화:** DDIM, Guidance, SAG 등 추론 파라미터를 조정해 의미 일관성을 유지하면서 변동성을 줄였습니다.
- **앙상블 검증:** SAM과 OpenCLIP으로 객체 단위 앙상블을 설계해 비교한 결과, 이 과제에서는 단일 모델 최적화가 더 나았고 최종 Score 0.70을 달성했습니다.

---

- [Award Certificate (PDF)](/uploads/awards/inha-ai-challenge-kim-sieun.pdf)
- [Related Article](https://aix.inha.ac.kr/?page_id=3841&vid=34)
