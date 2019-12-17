---
title:  "Chromium 문서 - Multi-process Architecture (2)"
excerpt: "Chromium 문서읽기 Multi-process Architecture (2) "

categories:
  - Chromium
tags:
  - 크로미움
  - Chromium
last_modified_at: 2019-12-18T17:30:00
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


# Sharing the render process
* 일반적으로 새로운 윈도우와 텝은 새로운 프로레스로 열린다.
* 브라우저는 새로운 프로세스를 낳고, RenderView를 만들라고 지시한다.

* 가끔 렌더 프로세스를 공유하는 것이 필요하거나 바람직한 경우가 있다. 
* 웹 어플리케이션이 새로운 윈도우를 열면 동적으로 통신하는 것을 예상한다.
* 예를 들면, 자바 스크립트로 “window.open”를 실행한 경우이다. 
* 이 경우, 우리가 새로운 윈도우나 텝을 생성하면서, 윈도우에 열려 있던 프로세스를 재사용이 필요하다. 
* 프로세스 갯수가 너무 많거나, 이미 해당 domain에 진입한 프로세스가 있을 다면, 새로운 텝을 남아 있는 프로세스에 할당하는 전략이 있다. 
* 이러한 프로세스 전략은 [Process Models](https://www.chromium.org/developers/design-documents/process-models)에 설명해 두었다.

## 망가진 렌더러 검출하기
* 브라우저 프로세스에 연결된 각 ipc 연결은 process handles를 지켜보고 있다. (watches)
* 만약 핸들이 시그널을 받으면, 렌더러 프로세스는 crash 된 것이고, 이를 각 텝에 전달한다. 그러면 “sad tab”을 보여주며, 유저에게 렌더러가 crash 되었다는 것을 알려준다. 
* 페이지는 새로고침을 버튼이나 새로운 이동을 통해 재실행 될 수 있다. 그 땐 프로세스가 없다는 것을 알리고 새로운 것을 만든다.

## Sandboxing the renderer
* 렌더러는 각자의 프로세스에서 동작하며, sandboxing을 통해 프로세스가 시스템 자원에 접근 하는 것을 제한한다. 
* 예를 들면, 렌더러가 네트워크에 접근 하려면, 부모 브라우저 프로세스를 통해서만 가능하다. 파일 시스템 접근도 운영체제의 내부 권한을 통해 제한한다. 

* 파일 시스템과 네트워크 접근을 제한하기 뿐만 아니라, 유저의 display와 관련된 object에 접근 하는 것도 제한할 수 있다. 
* 각각 동작하는 렌더 프로세스는 분리된 윈도우즈 “Desktop”에서 동작하며 유저에는 보이지 않는다. 이 것은 새로운 창이 열리거나, 키 입력을 복제하는 것을 보호하기 위함이다.

## Giving back memory
* 분리된 프로세스에서 동작하는 렌더러는 낮은 우선순위 처럼 숨기기 적합하다.
* 보통, 윈도우즈의 최소화된 프로세스는 자기 메모리를 자동적으로 “가용 메모리” pool에 넣는다. 
* 윈도우즈는 적은 메모리 상황에서 유저가 보고 있는 프로그램은 응답할 수 있도록, 높은 우선순위의 메모리 스왑이 발생하기 전에  낮은 우선순위의 메모리를 디스크로 스왑 한다.   
* 우리는 텝을 숨기는 방식으로 비슷한 원리를 적용할 수 있다. 
* 렌더러 프로세스가 top-level 텝을 갖지 않을 때, 프로세스의 “working set” 크기를 release할 수 있다. 이는 시스템이 필요에 의해 스왑할 메모리를 선택할 때 힌트가 될 수 있도록 한다.
* 왜냐하면, working set 크기를 줄이는 것은 유저가 텝을 바꿀 때,  텝 변경 성능이 줄어드는 것을 확인 하였다. 일반적으로 이 메모리를 release 한다. 

* 유저가 최근에 사용하는 텝으로 이동하면, 텝의 메모리는 복구된며, 최근에 사용된 메모리가 오래된 텝보다 페이징 될 가능성이 높다.
* 충분한 메모리를 갖은 유저는 이런 절차에 대해 알지 않는다. 

* 이런 도움(?)은 낮은 메모리 상황에서는 최적의 메모리 발자국(footprint)을 알려준다. 가끔 사용되는 background tab과 관련된 메모리는 foreground tab의 데이터가 모두 메로리가 로드 되는 동안 스왑 될 수 있다. 
* 단일 프로세스 브라우저는 메모리에 무작위로 분포한 (모든 텝의) 데이터를 갖는다. 그래서 사용/미사용 데이터를 깔끔하게 구분할 수 없어, 메모리랑 성능을 버릴 수 밖에 없다. 

## plug-ins And Extensions
* Firefox 스타일의 NPAPI 플러그인은 렌더러로 부터 분리된 자기 프로세서 동작 한다.  상세 정보 [Plugin Architecture](https://www.chromium.org/developers/design-documents/plugin-architecture)

* [Site Isolation](https://www.chromium.org/developers/design-documents/site-isolation) 프로젝트는 렌더러끼리 더욱 분리되는 것을 목표로 한다. 
