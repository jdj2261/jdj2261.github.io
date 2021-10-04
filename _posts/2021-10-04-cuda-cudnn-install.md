---
layout: post
title: NVDIA-DRIVER, CUDA, CuDnn 설치하기
date: 2021-10-04 11:17:00 09:00
category: ubuntu
---

ubuntu 18.04 버전에 Nvidia driver, CUDA, CUDnn 설치 방법을 정리하였습니다.

## 1. NVIDIA-DRIVER Install

### 1-1. 그래픽 드라이버 설치

아래의 홈페이지 들어가서 그래픽 사양에 맞는 NVIDIA 버전을 확인한다.

<a href="https://www.nvidia.co.kr/Download/index.aspx?lang=kr" target="_blank">Driver Download</a>

필자의 경우, 그래픽 사양 확인 후 nvidia-440을 설치하였다.

~~~
$ sudo add-apt-repository ppa:graphics-drivers/ppa
$ sudo apt-get update
$ apt-cache search nvidia | grep nvidia-driver-440
$ sudo apt-get install nvidia-driver-440
~~~

### 1-2. 설치가 되었는지 확인

~~~
$ reboot
$ nvidia-smi
~~~

NVIDIA, CUDA version을 확인한다.

### 1-3. 기존에 설치된 프로그램과 충돌 시 

nvidia 관련 파일을 모두 삭제한다.

~~~
$ sudo apt --purge autoremove nvidia*
~~~

---

## 2. CUDA Install

### 2-1. CUDA 설치

필자의 경우 CUDA 10.0을 필요로 하니 CUDA 10.0을 설치하려고 함.

아래의 주소로 들어가서 runfile 설치 (크기 2.3G)

https://developer.nvidia.com/cuda-10.0-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=runfilelocal

### 2-2. 설치 파일 실행

다운로드 경로로 들어가 run파일 실행

~~~
$ sudo sh cuda_10.0.130_410.48_linux.run
~~~

입력 시 나오는 라이센스 문구는 Ctrl + c를 입력하여 한번에 넘긴다.

Driver 설치 체크만 해제하고 질문이 나오면 엔터를 누른다.

### 2-3. 환경변수 설정

입력하는 CUDA 경로는 설치 시 입력했던 경로와 동일해야 함

~~~
echo -e "\n## CUDA and cuDNN paths"  >> ~/.bashrc
echo 'export PATH=/usr/local/cuda-10.0/bin:${PATH}' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64:${LD_LIBRARY_PATH}' >> ~/.bashrc
source ~/.bashrc
~~~

### 2-4. CUDA가 설치되었는지 확인 

~~~
$ nvcc --version
~~~

설치된 경로 확인

~~~
$ which nvcc
~~~

### 2-5. 완전 삭제

만약 설치를 잘못 했다면 삭제 후 재설치

~~~
$ sudo apt-get --purge -y remove 'nvidia*'
~~~

---

## 3. Cudnn Install

ubuntu 18.04버전, cuda 10.0 설치 했음에 유의

### 3-1. CuDNN 다운로드

아래의 주소로 들어가 버젼에 맞는 cuDNN 다운로드

https://developer.nvidia.com/rdp/cudnn-archive

로그인 후 다운받을 수 있음

### 3-2. 다운받은 파일의 압축 풀기

~~~
$ tar xzvf cudnn-10.0-linux-x64-v7.6.4.38.tgz
~~~

### 3-3. 폴더 복사 및 권한 변경

폴더 위치 확인 후 해당폴더에 압축 해제한 파일을 복사하고 권한 변경

~~~
$ which nvcc
/usr/local/cuda-10.0/bin/nvcc

$ sudo cp cuda/lib64/* /usr/local/cuda-10.0/lib64/
$ sudo cp cuda/include/* /usr/local/cuda-10.0/include/

$ sudo chmod a+r /usr/local/cuda-10.0/lib64/libcudnn*
$ sudo chmod a+r /usr/local/cuda-10.0/include/cudnn.h
~~~

### 4. 설치 완료 확인

다음 명령어를 입력하여 비슷한 결과가 나오면 설치가 완료된 것

~~~
$ cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2  
~~~

~~~shell
#define CUDNN_MAJOR 7
#define CUDNN_MINOR 6
#define CUDNN_PATCHLEVEL 4
--
#define CUDNN_VERSION (CUDNN_MAJOR * 1000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)
#include "driver_types.h"
~~~

### 5. 마지막으로 필요한 패키지 설치

~~~
$ sudo apt-get install libcupti-dev
~~~
