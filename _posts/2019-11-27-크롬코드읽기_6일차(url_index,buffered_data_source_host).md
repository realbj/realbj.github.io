---
title:  "Chromium Media Code - url index, buffered_data_source_host"
excerpt: "Chromium Media 코드읽기 (url index, buffered_data_source_host)"

categories:
  - webMedia
tags:
  - 크로미움
  - Chromium
  - webMedia
last_modified_at: 2019-12-07T17:30:00
---

# url index
* url 관련 정보 + multibuffer 로 구성되어 있다. 
* unittest에는 URL과 직접적으로 관련된 내용만 작성되어 있다. 

# buffered_data_source_host
* AddBufferedByteRange  : 버퍼 Range를 추가 함.
* AddBufferedTimeRanges : byte Ranges를 Time Ranges로 변환하고 추가한다. 
* DidLoadingProgress    : 현재 did_loading_progress_ 값을 리턴하고 did_loading_progress_를 false로 만든다.
* CanPlayThrough : 지금 재생하기 충분한 버퍼를 갖을 경우 True 리턴

