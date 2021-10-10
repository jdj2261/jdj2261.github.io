---
layout: post
title: Kinematics-Coordinate Transformations
date: 2021-10-10 14:28:00 +09:00
category: robotics
use_math: true
---
# Chapter 2. Kinematics

## 2.1. Coordinate Transformations

### 2.1.1 World Coordinates

- 로봇을 개발할 때 가장 첫 번째로 생각해야 할 일은 무엇일까?

  - 로봇의 각 part(link or joint)의 위치를 명확하게 정의하는 것
  - 월드 좌표계를 사용해서 표현하자.

- **World Coordinates를 사용함으로써** 로봇과 주변 객체, 물체를 성공적으로 잡는 것, 충돌 등을 define 할 수 있다.

- World Corrdinates에서 사용되는 Position은 **Absolute Positions** 라고 불린다.

  - 아래의 식을 이용하여 우리는 월드 좌표계 안에서 Absolute Attitude와 Absolute Velocity를 정의할 수 있다.

    <img src="/public/img/2021-10-10-transform0.png" height="80"/>

### 2.1.2. Local Coordinates and Homogeneous Transformations

<img src="/public/img/2021-10-10-transform1.png" height="400"/><br>

<img src="/public/img/2021-10-10-transform2.png" height="40"/>.  <img src="/public/img/2021-10-10-transform3.png" height="60"/>


$$
P_h : the\ hand\ tip\ position\\
P_a : the\ absolute\ position\ of\ the\ left\ shoulder\\
r: the\ position\ of\ the\ hand\  end\ relative\ to\ the\ shoulder
$$

- 로봇 왼팔이 x축을 중심으로 파이만큼 회전을 할 때 월드 좌표계에서 어떻게 표현을 할 것인지 설명하려고 한다.

  - Rotation Matrix의 정의와, local coordinate 상의 position, attitude를 표현하는 방법에 대해서 알아야 한다.

    - a frame의 x축, y축, z축으로 향하는 방향 벡터를 모아 Rotation Matrix로 표현한다.

      <img src="/public/img/2021-10-10-transform4.png" height="40"/>

    - Rotation Matrix R_a는 벡터 r과 r' 사이의 관계를 나타낸다.
      기존 r 벡터를 a프레임의 x축에 대하여 파이만큼 회전된 r' 벡터를 나타내는 것이다.

        <img src="/public/img/2021-10-10-transform5.png" height="60"/>

    - local coordinate에서 a에서 바라본 h의 위치는 아래처럼 나타낸다.

      <img src="/public/img/2021-10-10-transform6.png" height="60"/>

  - 위의 내용을 토대로 월드 좌표계에서 hand의 위치는 아래처럼 표현된다

    <img src="/public/img/2021-10-10-transform7.png" height="60"/>

    - 위의 식을 행렬로 표현하면 아래와 같다.

      <img src="/public/img/2021-10-10-transform8.png" height="60"/>

    - 0과 1을 넣어주는 이유는 position vector P_a와 rotation matrix R_a를 함께 사용하기 위함이며 4x4 크기의 Matrix가 된다.
      이 Matrix를 **Homogeneous transformation matrices**라고 한다.

      <img src="/public/img/2021-10-10-transform9.png" height="60"/>

    - 따라서, local 좌표계에서 표현된 위치를 world 좌표계에서 표현하기 위해 **Homogeneous transformation matrix**를 사용한다.

      <img src="/public/img/2021-10-10-transform10.png" height="60"/>

- **Homogeneous Transformation**을 통해 World Coordinates에서 물체의 position과 attitude를 표현할 수 있다.

### 2.1.3 Local Coordinate Systems Local to Local Coordinate Systems

<img src="/public/img/2021-10-10-transform11.png" height="300"/>

- 하나의 Local Coordinate System을 다른 Local Coordinate System으로 표현할 수도 있다.
  위의 그림은 어깨에 해당하는 좌표계(Frame a)를 가지고  팔꿈치에 해당하는 좌표계(Frame b)로 어떻게 표현할 것인지 보여주는 그림이다.

- Shoulder 위치 a에서 straight로 내리면 elbow의 위치인 b가 되며 a의 좌표계에서 표현된 방향벡터를 구할 수 있다.
  이 때, straight로 내리는 것은 a 프레임의 y축을 평행이동 시킨 것이다.

- 따라서, a에서 바라본 b의 방향 벡터는 아래와 같이 나타낸다.

  <img src="/public/img/2021-10-10-transform12.png" height="60"/>

- 위 방향벡터를 가지고 a에서 바라본 b의 Rotation Matrix를 나타낼 수 있다.

  - 팔꿈치를 회전할 때 각도 theta가 생성되며 아래의 행렬로 표현한다.

    <img src="/public/img/2021-10-10-transform13.png" height="60"/>

- b 좌표계에서 표현한 hand position과 a 좌표계에서 표현한 hand position의 관계는 아래와 같다.

  <img src="/public/img/2021-10-10-transform14.png" height="60"/>

- 마찬가지로 Homogeneous transformation를 이용하여 표현할 수 있다.

  <img src="/public/img/2021-10-10-transform15.png" height="60"/>

  - aP_b는 frame a에서 바라본 b의 position이며 y축에 대해 평행이동 시킨 거리이다.

- 지금까지 말한 것을 종합하면 월드 좌표계에서 hand의 position은 
  아래와 같이 표현할 수 있다.

    <img src="/public/img/2021-10-10-transform16.png" height="60"/>

- **homogeneous transformation의 곱**을 통해 다른 좌표계에서 표현된 position을 world 좌표계로 표현할 수 있게 되었으며 
  T_b는 world 좌표계에서 표현된 homogenous transformation이다.

    <img src="/public/img/2021-10-10-transform17.png" height="60"/>

- 위와 같이 Homogeneous Transformation들의  **Chain Rules을** 통해 다른 좌표계를 하나의 좌표계로 표현할 수 있다는 것이다.

### 2.1.4 Homogeneous Transformations and Chain Rules

- i 번째 frame에서 바라본 i+1 번째의 frame 관계를 나타낸 Homogeneous Transformation을 일반화 시켜보면 아래와 같이 표현되고

  <img src="/public/img/2021-10-10-transform18.png" height="40"/>

- 월드 좌표계에서 나타낸 Homogeneous Transformation은 첫 번째부터 마지막 frame까지 연속된 T의 곱으로 표현할 수 있다.

  <img src="/public/img/2021-10-10-transform19.png" height="60"/>

위의 형태를 **chain rule**이라고 하며, 체인룰을 통해 Robot의 Kinematics를 너무 복잡하지 않게 풀 수가 있다.

다음 시간에는 회전 운동의 특성에 대해 정리하고자 한다.
