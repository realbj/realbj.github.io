---
title:  "Chromium 문서 - Audio / Video Playback"
excerpt: "Audio / Video Playback"

categories:
  - Chromium
tags:
  - 크로미움
  - Chromium
  - webMedia
last_modified_at: 2020-01-01T17:30:00
---
# Audio / Video Playback
* Document
  * [HTMLAudioElement](https://html.spec.whatwg.org/#htmlaudioelement)
  * [HTMLMediaElement](https://html.spec.whatwg.org/#htmlmediaelement)
  * [HTMLVideoElement](https://html.spec.whatwg.org/#htmlvideoelement)
  
* 아래 문서 대부분이 오래되어 상당한 차이가 있습니다. 
* 최신 문서는 [README.md](https://chromium.googlesource.com/chromium/src/+/master/media/README.md) 를 참고하세요 (media/ directory) (Chromium의 미디어 파이프라인에 대한 코드가 있는 위치입니다.).

## Overview
Chromium의 미디어 재생 구현에 주요한 컴포넌트 중 가장 관심이 많은 다음 3가지가 있습니다. 

- [Pipeline](https://cs.chromium.org/chromium/src/media/base/pipeline.h)  
  - Chromium에 구현된 미디어 재생 엔진  
  - 오디오/비디오 동기화와 리소스 관리를 처리한다.
- FFmpeg{[Demuxer](https://cs.chromium.org/chromium/src/media/filters/ffmpeg_demuxer.h), [AudioDecoder](https://cs.chromium.org/chromium/src/media/filters/ffmpeg_audio_decoder.h), [VideoDecoder](https://cs.chromium.org/chromium/src/media/filters/ffmpeg_video_decoder.h)}  
  - Container 파싱과 오디오/비디오 디코딩에 사용되는 오픈소스 라이브러리
- Blink's [HTMLMediaElement](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/core/html/media/HTMLMediaElement.h)  
  - WHATWG에서 지정한 HTML 및 javascript 바인딩을 구현합니다.   
  - 유저 에이전트 컨트롤 렌더링을 처리합니다.
  
### Pipeline
* 파이프라인은 풀 기반(Pull-based) 미디어 재생 엔진 입니다. 
  * 각 단계를 6 가지 필터로 추상화 합니다. 
  * data source, demuxing, audio decoding, video decoding, audio rendering, video rendering.
* 파이프라인은 렌더러의 주기를 관리하고, 스레드 safe 인터페이스를 클라이언트에 제공합니다.
* 필터는 서로 연결 되어 필터 그레프를 형성한다. 

- 디자인 목표:  
  - 크로미움 쓰레드 구조를 사용한다. 예시 [TaskRunner](https://cs.chromium.org/chromium/src/base/task_runner.h)
  - 필터는 쓰레드 모델을 결정하지 않는다.   
  - 모든 필터 동작은 비동기고, 완료를 시그널로 알리는 콜백을 사용한다.   
  - Upstream 필터는 downstream 필터를 인식하지 못합니다. (예, DataSource는 Demuxer를 알지 못 한다.)  
  - 일반적인 타입과 메소드보다 명시적인 타입과 메소드를 선호한다. (예, foo->Bar() 보다 foo->SendMessage(MSG_BAR) )  
  - 안전한 센드박스에서 동작할 수 있어야 한다. 
  - x86, ARM /  윈도우, 맥, 리눅스 / 에서 동작 한다.
  - 임의의 오디오/비디오 코덱을 지원한다. 
 
- 디자인 목표가 아닌 것:  
  - 공유 라이브러리를 통한 필터의 동적 로딩 
  - 버퍼 관리를 조정(Buffer management negotiation)    
  - 임의의 필터 그래프 적용  
  - 미디어 재생을 넘는 필터 지원
  
Chromium의 동영상 지원은 2008년 9월에 시작 되었습니다. 
미디어 재생 엔진 구현을 결정하기 전에는 아래와 같은 기술을 대안으로 고민 했었습니다. 
  - DirectShow (윈도우즈 전용, 해킹 없는 안전한 센드박스에서 실행 안됨)
  - GStreamer (당시 윈도우즈에서 지원할지 의문이였고, 2Mb의 DLL의 의존성이 있었다.)
  - VLC (GPL에 따른 사용 불가)
  - MPlayer (GPL에 따른 사용 불가)
  - OpenMAX (목적에 비해 과도함, 너무 큼)
  - liboggplay (Ogg Theora/Vorbis 전용)
  
우리의 방법은 오디오/비디오 코덱에 관계 없이 재생할 수 있는 미디어 재생 엔진을 작성하는 것입니다. 
FFmpeg를 사용하여 사유/상업용 코덱 사용을 피했으며, 미디어 엔진에서는 FFmpeg의 빌드 구성에 따라 다양한 포멧을 지원할 수 있습니다.


![](https://www.chromium.org/_/rsrc/1259798424559/developers/design-documents/video/video_stack_arch.png)

위에 언급한 것과 같이, 파이프라인은 완전한 불 방식(pull-based)이며, 사운드 카드를 사용하여 재생 한다. 

사운드 카드는 추가 데이터를 요청으로 
* [audio renderer](https://cs.chromium.org/chromium/src/media/base/audio_renderer.h)는 [audio decoder](https://cs.chromium.org/chromium/src/media/base/audio_decoder.h)에게 디코딩된 audo data를 요청합니다. 
  * [audio decoder](https://cs.chromium.org/chromium/src/media/base/audio_decoder.h)는 the [demuxer](https://cs.chromium.org/chromium/src/media/base/demuxer.h)에게 인코딩된 buffers를 요청합니다. 
    * the [demuxer](https://cs.chromium.org/chromium/src/media/base/demuxer.h)는 [data source](https://cs.chromium.org/chromium/src/media/base/data_source.h)에서 읽은 것이다. 
      * 등등...


디코딩된 오디오 데이터는 사운드 카드에 공급되며, 파이프 라인의 글로벌 클럭은 업데이트 됩니다. 
[video renderer](https://cs.chromium.org/chromium/src/media/base/video_renderer.h)는
* 글로벌 클럭 폴링하여 vsync를 확인 한다. 
* vscync를 통해 
  * [video decoder](https://cs.chromium.org/chromium/src/media/base/video_decoder.h)에 디코딩된 프레임을 요청할 시점을 결정한다. 
  * 언제 새로운 프레임을 비디오 디스플레이에 렌더할지 결정한다.   
사운드카드나 오디오 트랙이 없는 경우 시스템 글럭을 사용하여 비디오 디코딩 및 렌더링을 수행한다. 
관련 코드는 [media](https://cs.chromium.org/chromium/src/media/)에 있습니다. 


파이프라인은 재생과 이벤트(예를 들명 일시정지, 이동, 정지)를 처리하기 위해 state 머신을 사용합니다. 
상태 변경은 일반적으로 이벤트의 모든 필터에 알리고, 변경이 완료하기 전에 완료 콜백을 기다리는 것으로 구성됩니다. 
(다이어 그램 출처 [pipeline_impl.h](https://cs.chromium.org/chromium/src/media/base/pipeline_impl.h)) 

![](https://github.com/realbj/realbj.github.io/blob/master/assets/images/playbackState.JPG)

풀 기반(pull-based) 설계에서 일시 정지의 경우 재생 비율(속도)를 0으로 설정하므로써 오디오와 비디오 렌더러가 upstream 필터에게 데이터 요청을 중지하는 방식입니다. 보류중인 요청이 없다면, 전체 파이프라인은 암시적인 "일시 정지" 상태가 됩니다. 


## Integration
아래 다이어그램은 미디어 재생 파이프라인을 WebKit / Chromium 브라우저에서 통합된 것을 보여줍니다. 간 오래되었지만 요점은 동일합니다. 

![](https://www.chromium.org/_/rsrc/1259966540019/developers/design-documents/video/video_stack_chrome.png)

(1) WebKit은 media player를 생성할 것을 요청합니다. chromium의 경우 WebMediaPlayerImpl과 Pipeline을 생성합니다. 

(2) BufferedDataSource는 ResourceLoader에 현재 비디오 URL을 가져 오도록 요청합니다. 

(3) ResourceDispatcher는 브라우저 프로세스에 요청을 전달 합니다. 

(4) URLRequest는 요청을 위해 생성되며, 캐쉬 된 데이터는 HttpCache에 있을 것입니다. 데이터 사용이 가능하면 BufferedDataSource로 전송 됩니다. 

(5) FFmpeg는 오디오/비디오 데이터를 디코딩(decodes)/디먹스(demuxes) 합니다.

(6) 센드박스 때문에 AudioRendererImpl는 오디오 장치를 직접 사용할 수 없기 때문에 브라우저에게 장치 사용을 요청합니다. 

(7) 브라우저는 새로운 오디오장치를 열고, 알맞은 렌더링 프로세스에 audio callback을 전달합니다. 

(8) 새 프레임을 사용할 수 있게 되면 WebKit에 Invalidates가 보내집니다. 
