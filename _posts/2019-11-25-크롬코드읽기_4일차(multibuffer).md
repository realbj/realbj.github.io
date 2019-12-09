---
title:  "Chromium Media Code - Multibuffer"
excerpt: "Chromium Media 코드읽기 (Multibuffer)"

categories:
  - webMedia
tags:
  - 크로미움
  - Chromium
  - webMedia
last_modified_at: 2019-12-07T17:30:00
---

# Multibuffer
* File
  * /src/media/blink/multibuffer.cc
  * /src/media/blink/multibuffer.h
  * /src/media/blink/multibuffer_unittest.cc
  * /src/media/blink/lru.h

* 검토 내용
  * 다수의 Reader와 DataProvider(Writer)가 사용할 수 있는 buffer
    * (과제) 관련 내용은 몇 차례 다른 파일을 읽으면서 확인해야 함. 
  * Data 타입은 DataBuffer이며, Data의 제공받고 전달하는 역할에 충실 한다.
  * LRU 알고리즘을 통해 다음에 제공해야 할 Data를 선정한다. 
    * 알고리즘 은 lru.h에 -> multibuffer용은 GlobalLRU로 구현
    * 참고1) https://j2wooooo.tistory.com/121
    * 참고2) https://jins-dev.tistory.com/entry/LRU-Cache-Algorithm-%EC%A0%95%EB%A6%AC
    
