---
layout: post
title: 바닥부터 배우는 강화학습 - ch8. 가치 기반 에이전트
date: 2021-10-10 16:41:00 +09:00
category: RL
use_math: true
---

"바닥부터 배우는 강화학습"([저자 Github](https://github.com/seungeunrho/RLfrombasics)) 책을 읽으면서 정리하고자 합니다.

# 바닥부터 배우는 강화학습

## Chapter 8. 가치 기반 에이전트

- 강화학습에 뉴럴넷 접목시키기

  - 가치 함수를 뉴럴넷으로 표현하기<br>
    $$
    v_{\pi}(s), \ q_{\pi}(s,a)
    $$

  - 정책 함수 자체를 뉴럴넷으로 표현하기<br>
    $$
    \pi (a|s)
    $$

- 에이전트 분류

  - 가치 기반(value-based) 에이전트

    - 가치 함수 q(s,a)에 근거하여 액션 선택
      - SARSA, Q러닝
      - q(s,a)가 곧 정책 함수 역할

  - 정책 기반(policy-based) 에이전트

    - 정책함수를 보고 직접 액션 선택

  - 액터 크리틱(actor-critic)

    - 가치 함수와 정책 함수 모두 사용

    - 액터 : 정책

      크리틱 : 가치함수

### 8.1 밸류 네트워크의 학습

- 밸류 네트워크

  - 뉴럴넷으로 이루어진 가치함수<br> 
    $$
    v_{\theta,\pi}(s)\\
    \theta: 뉴럴넷 파라미터
    $$

- 손실 함수 정의

  - 뉴럴넷을 학습하려면 예측과 정답 사이 차이를 측정하는 손실 함수를 정의해야 함<br>
    $$
    L(\theta) = (v_{true}(s)-v_{\theta}(s))^{2}
    $$

  - 존재하는 모든 상태를 방문해볼 수 없으니 기댓값을 사용하여 정의<br>
    $$
    L(\theta) = E_{\pi}[(v_{true}(s)-v_{\theta}(s))^{2}]
    $$

- 그라디언트 계산<br>
  $$
  L(\theta)의 \ \theta에\ 대한\ 그라디언트\ 계산\\
  \bigtriangleup_{\theta}L(\theta) = -E_{\pi}[(v_{true}(s)-v_{\theta}(s))\bigtriangleup_{\theta}v_{\theta}(s)]
  $$

- 파라미터 업데이트<br>
  $$
  \theta{}' = \theta-\alpha\triangledown_{\theta}L(\theta) \\
  \theta_{}'= \theta+\alpha(v_{true}(s)-v_{\theta}(s))\triangledown_{\theta}L(\theta)
  $$
  

**[몬테카를로 리턴]**<br>
$$
\theta{}' = \theta+\alpha(G_{t}-v_{\theta}(s))\triangledown_{\theta}v_{\theta}(s_{t})
$$

- 샘플 상태 s를 모으고 파라미터를 계속해서 업데이트해 나가면 점점 손실 함수의 값이 줄어들고 결국 뉴럴넷의 아웃풋이 실제 밸류에 수렴한다.

- 에피소드가 끝나야만 리턴을 계산할 수 있으니 실시간으로 업데이트가 불가능하다

  에피소드가 끝날 때가지 많은 확률적 요소가 결합되어 있어 정답지 분산이 크다

**[TD 타깃]**<br>
$$
\theta{}' = \theta+\alpha(r_{t+1}+\gamma v_{\theta}(s_{t+1})-v_{\theta}(s))\triangledown_{\theta}v_{\theta}(s_{t})\\
v_{\theta}(s_{t+1})은\ 반드시\ 상수\ 취급해야\ 한다.
$$

- 반드시 상수 취급해야 하는 이유?
  - 목적지를 변하지 않기 위함
  - 상수 취급하라는 것은 편미분할 때 0으로 만들어 버리는 것
  - 처음 강화 학습 구현할 때 가장 많이 하는 실수
  - TD 타깃 계산시 텐서에서 detach 함수 호출하면 됨

