# 크로미움 소스 디렉토리 구조 알아 보기 (작성중)
## 개요
* 크로미움은 브라우저(browser)와 렌더러(renderer, Blink 포함)로 2가지로 나뉘어 진다. 
  * 브라우저는 메인 프로세스로 모든 UI와 입출력(I/O)를 담당한다. 
  * 렌더러는 브라우저가 탭별로 구동 한 하위 프로세스입니다.
* 필요하다면 2가지 문서를 참고하세요 : [구글 멀티프로세스 아키텍트](https://www.chromium.org/developers/design-documents/multi-process-architecture), [어떻게 그로미움은 웹페이지를 보여주는가](https://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome)

## Top-Level 프로젝트
* 디렉토리마다 담당하고 있는 프로젝트는 다음과 같습니다. 
  * android_webview: 
    * 안드로이드 플랫폼에 통합하기 적합한 src/content용 [파사트](https://ko.wikipedia.org/wiki/%ED%8D%BC%EC%82%AC%EB%93%9C_%ED%8C%A8%ED%84%B4)를 제공한다.  
    * 개별 안드로이드 App에 대한 고려는 되어 있지 않는다. 
    * 더 많은 정보는 Android WebView 담당(조직) 에게 있습니다. 
  * apps
    * [그롬 앱(Chrome Apps)](https://developer.chrome.com/apps/about_apps)    
  * base
    * 모든 프로젝트 사이에서 공유하는 공통 코드로 문자열 관리, 그래픽 유틸리티 등이 있다. 
    * 2개 이상의 top-level 프로젝트 사이에 공유 하는 것만 있는다. 
  * breakpad
    * 구글의 crash reporting 프로젝트 (오픈소스)
    * 구글의 별도 저장소에서 운영 한다.
  * build
    * 모든 프로젝트로 부터 빌드 관련된 설정이 공유 된다.
  * cc
    * The Chromium compositor implementation.
  * chrome
    * 그롬 브라우저
    * chrome/test/data
      * 테스트에 필요한 Data 파일들
  * components
    * 컨텐츠 모듈와 관련된 컴포넌트 (컨텐츠 모듈이 최상위 계층으로 사용)
  * content
    * 핵심 코드는 멀티 프로세스 sandboxed 브라우저가 필요하다.
    * 왜 우리가 이 코드를 분리 하였는가는 [link](https://www.chromium.org/developers/content-module)를 참고 하세요
  * device    
    * low 레벨의 공통 HW API에 대한 abstraction
  * net
    * 크롬용 네트워킹 라이브러리 The networking library developed for Chromium. 
    * webkit 저장소에서 test_shell를 실행 할 땐, 크롬과 별개로 사용할 수 있다. chrome/common/net을 참고하세요
  * sandbox    
    * 오염된 시스템에서 해킹 된 렌더러를 방어위한 sandbox 프로젝트
  * skia + third_party/skia
    * 구글의 Skia 그래픽 라이브러리 
  * sql
    * sqlite에 대한 wrapper.
  * testing
    * 유닛 테스트를 위한 GTest 코드가 포함되어 있다. 
  * third_party    
    * 200개 이상의 크고작은 외부 라이브러리
    * 이미지 디코더, 압축 라이브러리, 웹엔진 Blink (WebKit의 라이센스 제한을 상속하기 때문에 여기있다.) 
    * [패키지 추가하기](https://www.chromium.org/developers/adding-3rd-party-libraries)
    * .../blink/renderer      
      * paint 명령(그래픽 효과?)과 기타 상태 변경에 의한 HTML, CSS,script 변화를 책임지는 web Engine
  * tools (설명없음)
  * ui/gfx
    * 크롬의 UI 그래픽을 기초로 하는 공유 그래픽 클래스     
  * ui/views
    * UI 개발, 렌터링, 레이아웃, 이벤트 핸들링을 제공하기 위한 (간단한?) 프레임워크
    * 대부분 브라우저 UI는 여기에 구현되어 있다.
    * the base objects를 포함한다.
    * 브라우저 특화된 object는 chrome/browser/ui/views에 있다.
  * url
    * 구글의 URL 파싱 및 정규화 오픈소스 라이브러리
  * v8
    * V8 Javascript 라이브러리
    * 별도의 구글 저장소에서 관리하다. 
  
 * 여러 역사적인 이유로, 몇몇의 작은 top level 디렉토리가 있습니다. 
 * 다음은 어플리케이션(예를 들면 크롬, 안드로이드 WebView, Ash)을 위한 신규 top level 디렉토리 안내입니다.  
 * 어플리케이션이 중복 실행 될 수 있더라도 코드는 어플리케이션 내 하위 디렉토리에 위치해야 합니다. 
 * 약간 구식의 디텍토리 다이어그램이 있습니다. 부분적으로 WebKit은 blink/renderer로 대체 해야 합니다. 
 * 아래 있는 모듈은 높은 상위 모듈의 코드를 include 할 수 없습니다. (예를 들면 content는 chrome 헤더를 include할 수 없다.) 하지만 임베디드 API을 사용할 수 있다.
 ![구식다이어그램](https://www.chromium.org/_/rsrc/1308680092356/developers/how-tos/getting-around-the-chrome-source-code/Content.png)

## Content/
* browser
  * 어플리케이션을 위한 백엔드(backend)로 모든 I/O와 자식 프로세스와의 통신을 관리한다. 웹페이지의 렌더링을 위한 소통에 필요하다
* common 
  * 여기 파일들은 여러 프로세스 사이에 공유된다. 예를 들면 브라우저와 렌더러, 렌더러와 플러그인 등...
  * 크롬에 특화된 코드이며, base에 적용할 수 없다.
* gpu
  * GPU 프로세스 관련 코드로 3D 합성/ 3D Api에 사용된다.
* plugin
  * 브라우저 플러그인(다른 프로세스)을 실행하기 위한 코드이다.
* ppapi_plugin
  * Pepper plugin 프로세스를 위함 [the Pepper plugin](http://egloos.zum.com/Aimez/v/1868756)
* renderer
  * 각 텝의 하위 프로세스와 관련된 코드
  * I/O를 위해 브라우저와 통신하고, WebKit을 심는다(embed).
* utility
  * 센드박스 프로세스에서 렌덤처리를 위한 코드.  
  * 브라우저에서 신뢰하지 않은 데이터를 처리할 때 사용한다.
* worker
  * HTML5 Web Workers.

