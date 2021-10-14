---
layout: post
title: 바닥부터 배우는 강화학습 - ch9. 정책 기반 에이전트
date: 2021-10-14 11:49:00 +09:00
category: RL
use_math: true
mathjax: true
---

"바닥부터 배우는 강화학습"([저자 Github](https://github.com/seungeunrho/RLfrombasics)) 책을 읽으면서 정리하고자 합니다.



# 바닥부터 배우는 강화학습

## Chapter 9. 정책 기반 에이전트

### 9.1 Policy Gradient

- 정책 기반 에이전트가 필요한 이유?
  - 가치 기반 에이전트는 결정론적 정책이지만 정책 기반 에이전트는 확률적 정책으로 좀 더 유연한 정책이다.
  - 액션공간이 연속적인 경우 정책 기반 에이전트는 𝝿(s)가 주어져 있다면 바로 액션을 뽑아줄 수 있다.
  - 환경 자체가 변하는 경우에도 더 유연하게 대처할 수 있다.
- 정책 네트워크를 강화하는 목적
  - MDP 정보를 모르는 모델 프리 상황
  - 커다란 문제여서 테이블에 값을 담을 수 없어서 뉴럴넷이 꼭 필요한 상황
  - 뉴럴넷을 이용하여 필요한 정책을 강화 하는것이 목적이다.

**[목적 함수 정하기]**

- 정책 네트워크<br>
  $$
  \pi_{\theta}(s, a)
  $$
  <br>

  - θ는 정책 네트워크 파라미터

  - 𝝿(s,a)를 계속해서 강화하는 것이 목적이다.

    *𝝿(s,a)의 손실 함수를 어떻게 정의할까?*

  - 손실 함수가 정의되어야 이를 줄이는 방법으로 파라미터를 업데이트할 수 있다.

    - 밸류를 학습할 때는 정답지를 MC와 TD 기법으로 구했지만 정책 함수의 정답을 구하는 방법은 막연하다.

    - 정책 함수의 정답이 곧 최적 정책인데, 최적 정책을 알 수가 없다.

  - 손실 함수를 줄이는 방향이 아니라, 정책을 평가하는 기준을 세워서 그 값을 증가시키도록 하자.

- 평가 함수 J(θ)

  - 𝝿를 인풋으로 받아 점수를 리턴하는 함수

  - 함수의 값을 증가시키는 방향으로 **그라디언트 어센트**하자

  - 보상의 합에 기댓값을 취한 것이 곧 J(θ)<br>
    $$
    J(\theta) = E_{\pi_{\theta}}\left [\sum_{t}r_{t}\right ]=v_{\pi_{\theta}}(s_{0})
    $$
    <br>

  - 즉,  J(θ)는 S0의 밸류

  - 그러나 시작하는 상태가 항상 고정된 것이 아니기 때문에 확률분포 d(s)를 사용한다.<br>
    $$
    J(\theta) = \sum_{s\in S}d(s)*v_{\pi_{\theta}(s)}
    $$
    <br>

- 그라디언트 어센트<br>

  - 목적 함수를 최대화하기 위해 그라디언트 방향으로 파라미터 업데이트하자.<br>
    $$
    \theta' \leftarrow \theta + \alpha*\triangledown_{\theta}J_{\theta}
    $$

  <br>

- 따라서 우리의 목적은 
  $$
  \triangledown_{\theta}J_{\theta}
  $$
  를 구하는 것이다.

**[1-Step MDP]**<br>
$$
\begin{align*}
&\quad J(\theta) = \sum_{s\in S}d(s)*v_{\pi_{\theta}(s)} \\
&\qquad\quad = \sum_{s\in S}d(s)*\sum_{a\in A}\pi_{\theta}(s,a)*R_{s,a}
\end{align*}
$$
<br>

- R을 모르기 때문에 계산할 수가 없다.<br>
  $$
  \begin{align*}
  &\quad \triangledown_{\theta}J(\theta) = \sum_{s\in S}d(s)*\sum_{a\in A}\pi_{\theta}(s,a)\triangledown_{\theta}log\pi_{\theta}(s,a)*R_{s,a} \\
  &\qquad\quad = E_{\pi_{\theta}}[\triangledown_{\theta}log\pi_{\theta}(s,a)*R_{s,a}]
  \end{align*}
  $$

<br>

- 즉,
  $$
  R_{s,a}
  $$
  는 s에서 a를 선택하고 얻는 보상을 관측하면 되고 이 값을 여러 번 모아 평균을 내면, 그 평균이 곧
  $$
  \triangledown_{\theta}J_{\theta}
  $$
  와 같다.

- 기댓값 연산자를 이용한 샘플 기반 방법론의 힘이며<br>
  $$
  \triangledown_{\theta}log\pi_{\theta}(s,a)*R_{s,a}
  $$
  이 식만 계산하면 끝이다.

**[일반적 MDP에서의 Policy Gradient]**

**Policy Gradient Theorem** <br>
$$
\triangledown_{\theta}J(\theta) = E_{\pi_{\theta}}[\triangledown_{\theta}log\pi_{\theta}(s,a)*Q_{\pi_{\theta}}(s,a)]
$$
<br>

- R이 Q로 바뀐 것

  s에서 a를 할 때 받는 보상 대신 s에서 a를 할 떄 얻는 리턴의 기댓값으로 바뀐 것이다.

  한 스텝만 밟고 MDP가 종료되는 것이 아니기 때문에 이후에 받을 보상까지 더해주는 개념

- policy gradient는 J(θ)에 대한 그리다이언트이며 𝝿(s,a)가 경험한 데이터를 기반으로 계산할 수 있게 해주는 방법론이다.

### 9.2 REINFORCE 알고리즘

**[이론적 배경]**<br>
$$
\triangledown_{\theta}J(\theta) = E_{\pi_{\theta}}[\triangledown_{\theta}log\pi_{\theta}(s,a)*G_{t}]
$$

- Gt는 편향되지 않는 샘플, 즉 Gt의 샘플을 여러 개 얻어서 평균을 내면 그 값이 실제 액션밸류인 Q(s,a)에 근사해진다.

- 수도코드

  <img src="/public/img/2021-10-14-rl1.png" height="400"/>

## 9.3 액터-크리틱

액터 크리틱 방법은 정책 네트워크와 밸류 네트워크를 함께 학습하는 방법이며

Q 액터-크리틱, 어드밴티지 액터-크리틱, TD 액터-크리틱 

총 세가지 방법론을 살펴보려 한다.

**[Q 액터-크리틱]**<br>
$$
\triangledown_{\theta}J(\theta) = E_{\pi_{\theta}}[\triangledown_{\theta}log\pi_{\theta}(s,a)*Q_{\pi_{\theta}}(s,a)]
$$
<br>

- Q(s,a)를 사용하면 Q 액터-크리틱
- 𝝿는 실행할때 액션 a를 선택하는 액터 역할
- Qw는 선택된 액션 a의 밸류를 평가하는 크리틱 역할
- 즉, 정책 𝝿와 밸류 Q를 모두 학습하는 방식을 액터-크리틱이라고 한다.

<img src="/public/img/2021-10-14-rl2.png" height="400"/>

**[어드밴티지 액터-크리틱]**

- 샘플을 무한히 많이 모아서 계산하면 분명 올바른 그라디언트 값을 계산해 주겠지만 과연 효율적인가? 하는 부분에서 의문을 제기할 수 있다.

- 상태 s에 있는 것보다 액션 a를 실행함으로써 추가로 얼마나 가치를 더 얻게 되느냐를 생각해보자

- 이 값을 어드밴티지라고 한다.<br>

  $$
  A_{\pi_{\theta}}(s,a) = Q_{\pi_{\theta}}(s,a) - V_{\pi_{\theta}}(s)
  $$
  <br>
  
  <img src="/public/img/2021-10-14-rl3.png" height="300"/>
  
- V를 기저라고한다.

- 어드밴티지 A를 곱해줌으로써 그라디언트 추정치의 변동성이 작아져 학습 성능 향상에 도움이 된다.

- 즉, 분산을 상당히 개선시킬 수 있다.

- 그러나, Value 함수와 Action Value 함수 모두 근사화를 해야하는 단점이 있다.

<img src="/public/img/2021-10-14-rl4.png" height="500"/>

**[TD 액터-크리틱]**

- 가치 함수 V(s)의 TD 에러 δ<br>
  $$
  \delta = r + \gamma V(s') - V(s)
  $$
  <br>

- TD 에러인 δ의 기댓값 = 어드밴티지 A(s, a)

  다른 말로 A(s,a)의 불편 추정량(unbiased estimate)

- policy gradient 수식<br>
  $$
  \triangledown_{\theta}J(\theta) = E_{\pi_{\theta}}[\triangledown_{\theta}log\pi_{\theta}(s,a)*\delta]
  $$



### Summary of Policy Gradient Algorithms

<img src="/public/img/2021-10-14-rl5.png" height="400"/>

