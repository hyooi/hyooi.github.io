---
layout: post
title:  "Intelij에서 git bash사용하기"
categories:
- git
tags:
- git
---

window에서 bash를 편리하게 사용하고 싶은 경우, 
Intelij의 기본 terminal를 git bash로 설정하면 편리하다.

먼저 git bash를 설치한 후, 다음과 같이 <var>File > Settings > Tools > Terminal</var>에
설치한 git bash 경로를 지정해주면 된다.

![Intelij](/assets/images/intelij-gitbash.PNG)

반드시 **"D:Program Files\Git\bin\sh.exe" -login -i** 와 같이 입력함에 유의한다.
