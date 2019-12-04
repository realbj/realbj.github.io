## chrome
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
   * The frontend including the main window, UI, and the backend for the application which handles all I/O and storage. This talks to the renderer to manage web pages. ui model, view and controller code for UI features and functionality
* common
  * Files shared between the browser and the renderer that is specific to the Chrome module.
* net
  * Some Chromium-specific stuff on top of the net top-level module. This should be merged with browser/net.
* installer
  * Source files and projects for making the installer (MSI package).
* renderer
  * Chrome specific code that runs in the renderer process.  This adds Chrome features like autofill, translate etc to the content module.
* test:
* automation: Used by tests to drive the browser UI, for example, in test/ui, test/startup, etc. This communicates with browser/automation in the browser.
* page_cycler: Code for running page cycler tests (for performance measurement). See tools/perf/dashboard.
* reliability: Reliability tests for distributed testing of page loads for reliability metrics and crash finding.
* selenium: Code for running the selenium tests, which is a third-party test suite for Ajaxy and JavaScript stuff. See test/third_party/selenium_core.
* startup: Tests for measuring startup performance. See tools/perf/dashboard and tools/test/reference_build.
* ui: UI tests for poking at the browser UI, opening tabs, etc. It uses test/automation for doing most operations.
* unit: The base code for the unit tests. The test code for individual tests is generally alongside the code it is testing in a *_unittest.cc file.
* third_party: Third party libraries that are specific to Chromium. Some other third party libraries are in the top-level third_party library.
* tools
* build: Tools and random stuff related to building.
* buildbot: Buildbot configuration. Buildbot manages our automated build system. See third_pary/buildbot.
* win: Windows build stuff, including some .vsprops files used for project properties and scripts.
* memory: Tools for memory stuff. Currently includes gflags for setting page heap options.
* perf/dashboard: Code for converting performance logs (for example test/startup_test) into data and graphs.
* profiles: Generator for random history data. Used to make test profiles.
