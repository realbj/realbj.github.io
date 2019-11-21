IntervalMap
====
* file
  * /src/media/blink/interval_map_unittest.cc
  * /src/media/blink/interval_map.h
  <br>  
* /media/blink에서 사용되는 template로 만든 object.
* std::map를 사용하여 만들어짐
* 특징: 특정 범위에 대해 값을 설정하거나, 증가감 시킬 수 있다.
<br>
* test code를 통해 method의 의도를 파악 할 수 있다.
<br>

```c
/* 비교를 위해 작성된 test code */
void IncrementInterval(int32_t from, int32_t to, int32_t how_much) {
    for (int32_t i = from; i < to; i++) {
      data_[i] += how_much;
    }
  }

  void SetInterval(int32_t from, int32_t to, int32_t how_much) {
    for (int32_t i = from; i < to; i++) {
      data_[i] = how_much;
    }
  }

  int32_t operator[](int32_t index) const { return data_[index]; }

```

