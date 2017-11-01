# 디버그 값 주고 받기

소프트웨어 개발하면서 중요한 값들을 출력하는 것이 필요합니다. 여기서는 MAVLink의 일반 `NAMED_VALUE_FLOAT`, `DEBUG` 그리고 `DEBUG_VECT` 패킷에 대해서 알아봅니다.

## MAVLink 디버그 메시지와 uORB Topics 매핑하기

MAVLink 디버그 메시지를 uORB topic으로 상호변환합니다. MAVLink 디버그 메시지를 주고 받기 위해서는 각각 대응되는 topic으로 publish나 subscribe해야만 합니다.
MAVLink 디버그 메시지와 uORB topic간 매핑 요약 테이블은 다음과 같습니다. :

|  MAVLink message  |    uORB topic   |
|-------------------|-----------------|
| NAMED_VALUE_FLOAT | debug_key_value |
| DEBUG             | debug_value     |
| DEBUG_VECT        | debug_vect      |

## 튜터리얼: String / Float Pairs 보내기

여기서는 관련된 uORB topic `debug_key_value`을 사용해서 MAVLink 메시지 `NAMED_VALUE_FLOAT`을 보내는 방법을 보여줍니다.

이 튜터리얼에 대한 코드는 다음에서 찾을 수 있습니다:

  * [Debug 튜터리얼 Code](https://github.com/PX4/Firmware/blob/master/src/examples/px4_mavlink_debug/px4_mavlink_debug.c)
  * [튜터리얼 app 사용하기](https://github.com/PX4/Firmware/tree/master/cmake/configs) 보드의 config에 있는 mavlink debug app을 커멘트 처리를 제거해서 활성화시키세요.

debug publication을 셋업은 다음과 같이 해주면 됩니다. 먼저 헤더 파일을 추가합니다:

<div class="host-code"></div>

```C
#include <uORB/uORB.h>
#include <uORB/topics/debug_key_value.h>
```

다음으로 debug value topic을 advertise합니다.(다른 published name에 대해서 advertisement 하나면 충분합니다) main 루프 앞에 다음을 추가합니다.:

<div class="host-code"></div>

```C
/* advertise debug value */
struct debug_key_value_s dbg = { .key = "velx", .value = 0.0f };
orb_advert_t pub_dbg = orb_advertise(ORB_ID(debug_key_value), &dbg);
```

그리고 main 루프에서 보내기는 더 간단합니다.:

<div class="host-code"></div>

```C
dbg.value = position[0];
orb_publish(ORB_ID(debug_key_value), pub_dbg, &dbg);
```

> **Caution** 여러 debug 메시지들은 Mavlink가 개별적으로 publish할 수 있도록 충분한 시간이 주어져야 합니다. 이 말은 해당 코드는 여러 debug message들이 publish되는 사이를 기다리거나 각 함수 호출에서 번걸아가면서 처리해야한다는 뜻입니다.

결과적으로 QGroundControl에서는 실시간 챠트로 아래와 같이 보입니다:

![](../../assets/gcs/qgc-debugval-plot.jpg)


## 튜터리얼: String / Float Pairs 수신하기

다음 코드 예제는 이전 튜터리얼에서 보낸 `velx` 디버그 값을 수신하는 방법을 보여줍니다.

먼저 topic `debug_key_value`을 subscribe합니다:

<div class="host-code"></div>

```C
#include <poll.h>
#include <uORB/topics/debug_key_value.h>

int debug_sub_fd = orb_subscribe(ORB_ID(debug_key_value));
[...]
```

다음으로 해당 topic을 poll합니다.:

<div class="host-code"></div>

```C
[...]
/* one could wait for multiple topics with this technique, just using one here */
px4_pollfd_struct_t fds[] = {
    { .fd = debug_sub_fd,   .events = POLLIN },
};

while (true) {
    /* wait for debug_key_value for 1000 ms (1 second) */
    int poll_ret = px4_poll(fds, 1, 1000);

    [...]
```

`debug_key_value` topic에서 새로운 메시지가 유효한 상태가 되면, `velx`와 다른 key를 가진 메시지를 제거하기 위해서 key 속성을 기반으로 필터링하는 것을 잊지마세요.:

<div class="host-code"></div>

```C
    [...]
    if (fds[0].revents & POLLIN) {
        /* obtained data for the first file descriptor */
        struct debug_key_value_s dbg;

        /* copy data into local buffer */
        orb_copy(ORB_ID(debug_key_value), debug_sub_fd, &dbg);

        /* filter message based on its key attribute */
        if (strcmp(_sub_debug_vect.get().key, "velx") == 0) {
            PX4_INFO("velx:\t%8.4f", dbg.value);
        }
    }
}

```
