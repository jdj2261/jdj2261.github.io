---
layout: post
title: 자동시작 스크립트 작성하기
date: 2021-10-03 01:50:00 +0900
category: ubuntu
---

자동스크립트를 만들게 되면 일을 좀 더 효율적으로 할 수 있습니다.

예를 들어 파이썬 또는 쉘 스크립트를 이용하여 만들어 로봇의 PC가 켜졌을 때 초기 셋팅을 자동으로 맞춰줍니다.

아래의 내용은 간단한 예제를 가지고 어떻게 스크립트를 작성하는지 보여줍니다.

# UBUNTU 자동시작 셋팅

바탕화면에 실행 아이콘을 만들고 부팅 시

자동으로 그 파일을 실행하는 방법

예시 : 10초 동안 1초에 한번 씩 Test라는 문자열을 출력하는 프로그램

---

## 1. 스크립트 작성하기

- python script 만들기

  ~~~
  $ mkdir -p examples && cd examples
  $ vi pthon_script.py
  ~~~

  아래의 내용 복사 후 저장

  ~~~python
  #! /usr/bin/python
  import time
  
  i=0
  while(i < 10):
    i += 1
    print("Test")
    time.sleep(1)
  print("Finished ..")
  ~~~

  

  실행되는지 확인

  ~~~
  $ chomd +x pthon_script.py
  $ ./pthon_script.py
  ~~~

- bash script 만들기

  ~~~
  $ vi bash_script
  ~~~

  아래의 내용 복사 후 저장

  >#! /bin/bash -e
  >/opt/examples/python_script

  실행되는지 확인

  ~~~
  $ chmod +x bash_script
  $ ./bash_script
  ~~~

- 폴더 경로를 /opt 디렉토리 안으로 링크걸기

  >opt 디렉토리에 옮기는 이유?
  >
  >응용프로그램(Application)이 /opt 디렉토리 안에 설치됨

  실행되는지 확인

  ~~~
  $ sudo ln -s ~/examples /opt
  $ /opt/examples/bash_script
  ~~~

  

## 2. 바탕화면에 아이콘 만들기

바탕화면에 아이콘을 만드는 방법은 두가지가 있다.

gnome 편집기를 이용하는 방법, 이용하지 않는 방법

### 1) gnome 편집기 이용

1. gnome-panel 설치
   ```
   $ sudo apt-get install gnome-panel
   ```
   
2. gnome-desktop-item-edit 명령어 이용
   ```
   $ gnome-desktop-item-edit ~/Desktop --create-new
   ```
   
3. 이름, 명령어 ,설명 입력 후 OK 버튼 누르기
   
   이름 : 바탕화면에 보여지는 이름
   
   명령어 :  gnome-terminal  -- /opt/examples/bash_script
   
   설명 : 블라블라
   
   ~~~
   명령어에 " gnome-terminal -- " 후 실행 명령어를 적으면 터미널이 띄워진 상태에서 명령어가 실행된다. 
   터미널을 안보이게 하려면 /opt/examples/bash_script 이것만 입력하면 된다.
   ~~~
   
4. 바탕화면에 만들어진 아이콘을 눌러 실행됨을 확인한다.

### 2) 수기 작성

```
$ cd ~/Desktop
$ vi example.desktop
```

아래의 내용 복사 후 저장

~~~
#!/usr/bin/env xdg-open
[Desktop Entry]
Version=1.0
Type=Application
Icon[en_US]=gnome-panel-launcher
Terminal=false
Name[en_US]=TEST
Exec=gnome-terminal -- /opt/examples/bash_script
Name=TEST
Icon=gnome-panel-launcher
~~~

아이콘 버튼 눌러 실행됨을 확인

```
$ chmod +x example.desktop
```

### 3. 자동 시작프로그램 등록

시작 프로그램 검색하여 실행

```
이름 : 바탕화면에 보여지는 이름
명령어 : gnome-terminal -- /opt/examples/bash_script
설명 : 블라블라
```

입력 후 저장

재부팅 후 자동으로 터미널이 띄워지며 실행되는지 확인한다.
