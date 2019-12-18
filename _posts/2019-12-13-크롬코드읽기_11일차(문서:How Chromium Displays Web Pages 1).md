---
title:  "Chromium 문서 - How Chromium Displays Web Pages (1)"
excerpt: "How Chromium Displays Web Pages (1) "

categories:
  - Chromium
tags:
  - 크로미움
  - Chromium
last_modified_at: 2019-12-18T17:30:00
---

# Chromium는 어떻게  웹페이지를 보여 줄까
* 이 문서는 Chromium에서 웹 페이지를 어떻게 보여주는 가를 아래에서 위로 설명합니다. 
* multi-process 아키텍트 디자인 문서를 먼저 읽어야 합니다. 
* 당시는 주요 component의 블록 다이어그램을 이해하고 싶을 것입니다. 
* multi-process 자원 로딩에서 웹 페이지가 네트워크로 부터 어떻게 패치되고 관심을 갖을 수 있습니다. 

## Conceptual application layers
* 각각의 상자는 어플리케이션 레이어 개념을 나타내며, 상위 레이어 지식이나 의존성이 있는 레이어는 없습니다.

* WebKit
  * 렌더링 엔진은 Safari, Crhomium 그리고 모든 WebKit 기반 브라우저들과 공유 됩니다. 
  * The Port는 WebKit의 일부이며, 플렛폼 종속 시스템 서비스로 통합된 것입니다. (예를 들면 리소스 로딩과 그래픽)
* Glue
  * 우리(크로미움)의 WebKit embedding 계층으로, WeKit 타입을 Chromium 타입으로 전환합니다. 
  * 두 가지 브라우저(Chromium과 test_shell)를  기반으로 합니다.
    * test_shell을 통해 WebKit을 테스트 할 수 있습니다. 
* Renderer / Render host		
  * Chormium의 multi-process embedding layer입니다. 
  * 프로세스간의 알림(notification)과 명령(command)을 대리(proxy)합니다. 
* WebContents
  * 재사용 가능한 component로 컨텐츠 모듈의 메인 클래스입니다. 
  * HTML의 멀티프로세스 렌더링을 전망(view)하기 위해 쉽게 입베디드 됩니다.
  * 상세한 전보듣 [content module pages](https://www.chromium.org/developers/content-module)를 참고하세요	
* Browser
  * 브라우저 윈도우를 표시하며, 다수의 WebContentses를 포함한다. 
* Tab Helpers
  * (WebContentsUserData mixin을 통해) WebContents에 연결될 수 있는 개별적인 객체
  * 브라우저는 WebContentses에 Tab Helpers가 갖고 있는 잡다한 것들을 첨부한다. (즐겨찾기 아이콘, infobars, 등)
	

## WebKit
* 우리는 웹 페이지를 배치(to lay out)하기 위해 WebKit 오픈소스 프로젝트를 사용합니다. 
* 이 코드는 Apple에서 갖고 왔으며, /third_party/WebKit 디렉토리에 저장되어 있습니다. 
* WebKit은 WebCore와 JavaScriptCore로 구성되어 있습니다.
  * WebCore: layout 기능의 코어 부분을 의미합니다. 
  * JavaScriptCore: JavaScript를 실행합니다. 
* JavaScriptCore는 테스트 목적으로만 실행합니다. 기본적으로 우리는  고성능의  V8 JavaScript engine을 사용합니다. 
* Apple가 "WebKit"이라고 부르는 layer는 사용하지 않습니다. 
  * WebCore와 OS X applications 사이에 사용되는 API 포함된 Layer, 예를 들어 Safari.
* 일반적으로 Apple의 코드를 편의상 "WebKit"이라고 합니다.


### The WebKit port
* 가장 낮은 레벨는  WebKit "port"를 갖고 있다. 
* 플렛폼 독립적인 WebCore 코드와 인터페이스로 연결되는 플렛폼용 기능 요구 사항이 구현되어 있다. 
* 파일들은 WebKit 트리(디렉토리)에 위치되었고, 일반적으로 chormium 디렉토리나 Chromium 접미사 파일에 있다. 
* port 대부분은 OS 특화되진 않았다. webCore의 "Chromium port"라고 생각할 수 있다. 
* 폰트 렌더링 같은 일부분은  플렛폼에 따라 다르게 처리되어야 한다. 
	
* 네트워크 트레픽은 렌더 프로세스에서 OS로 직접 연결하지 않고, 멀티프로세스 리소스 로딩 시스템에 의해 관리 된다. 
  * 그래픽은 안드로이드용 Skia 그래픽 라이브러리를 사용하며, 모든 이미지와 글씨(Text)를 제외한 그래픽 작품(graphics primitives)을 관리한다. 
  * Skia는 /third_party/skia에 위치되어 있으며, 그래픽 동작의 주요 진입 지점은 /webkit/port/platform/graphics/GraphicsContextSkia.cpp. 이다. 
  * 같은 경로와 /base/gfx의  여러 파일들을 사용한다. 
		

### The WebKit glue
* 크로미움 어플리케이션은 써드파티 WebKit 코드와 다른 유형(type), 코딩 스타일 그리고 코드 배치를 사용한다. 
* WebKit "glue" 는 구글의 코딩 규칙과 타입을 사용하여 WebKit에 편리한 임베디드 API 를 제공한다.
  * (예를 들면, std:string 대신 WebCore::String , GURL 대신 KURL을 사용한다.)
* glue(글루) 코드는  /webkit/glue 에 위치한다.
* 글루 객체는 보통 WebKit 객체와 비슷한 이름을 사용한다. 하지만, "Web"으로 시작한다. 
  * 예를 들면 WebCore::Frame 이 WebFrame 이 된다.

* WebKit "glue"레이어는 WebCore 데이터 유형에서 나머지 크로미움 코드베이스를 격리하다.
  * 크로미움 코드 기반의 WebCore 변경의 영향을 최소화하기 위해. 
* 예를 들면 크로미움이 WebCore 데이터 유형을 직접 사용하지 않는다.
* API는 WebKit "glue"에 추가되었다. 
  * 일부 WebCore 객체를 사용이 해야 할 때, 크로미움의 benefit을 위해
  
* test shell 어플리케이션은 
  * WebKit 포트와 glue 코드를 테스트 하기 위한 웹 브라우저의 뼈대이다. 
  * Chromium이 WebKit과 통신에 사용하는 glue 인터페이스를 동일하게 사용한다. 
  * 유사한 방법을 사용하면 개발자가 신규 코드를 테스트 할 수 있다.
    * 복잡한 다수의 브라우저 기능, 쓰레드 프로세스가 없어도 된다.
  * 이 어플리케이션은 자동화된 WebKit test를 실행하기 위해 사용된다.
  * 그러나, "test shell"의 단점은 크로미움 처럼 멀티 프로세스 방식으로  WebKit을 사용하지 않습니다.
    * Content 모듈은 "content shell"에 임베디드 되어있습니다. 
      * "content shell은 곧 test를 실행할 것입니다. 
