---
title:  "Chromium 문서 - 소스 디렉토리(Directory Structure) 2"
excerpt: "Chromium 문서읽기 (소스 디렉토리,Directory Structure) 2"

categories:
  - Chromium
tags:
  - 크로미움
  - Chromium
last_modified_at: 2019-12-07T17:30:00
---

## chrome/
* app
  * app은 프로그램의 가장 기본적인 레벨이다.
  * 실행 시작때 동작하며, 현재 프로세스의 기능에 따라 브라우저 혹은 렌더링 코드 중 하나를 호출 할지 결정한다. [dispatch](https://feco.tistory.com/86) 
  * chrome.exe and chrome.dll을 위한 프로젝트가 포함되어 있다.
  * 이미지와 문자열과 같은 리소스를 제외하고는 변경할 필요가 없을 것이다.
  * locales
    * 지역화 DLL을 위한 프로젝트
  * resources
    * 아이콘과 커서
  * theme
    * 윈도우 테마를 위한 이미지
* browser
   * 메인 윈도우와 UI가 포함된 프론트엔드(frontend)와 어플리케이션의 모든 I/O와 저장소를 관리를 위한 백엔드(backend)가 있다.
   * web페이지를 관리하기 위해 렌더러와 소통한다.
     * ui
       * UI 기능을 위한 model, view and controller code
* common
  * 브라우저와 렌더러 사이에 공유되는 파일로, Chrome 모듈에 특화되어 있다.
  * net
    * 크로미움용 최상위 레벨의 Net 모듈. browser/net으로 통합될 수 있다.
* installer
  * 인스톨러를 만들기 위한 소스와 프로젝트 (MSI package).
* renderer
  * 렌더러 프로세스에서 크롬 스펙을 동작시키기 위한 코드들
  * 자동완성, 번역같은 content 모듈 기능이 추가되어 있다.   
* test
  * automation
    * 브라우저 UI에 의해 나뉘어 사용된다. (test/ui, test/startup 등)
    * browser/automation과 통신한다. 
  * page_cycler
    * page cycler 테스트 (성능측정용). (tools/perf/dashboard 참고)
  * reliability
    * "신뢰성 메트릭스"(reliability)와 "크레쉬 찾기(crash finding)"를 로드하는 페이지의 분산 테스트 
  * selenium
    * the selenium tests
    * Ajaxy와 JavaScript용 third-party 테스트 시트  (test/third_party/selenium_core 참고)
  * startup
    * 실행시작(startup) 성능 측정 테스트. (tools/perf/dashboard 와 tools/test/reference_build 참고)
  * ui
    * UI tests for poking at the browser UI, opening tabs, etc. It uses test/automation for doing most operations.
  * unit
    * 유닉 테스트를 위한 코드
    * 개별 기능의 테스트는 *_unittest.cc 파일에서 실행 된다. 
* third_party
  * 크로미움에 특화된 서드파티 라이브러리
  * 몇몇의 라이브러리는 the top-level third_party에 있다.
* tools
  * build
    * 빌드 관련 radom 스터프(stuff)와 툴이 있다.
  * buildbot
    * Buildbot 설정. 
    * Buildbot은 자동 빌드 시스템을 관리 한다. (third_pary/buildbot 참고)
  * win
    * Windows용 빌드 스터프(stuff)로 프로젝트 옵션과 스크립트가 포함된 .vsprops 파일이 있다.
  * memory
    * 메모리 stuff를 위한 툴
    * 현재 page heap 옵션을 설정하기 위한 gflags를 포함한다.
  * perf/dashboard
    * 퍼포먼스 로그를 (데이터,그래픽)로 변환하기 위한 코드 for example test/startup_test
  * profiles
    * 일반적인 random history data. 테스트를 위해 사용된다.
    
## 학습자료
* [Top Quality Design docs](https://www.chromium.org/developers/design-documents)
* important dev docs
  * [multi-process-architecture](https://www.chromium.org/developers/design-documents/multi-process-architecture)
  * [displaying-a-web-page-in-chrome](https://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome)
  * [inter-process-communication](https://www.chromium.org/developers/design-documents/inter-process-communication)
  * [threading](https://www.chromium.org/developers/design-documents/threading)
* code idioms
  * [important-abstractions-and-data-structures](https://www.chromium.org/developers/coding-style/important-abstractions-and-data-structures)
  * [smart-pointer-guidelines](https://www.chromium.org/developers/smart-pointer-guidelines)
  * [chromium-string-usage](https://www.chromium.org/developers/chromium-string-usage)

## 공통 동작(common operations)을 위한 코드 위치(Paths)
* Application startup
  * WinMain는(chrome/app/main.cc)에 있고, chrome 프로젝트와 링크되어 있다. 
  * WinMain은 구글 업데이트 클라이언트를 실행시킨다.
    * 구글 업데이트 클라이언트는 설치/자동업데이트를 담당한다.
    * 업데이터는 하위 경로에서 현재 버전을 찾고, chrome.dll을 로드한다. 
  * 새로 로드된 라이브러리에서 ChromeMain(chrome_main.cc, chrome_dll)를 호출한다. 
  * ChromeMain은 공통 컴퍼넌트를 초기화 한다, 그 후 둘중 하나를 진행한다. 그러면 브라우저가 실행 된다.
    * 실행 옵션(command line flag)에 의해 하위 프로세스 실행이 필요하다면 RendererMain (chrome/renderer/renderer_main.cc) 
    * 새로운 어플리케이션을 로드할 필요하 없다면 BrowserMain (chrome/browser/browser_main.cc)
  * BrowserMain은 브라우저 초기화를 한다.
    * 다양한 실행모드가 있다. (설치된 webapp 실행 모드 / 브라우저 테스트에 연결 모드 등)
  * LaunchWithProfile(browser_init.cc)는 새로운 Browser object(chrome/browser/ui/browser.cc)를 생성한다. 
    * 이 object는 어플리케이션의 최상위 윈도우를 캡슐화 한 것이다. 
    * 첫번째 tab과 동시에 추가된다.    
* Tab startup & initial navigation
  * Browser::AddTab (chrome/browser/ui/broser.cc)는 신규 텝이 추가될 때 호출 된다.
  * 이는 TabContents (browser/tab_contents/tab_contents.cc)를 생성한다.
  * TabContents는  
    * RenderViewHostManager의 초기화 함수(chrome/browser/tab_contents/render_view_host_manager.cc)를 통해
    * RenderViewHost(chrome/browser/renderer_host/render_view_host.cc)를 생성한다. 
  * SiteInstance의 RenderViewHost (Object)는 1개의 렌더러 하위 프로세스를 의미한다.   (???)
    * RenderViewHost는 새로운 렌더러 프로세스러로 실행되거나, 현존하는 RenderProcessHost, RenderProcessHost를 재사용된것.
  * Tab Contents의 NavigationController(chrome/browser/tab_contents/navigation_controller.cc)는 URL 목적지를 URL Bar로 부터 전달 받는다. 
    * NavigationController::LoadURL
* Navigating from the URL bar
  * URL bar에 입력된 URL은 AutocompleteEdit::OpenURL에 전달 된다. 
    * 물론 검색어를 입력한 경우등의 예외는 있다.
  * Navigation controller는 URL로 이동할 것을 지시 받습니다. (NavigationController::LoadURL)
  * NavigationController는 TabContents::Navigate를 NavigationEntry와 같이 호출 한다. 
    * 페이지 전환을 나타내기 위해 만들어 진다.
    * 렌더러 프로세스의 RednerView가 필요하다면 RenderViewHost를 생성한다.
      * 첫번째 페이지 이동이거나 렌더러가 크레시 난 경우, RenderView가 없을 것이다. 
  * NavigationController는 navigation entry를 "pending" 마킹하고 저장한다. 
    * 페이지 전환이 가능한지 확실하지 않기 때문이다.
  * RenderViewHost::NavigateToEntry는 RenderView(renderer process)에게 a ViewMsg_Navigate를 보낸한다.
  * 네비게이트의 성공/실패, 변경이 있을 때, RenderViewHost 는 RenderView로부터 ViewHostMsg_FrameNavigate를 기다린다.
  * WebKit가 committed 하면(서버가 응답하면 그 data를 보낸다), the RenderView는 이 message를 보낸다. 이것은 RenderViewHost::OnMsgNavigate 에서 관리
  * NavigationEntry는 로드가 되었다고 업데이트 된다. 링크 클릭의 경우, 브라우저가 볼적 없는 URL인 경우, 브라우저 초기화된 경우, 최초 실행의 경우, 많은 redirect가 있는 URL인 경우 
  * NavigationController는  경로 목록에 정보를 업데이트 한다. 
  
  
* Navigations and session history
 * 개요
    * 각각의 NavigationEntry는 page ID, block of history state data를 저장한다.
    * page ID는 각 page load를 구분하기 위해 사용한다. 그래서 우리는 어떤 NavigationEntry에 해당하는지 알고 있다.
    * 페이지 로딩이 완료된 정보가 전달 되면, pending NavigationEntry는 -1의 page ID를 갖게 된다.
    * The history state data는 문자열로 연속된다.  (WebCore::HistoryItem) the page URL, subframe URLs, and form data 이 포함되어 있다.
  * 브라우저가 요청을 시작 할 때(URL bar에 주소 입력, back/forward/reload 클릭)
    * WebRequest 는 여러 추가 정보들 사이에서 네비게이션을 대표한다. 새로운 네비게이션은 -1의 ID를 갖는다.
    * “이전 항목”으로 이동할 때는 처음 방문 했을 때의  ID가 할당 됩니다.
    * 이 추가정보는 로드가 “commits”될 때, 쿼리된다. 
    * 메인 WebFrame은 새로운 요청의 로드를 요청한다.
  * 렌더러가 요청을 시작할 때, (유저가 링크를 클릭하거나, 자바스크립트가 location을 변경하는 등)
    * WebCore::FrameLoader는 bajillion varied 로드 메소드를 통해 로드를 요청한다. (??????)
    * 첫 포켓을 서버로 부터 받으면 로드는 완료 된다. (더 이상 “pending”, “provisional”가 없을 때)
  * 첫 방문이라면, webCore는 새로운 historyItem을 만들고 이전 탐색 리스트에  추가할 것이다. 
  * 우리는 새로운 탐색인지 이전 탐색에 있는 세션인지 구분 할 수 있습니다. 
    * RenderView::DidCommitLoadForFrame는 load commit을 관리 합니다.
    * 이전 페이지 상태는 viewHostMsg_UpdateState메시지를 통해  history session 에 저장됩니다. 
    * 브라우저에 네비게이션 상태를 업데이트 할 것을 요청한다. 
    * RenderView의 현재 페이지 id는 committed 페이제 의해 반영된다. 
  * 새로운 네비게이션을 위해 새로운 유니크 아이디가 생성되고, 이전 목록 세션을 위해 첫 번째 방문시 받은 ID로 할당 되며, 네비게이션이 시작될 때 WebRequest에 저장된다. 
 * ViewHostMsg_FrameNavigate 메시지는 브라우저로 보내진다. 네비게이션 목록의 결과를 새로운 url과 기타 정보와 함께 업데이트 한다. 
  
