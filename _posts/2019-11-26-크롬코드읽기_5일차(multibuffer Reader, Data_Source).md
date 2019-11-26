# multibuffer_reader / multibuffer_data_source
## reader public 함수 내용들.
* Available  : AvailableAt(현재위치)
* AvailableAt : 읽기 가능한 byte를 리턴, -1이면 에러
* Seek : 위치 이동, 내부 상태 점검(확인)을 위한 wait 동작이 있음을 고려해야 한다.
* Tell : 현재 위치 리턴
* TryReadAt : 특정 위치(pos)에서 len 만큼 읽어 data에 전달, 실제 읽은 byte 수를 리턴한다.
* TryRead : 현재 위치에서 len 만큼 읽어 data에 전달, 실제 읽은 byte 수를 리턴한다. TryReadAt와 달리 추가로 읽은 만큼 seek 한다!
* Wait : len 만큼 읽을 수 있을 때까지 기다린다. 완료시 net:OK, 문제 발생시 net:ERR_IO_PENDING  (Callback 관련 처리가 있다)
* SetPreload : preload 할 최대,최소 범위를 설정 한다. 
* SetPinRange : cache에서 고정(유지) 시킬 범위를 설정한다. 
* SetMaxBuffer : 얼마나 메모리를 사용할지 설정한다. 
* IsLoading : 현재 data를 로딩하고 있으면 true
* NotifyAvailableRange : 가용 범위가 달라지면 (아마, 버퍼에 데이터가 들어오면) 미리 정의된 callback을 실행
## data_source
* Initialize : reader가 준비 되었다면 startcallback을 바로 실행하고 아니라면 reader의 준비를 기다림
* MediaPlaybackRateChanged : playback_rate를 변경하고 버퍼 상태를 update  한다.
* MediaIsPlaying : play 상태로 변환한다.  
* OnBufferingHaveEnough :  ??
* GetMemoryUsage : url_data_ 를 통해 연산된 메모리 사이즈를 리턴한다.
* Stop
* Abort
* Read : Reader가 TryReadAt에 성공하면 다음 read을 위한 SeekTask를 bind 한다. 
* GetSize : total_bytes_를 회신, 조건에 따라 0을 회신 할 수 있다 (회신 조건에 따라 T/F 리턴)

## 기타 공부 꺼리
* base::Bind, base:BindOnce
* postTask
* base::AutoLock
