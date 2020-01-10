---
title:  "Chromium 문서 - GN Quick Start guide 1"
excerpt: "Build.gn 파일 문법 빨리 이해하기 1"

categories:
  - Chromium
tags:
  - 크로미움
  - Chromium
  - build
last_modified_at: 2020-01-01T17:30:00
---

# Running GN
* cui(command line)에서 gn을 실행 시킬 수 있다.
* 스크립트는 depot_tools에 위치한다. 
* 스크립트가 실행되면 현재 경로를 포함한 소스 트리에서 바이너리를 찾고, 실행 시킨다. 

# Setting up a build
* GYP에서, 시스템은 Debug나 Release 빌드 디렉토리를 생성하고, 적절한 설정을 한다. 
* GN은 이런 것을 하지 않는다. 
* 대신, 무엇이 빌드 디렉토리이고, 어떤 설정을 원하는지 직접 set up해야 한다. 
* Ninja 파일은 빌드를 할 때, 오래된 상태라면, 자동으로 재생성된다. 
* 빌드 디렉토리 만들기
   * ``` gn gen out/my_build ```

# Passing build arguments
* 빌드 경로에 대한 빌드변수 설정하기 
   * ``` gn args out/my_build ```
* 빌드변수를 파일에 작성할 때 
```
is_component_build = true 
is_debug = false 
```
* 빌드 경로에 설정된 변수 값(초기값) 확인하기 
```
gn args --list out/my_build
```

# Cross-compiling to a target OS or architecture
``` gn args out/Defult ```를 실행 하고, 하나 이상의 옵션을 추가할 수 있습니다.

참고) ``` gn args out/Defult ```를 실행하면 설정 내용을 변경 할 수 있는 editor가 실행 됩니다. 
```
target_os ="chromeos"
target_os = "android"

target_cpu = "arm"
target_cpu = "x86"
target_cpu = "x64"
```
* 상세한 크로스 컴파일 정보 -> [GNCrossCompiles](https://chromium.googlesource.com/chromium/src/tools/gn/+/48062805e19b4697c5fbd926dc649c78b6aaa138/docs/cross_compiles.md)

# Configuring goma 
* 필요시 ``` gn args out/Defult ```를 실행 하고 다음을 추가
```
use_goma = true
goma_dir = "~/foo/bar/goma"
```
* goma가 기본 경로(~/goma)에 위치한다면, goma_dir를 생략할 수 있다.

# Configuring component mode
* ``` gn args out/Defult ```를 실행 하고 다음을 추가
```
is_component_build = true
```


