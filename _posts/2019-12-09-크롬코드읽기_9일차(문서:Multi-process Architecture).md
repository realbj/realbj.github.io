# Multi-process Architecture
## 문제
* 거의 불가능한 이상적인 렌더링 엔진
  * 크래쉬(Crash)나 멈춤(Hangs)이 없는 렌더링 엔진
  * 완벽하게 안전한 렌더링 엔진
* 2006년 즈음 웹브라우저는 단일 사용자에 [협력식 멀티태스킹](https://en.wikipedia.org/wiki/Cooperative_multitasking) (cooperative multitasking) 시스템과 비슷했습니다. 
  * (이 구조는) 오작동하는 어플리케이션이 전체 시스템을 다운 시킬 수 있습니다. 
  * 오작동을 이르키는 웹브라우저도 마찬가지입니다. 
  * 하나의 브라우저 혹은 플러그인의 버그가 전체 브라우저 또는 실행 중인 모든 텝들을 다운시킬수있습니다.
* 운영체제가 브라우저보다 튼튼한 이유는 어플리게이션을 서로 분리된 프로세스로 넣기 때문입니다. 
* 크래쉬가 난 어플리케이션은 다른 시스템에 영향을 주지 않는다.
  * 다른 어플리케이션
  * OS의 무결성
  * 다른 유저의 데이터 접근 제한.
  
 ## 아키텍처 개요
* 브라우저 텝에는 개별 프로세스(separate processes)를 사용한다. 
  * 버그나 렌더링 엔진의 고장(Glitches)으로 부터 전체 어플리케이션을 지키기 위함
* 렌더링 엔진 프로세스는 다른 렌더링 엔진이나, 나머지 시스템으로 접근하는 것을 제한한다. 
  * 이는 웹 브라우징에 유리 한다. (메모리 보호(protection)와 OS 접속제어)

* 메인 프로세스는 UI, Tab 관리, plugin 프로세스를 실행한다. 그리고 "Browser precess" 혹은 "browser"라고 부른다.
* 각 Tab용 프로세스는 "render processes" 혹은 "renderers"라고 부른다.
  * renderers는 HTML해석과 배치를 위해 Blink 오픈소스 레이아웃 엔진을 사용한다. 
  
 ![](https://www.chromium.org/_/rsrc/1220197832277/developers/design-documents/multi-process-architecture/arch.png)
