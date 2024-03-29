---
layout: post
title: Crontab을 이용하여 주기적으로 파일 삭제하기
date: 2021-10-06 13:39:00 +09:00
category: ubuntu
---
Crontab을 이용한 파일 삭제 예시를 정리합니다.

# LOG 파일 삭제

임의의 Log 파일을 생성하여 매 분마다 삭제하는 작업을 하려고 한다.

1. 임의의 Log 파일을 생성

   ~~~
   $ mkdir Test
   $ cd Test
   $ tee 1.log 2.log 3.log
   ~~~
2. 스크립트 작성

   원하는 위치에 쉘 파일을 작성한다. (나의 경우 홈 디렉토리에 설치 /home/djjin)

   ~~~
   $ vi DeleteLog.sh
   아래 내용 저장

   #! /bin/sh
   /usr/bin/find /home/djjin/Test/*.log -type f -ctime -7 -exec rm {} +

   $ chmod +x DeleteLog.sh
   ~~~
   > -type f : 파일만 찾는 옵션
   >
   > ```
   > 디렉토리 찾으려면 -d 옵션으로 변경
   > ```
   >
   > -mtime : 수정된 날짜 시간 기록 찾는 옵션
   >
   > -ctime : 생성된 날짜 시간 기록 찾는 옵션
   >
   > -atime : 읽히거나 실행 됬을때의 기록 찾는 옵션
   >
   > ```
   > 날짜 기준이 + 7이면 7일 이후의 파일들을 찾으며 -7 이면 7일 이내의 파일을 찾음
   > ```
   >
3. crontab에 해당 로그 등록하기

   - 매분 로그 파일 삭제하기

     ~~~
     $ crontab -e
     * * * * * /home/djjin/DeleteLog.sh
     ~~~
   - 2주일에 한 번 저장된 기간이 2주가 넘은 로그 파일을 자동으로 삭제하기

     ~~~
     $ vi DeleteLog.sh

     아래 내용 저장
     #! /bin/sh
     /usr/bin/find /home/djjin/Test/*.log -type f -ctime +14 -exec rm {} +

     $ chmod +x DeleteLog.sh

     $ crontab -e
     * * 14 * * /home/djjin/DeleteLog.sh 
     ~~~
