---
title:  "Chromium 문서 - Multi-process Architecture (2)"
excerpt: "Chromium 문서읽기 Multi-process Architecture (2) "

categories:
  - Chromium
tags:
  - 크로미움
  - Chromium
last_modified_at: 2019-12-13T17:30:00
---

원본: https://www.chromium.org/developers/design-documents/multi-process-architecture

## 렌더 프로세스 관리
* 각 렌더 프로세스는 전역 RenderProcess를 갖고 있다. 
	* RenderProcess는 부모 프로세스(브라우저)와 통신을 하고 전역 상태(global state)를 유지한다. 
* 브라우저는 각각의 렌더 프로레스에 대응하는 RenderProcessHost를 유지한다.
	* RenderProcessHost는 브라우저 상태를 관리하고, 렌더러와 통신을 한다.
* 브라우저와 렌더러는 Chromium의 IPC 시스템을 이용하여 통신한다. 


## 관리법
* 각 렌더 프로세스는 한 개 혹은 그 이상의 RenderView를 갖고 있다.
	* RenderView는 텝의 컨텐츠와 일치하며, RenderProcess에 의해 관리 된다.
* RenderProcessHost 통신은 RenderViewHost가 렌더러의 view를 위한 통신을 관리한다. 
* 각각의 view는 view ID가 부여된다.
	* 같은 렌더러에서 다수의 view들의 구별하기 위해 사용된다.
* ID는 렌더러 안에서 유일(unique)하지만 브라우저 안에서는 그렇지 않다. 
	* 그래서 view를 구분하기 위해서는  RenderProcessHost 와 view ID 가 필요하다.
* 브라우저와 특정 텝의 컨텐츠의 통신은 RenderViewHost로 완성된다.
	* RenderViewHost는 RenderProcessHost통해 RenderProcess와 RenderView에게 메시지를 어떻게 전달 하는지 알고 있다.

# Components and interfaces

## Render 프로세스에서
* RenderProcess는 
	* 브라우저에서 RenderProcessHost와의 IPC를 처리한다.
	* 렌더러 프로세스마다 RenderProcess 1개가 있다. 
	* 브라우저와 렌더러간의 통신의 방식이다. 
* RenderView는 
	* (RenderProcess를 통해) 브라우저 프로세스의 RenderViewHost와 WebKit에 포함된 layer와 통신한다. 
	* 텝 혹은 popup 창의 web page의 컨텐츠를 나타낸다.


## 브라우저 프로세스에서
* Browser object는 브라우저 창의 최상위 레벨을 표시한다.
* RenderProcessHost object는 단일 브라우저의 브라우저 측과 렌터러간의 IPC 연결을 대리한다.
* 브라우저 프로세서에는 각 렌더러 프로세스 당 1개의  RenderProcessHost가  있습니다. 
* RenderViewHost는 remote renderView와의 통신을 보호한다. (encapsulates)
	* RenderWidgetHost는 입력과 브라우저의 RenderWidget 그리기를 처리한다. 
* 

보다 상세한 정보는 How Chromium displays web pages (https://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome)를 참고하세요

