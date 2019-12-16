# Chromium는 어떻게  웹페이지를 보여 줄까
이 문서는 Chromium에서 웹 페이지를 어떻게 보여주는 가를 아래에서 위로 설명합니다. 
multi-process 아키텍트 디자인 문서를 먼저 읽어야 합니다. 
당시는 주요 component의 블록 다이어그램을 이해하고 싶을 것입니다. 
multi-process 자원 로딩에서 웹 페이지가 네트워크로 부터 어떻게 패치되고 관심을 갖을 수 있습니다. 

각각의 상자는 어플리케이션 레이어 개념을 나타내며, 상위 레이어 지식이나 의존성이 있는 레이어는 없습니다.

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
		
