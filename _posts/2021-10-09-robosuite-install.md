---
layout: post
title: Robosuite 설치하기
date: 2021-10-09 23:10:00 +09:00
category: simulator
---
robosuite [ 공식 홈페이지](https://robosuite.ai/docs/overview.html)와 [github](https://github.com/openai/mujoco-py)를 참조하여 진행하였습니다.

# Robosuite installation

**Ubuntu 18.04 환경에서 Robosuit 설치 및 튜토리얼 실행 매뉴얼**

## 1. pip로 설치

```
$ source anaconda/bin/activate
$ conda activate mujoco-py3.7
```

- install robosuite

  ```
  $ pip install robosuite
  ```

- Test your installation with

  ```
  $ python -m robosuite.demos.demo_random_action
  ```

## 2. 소스파일 다운로드 후 설치

- Clone the robosuite repository

  ```
  $ cd ~/.mujoco
  $ git clone https://github.com/StanfordVL/robosuite.git
  $ cd robosuite
  ```

- Install the base requirements with

  ```
  $ pip install -r requirements.txt
  $ pip3 install -r requirements-extra.txt
  ```

## 3. Test

- Test your installation with

  ```
  $ cd .mujoco/robosuite/robosuite/demos
  $ python demo_random_action.py
  $ python demo_control.py
  $ python demo_device_control.py
  ```

- Demo result

  <img src="/public/img/2021-10-09-img.png/">
