---
layout: post
title: 바닥부터 배우는 강화학습 - ch8. 가치 기반 에이전트
date: 2021-10-10 16:41:00 +09:00
category: RL
use_math: true
mathjax: true
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

### 8.2 딥 Q러닝

- q(s,a) 사용

  내재된 정책(implicit policy) : 밸류만 평가하는 함수인데 마치 정책 함수처럼 사용

**[이론적 배경 - Q 러닝]**

- 벨만 최적방정식 이용<br>
  $$
  Q_{*}(s,a) = E_{s_{}'}[r+\gamma\underset{a'}{max}Q_{*}(s',a')-Q(s,a)]\\
  Q(s,a) = Q(s,a) + \alpha(r+\gamma\underset{a'}{max}Q(s',a')-Q(s,a))
  $$

- 손실함수 정의<br>
  $$
  L(\theta) = E[(r+\gamma\underset{a'}{max}Q_{*}(s',a')-Q_{\theta}(s,a))^{2}]
  $$
  <br>손실 함수를 정의할 때 기댓값 연산자 E가 반드시 필요하다.

- 파라미터 업데이트<br>
  $$
  \theta_{}'= \theta+\alpha(r+\gamma\underset{a'}{max}Q_{\theta}(s',a')-Q_{\theta}(s,a))\triangledown_{\theta}Q_{\theta}(s,a)
  $$
  <br>파라미터를 계속해서 업데이트해 나가면 Q(s,a)는 최적 액션가치 함수에 가까워진다.

- 미니 배치

  - 기댓값 연산자를 없애기 위해 여러 개의 샘플을 뽑아 평균을 이용해 업데이트
  - 미니 배치 사이즈
    - 하나의 미니 배치를 구성하는 데 몇 개의 데이터를 사용할 것인가
  - 배치 크기가 커질수록 더 장확한 그라디언트를 계산할 수 있지만 그만큼 한 번에 소모해버리는 데이터가 많아지므로 적절하게 조절해야 한다.

- 딥 Q러닝 pseudo code<br>
  $$
  \begin{align*}
  \text{1.}&\quad Q_{\theta}의\ 파라미터\ \theta\ 초기화 \\
  \text{2.}&\quad 에이전트의\ 상태\ s를\ 초기화(s\leftarrow s_{0})\\
  \text{3.}&\quad 에피소드가\ 끝날\ 때까지\ 다음(A\sim E)을 반복\\
  &\quad \text{A}.\ Q_{\theta}에 대한\ \varepsilon - greedy를\ 이용하여\ 액션\ a'를\ 선택\\
  &\quad \text{B}.\ a를\ 실행하여\ r과\ s'을 관측\\
  &\quad \text{C}.\ s'에서\ Q_{\theta}에\ 대한\ greedy를\ 이용하여\ 액션\ a'를\ 선택\\
  &\quad \text{D}.\ \theta_{}' \leftarrow  \theta+\alpha(r+\gamma\underset{a'}{max}Q_{\theta}(s',a')-Q_{\theta}(s,a))\triangledown_{\theta}Q_{\theta}(s,a)\\
  &\quad \text{E}.\ s\leftarrow s' \\
  \text{4.}&\quad 에피소드가\ 끝나면\ 다시\ 2번으로\ 돌아가서\ \theta가\ 수렴할\ 때까지\ 반복
  \end{align*}
  $$

**[익스피리언스 리플레이와 타깃 네트워크]**

- DQN (Deep Q-Network)

  - Experience Replay
    - 겪었던 경험 재사용
    - replay buffer 개념 도입
      - 버퍼에 가장 최근 데이터 n개를 저장해 놓자
    - 각각의 데이터 사이 상관성이 작아 더 효율적으로 학습 가능
    - 단, off-policy 알고리즘에만 사용가능 (행동 정책과 타깃 정책이 서로 다른 정책)
  - Target Network
    - $r+\gamma\underset{a'}{max}Q_{\theta}(s',a')$ 정답으로 사용되기 때문에 정답이 $\theta$에 의존
    - 정답지가 자주 변하는 것은 학습의 안정성을 매우 떨어뜨림
    - 정답을 계산하기 위한 타깃 네트워크, 학습을 받고 있는 Q 네트워크 2개 준비
    - 타깃 네트워크 고정해놓고 정답지를 계산

- DQN 구현

  <a href='https://github.com/jdj2261/RL-study/blob/master/RLfrombasics/ch8_DQN.py' target='_blank'>Github</a>

