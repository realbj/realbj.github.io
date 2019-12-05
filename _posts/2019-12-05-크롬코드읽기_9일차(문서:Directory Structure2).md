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
    
## 
