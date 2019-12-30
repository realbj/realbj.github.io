---
title:  "Chromium 문서 - How Chromium Displays Web Pages (2)"
excerpt: "How Chromium Displays Web Pages (2) "

categories:
  - Chromium
tags:
  - 크로미움
  - Chromium
last_modified_at: 2019-12-30T17:30:00
---
# The render process

![](https://www.chromium.org/_/rsrc/1342584100941/developers/design-documents/displaying-a-web-page-in-chrome/Renderingintherenderer-v2.png)

* 크로미움의 렌더 프로세스는 glue 인터페이스를 사용하여 WebKit Port를 포함한다.
* 많은 코드를 포함한 것은 아니다.  
  * 주요 작업은 렌더러 측의 IPC 채널에 있다. (IPC : 렌더러 -> 브라우저)

#### RenderView
* 렌더러에서 가장 중요한 class는 RenderView이며, /content/renderer/render_view_impl.cc 에 있다.
* 이 객체는 web page를 표현한다.
* 브라우저 프로세스에서 오거나 보내는 네비게이션 관련 명령을 처리한다. 
* RenderWidget에서 파생됩니다. (RenderWidget는 그리기와 입력 이벤트의 처리를 제공합니다.)
* RenderView는 전역(글로벌) RenderProcess Object를 통해 브라우저 프로세스와 통신합니다.

### FAQ: RenderWidget과 RenderView의 차이
#### RenderWidget
* RenderWidget은 하나의 WebCore:Widget 객체에 매핑됩니다. 
  * glue레이어에 WebWidgetDelegate라는 추상 인터페이스를 구현을 통해  
* 화면에 있는 윈도우에서 입력 이벤트를 수신하고 그곳에 페인트 한다.  
#### RenderView 
* RenderView는 RenderWidget에서 상속하고, 텝 혹은 팝업 창을 포함한다. 
* 페인팅과 위쳇의 입력 이벤트외에도 네비게이션용 명령도 처리합니다. 
* RenderWidget이 RenderView 없이 존재하는 경우는 한 가지 케이스
  * 웹 페이지에 select box를 위한 것입니다.   
  * 아래쪽 화살표와 같이 있는 박스는 옵션 리스트를 팝업 한다.   
  * select box는 네이티브 윈도우를 통해 렌더러 되어야 하며, 다른 모든 것 위에 나타낼 수 있고, 필요한 경우 프레임에서 팝업 됩니다. 
  * 윈도우는 페인트와 입력을 수신이 필요하지만, 별도의 그것을 위한 "웹 페이지"(RenderView)는 없다.


## Threads in the renderer
* 각 렌더러는 2개의 스레드를 갖는다. (멀티 프로세스 아키텍처 페이지의 다이어그램 혹은 [threading in Chromium](https://chromium.googlesource.com/chromium/src/+/master/docs/threading_and_tasks.md)을 참고하세요 
* render thread에서는 RenderView와 WebKit 코드의 주요 객체가 실행 된다.  
* 브라우저와 통신 할 때, 메시지는 처음에 main thread에 보내고, 브라우저 프로세스에 전달 된다. 
* 무엇보다도, 렌더러에서 브라우저로 동기적으로 메시지보내는 것을 허용한다. 
* 브라우저에서의 결과가 있는 소규모 작업은 연속성을 위해 필요하다. (??,This happens for a small set of operations where a result from the browser is required to continue.)
* 예를 들면 JavaScript로 부터 요구가 있을 때, 페이지용 쿠키를 얻는 것이다. 
* renderer 쓰레드는 차단 할 것이고, 메인 쓰레드는 정확한 응답을 찾을 때까지 모든 메시지를 쌓을 것이다.     
* 그 동안 수신된 모든 메시지는 정상적인 처리를 위해 렌더러 쓰레드에 보내진다. 

# The browser process
![](https://www.chromium.org/_/rsrc/1220197831473/developers/design-documents/displaying-a-web-page-in-chrome/rendering%20browser.png)

## Low-level browser process objects
* 모든 render 프로세스와의 IPC 는 브라우저의 I/O 스레드에서 수행된다. 
* 이 스레드는 모든 네트워크 통신을 처리하여 사용자 인터페이스를 방해되지 않도록 합니다.  

* (유저 인터페이스가 동작하는 곳에서)메인 스레드에서 RenderProcessHost가 초기화 될 때, renderer 프로세스와 렌더러로 가는 파이프 이름이 명명된 ChannelProxy IPC 객체가 생성된다. 
* 이 객체는 브라우저의 I/O 스레드에서 동작하며, 렌더러가는 이름을 수신, 모든 메시지를 UI 스레드에 있는 RenderProcessHost로 자동 전달 한다. 
* 특정 메시지를 걸러낼 수 있는 채널에 ResourceMessageFilter가 설치 되며, 네트워크 요청과 같은 I/O 스레드에서 직접 처리 할 수 있다. 
* 필터링은 ResourceMessageFilter::OnMessageReceived 에서 발생한다.

* UI 스레드에 있는 RenderProcessHost는 모든 특정 뷰 메시지를 적합한 RenderViewHost로 발송합니다. 
  * 제한된 수의 뷰와 관련 없는 메시지를 처리한다.
* 디스페치는 RenderProcessHost::OnMessageReceived에서 된다. 

## High-level browser process objects
* 뷰 관련 메시지는 RenderViewHost::OnMessageReceived에 들어 옵니다. 
* 대부분의 메시지를 여기서 처리되며, 나머지는 RenderWidgetHost 기본 클래스로 전달 된다. 
* These two objects map to the RenderView and the RenderWidget in the renderer (see "The Render Process" above for what these mean). 
* 렌더러에서 2가지 객체 RenderView와 RenderWidget가 매핑된다. 
* Each platform has a view class (RenderWidgetHostView[Aura|Gtk|Mac|Win]) to implement integration into the native view system.
* 각 플렛폼은 기본 view 시스템으로 통합을 구현하는 view 클래스(RenderWidgetHostView[Aura|Gtk|Mac|Win])가 있다.  
* 위 RenderView/Widget은 WenContents 객체이다. 대부분의 메시지는 실제로 해당 객체에 대한 함수 호출로 끝난다. 
* WebContents는 웹페이지의 컨텐츠를 나타낸다. 
* content 모듈의 top-level 객체이며, 웹 페이지를 직사각형 보기로 표시해야 한다. 
* 더 상세항 정보는 [content module pages](https://www.chromium.org/developers/content-module)
* WebContents 객체는 TabContentsWarapper에 포함되어 있다. 
* chrome/에 위치되어 있고, tab을 담당한다. 

## Illustrative examples (예제)
* 추가적인 네비게이션과 시작은 예시는 [Getting Around the Chromium Source Code](https://www.chromium.org/developers/how-tos/getting-around-the-chrome-source-code)에 있습니다 .

### Life of a "set cursor" message
Cursor를 설정하는 것은 렌더러에서 브라우저로 전달 되는 일반적인 메시지의 예이다. 렌더러에서 발생한다.

* "set Cursor" 메시지는 WebKit 내부에서 생성되며, 일반적으로 입력 이벤트에 응답하다. 
* "set cursor" 메시지는 RenderWidget::SetCursor in content/renderer/render_widget.cc에서 시작한다. 

* 메시지를 보내기 위해 RenderWidget::Send를 호출 할 것이다. 
* 이 method는 RenderView가 브라우저에 메시지를 보낼 때에도 사용된다. 
* RenderThread::Send가 호출 될 것이다. 

* IPC::SyncChannel를 호출 하면, 내부적으로 메시지를 렌더러의 메인 스레드에 전달 하며, 명시된 이름에 따라 브라우저에 보내진다. 

브라우저의 제어: 

* RenderProcessHost의 IPC::ChannelProxy는 브라우저의 I/O 쓰레드에서 모든 메시지를 받는다.  
* 먼저, 네트워크 요청과 I/O스레으에 직접 연관된 메시지는 ResourceMessageFilter통해 보내진다. 
* 우리 메시지로부터 필터링 되지 않고, 브라우저의 UI 쓰레드에서 계속 된다.  (IPC::ChannelProxy 내부에서 처리 한다. )

* RenderProcessHost::OnMessageReceived (content/browser/renderer_host/render_process_host_impl.cc)는 렌더 프로세스에서의 모든 view메시지를 수신한다. 
* 몇몇의 메시지 종류는 직접 처리하며, 나머지는 발송 측 RenderView에 해당하는 적절한 RenderViewHost에 전달된다. 

* 메시지는 RenderViewHost::OnMessageReceived(content/browser/renderer_host/render_view_host_impl.cc) 에 도착한다. 
* 많은 메시지가 이곳에서 처리 된다. 하지만, RenderWidget에서 보내지고, RenderVidgetHost에 의해 처리되는 메시지이기 때문임은 아니다. 

* RenderViewHost의 모든 미처리 메시지는 자동적으로 RenderWidgetHost에 보내진다. ("set cursor" 메시지 포함)

* 메시지 맵(content/browser/renderer_host/render_view_host_impl.cc)은 최종적으로 메시지를 RenderWidgetHost::OnMsgSetCursor에 수신하며, 적절한 UI 함수를 호출하여 마우스 커서를 set 한다. 

### Life of a "mouse click" message
마우스 클릭으로 보는 브라우저에서 렌더러로 전달 되는 메시지 예시

* 브라우저의 UI 쓰레드에서 RenderWidgetHostViewWin::OnMouseEvent와 ForwardMouseEventToRenderer가 호출로 윈도우 메시지가 수신된다. 
* forwarder 함수는 입력 이벤트를 cross-platform WebMouseEvent로 패키지하고 연결된 RenderWidgetHost로 전송합니다.

* RenderWidgetHost::ForwardInputEvent는 IPC message ViewMsg_HandleInputEvent를 생성하며, WebInputEvent를 직렬화 화며, RenderWidgetHost::Send를 호출 한다. 
* RenderProcessHost::Send로 전달 되면, 이 함수는 IPC:ChannelProxy에 메시지를 보낸다. 
* 내부적으로 IPC::ChannelProxy는 브라우저의 IO쓰레드에 메시지는 전달하고, 올바른 renderer가 무엇인지 명명한다. 

다른 메시지들은 WebContents에서 생성되며 (특히 탐색 메시지) WebContents에서 RenderViewHost로 유사한 경로를 따른다. 

렌더러 제어: 

* 렌더러의 메인 쓰레드의 IPC::Channel는 브라우저가 보낸 메시지를 읽으며, IPC::ChannelProxy가 렌더러 쓰레드에 전달한다. 
* RenderView::OnMessageReceived는 메시지를 받으며, 많은 종류의 메시지는 직접 처리한다. 
* 클릭 메시지가 아닌 미처리 메시지들과 함께 RenderWidget::OnMessageReceived에 보내면 이는 RenderWidget::OnHandleInputEvent로 전달 된다. 
* 입력 이벤트는 WebWidgetImpl::HandleInputEvent에 제공 되며, WebKit PlatformMouseEvent 클래스로 변환된 후, WebCore::Widget class inside WebKit에 제공 된다. 
