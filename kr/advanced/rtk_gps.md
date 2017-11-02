# RTK GPS (백그라운드)

[Real Time Kinematic](https://en.wikipedia.org/wiki/Real_Time_Kinematic) (RTK)은 센티미터 단위 GPS 정확성을 제공합니다. 여기서는 RTK가 PX4로 어떻게 통합되는지를 설명합니다.

> **Note** RTK GPS *사용* 에 관련된 설명은 [PX4 사용자 가이드](https://docs.px4.io/en/advanced_features/rtk-gps.html)를 참고하세요.

## 개요

RTK는 신호의 정보 컨텐츠라기 보다는 신호의 carrier wave의 phase를 특정하는 방식입니다. 실시간 보정을 제공하려면 단일 레퍼런스 스테이션에 의존합니다. 이는 여러 모바일 스테이션과도 동작시킬 수 있습니다.

2개 RTK GPS 모듈들과 데이터링크는 PX4와 RTK 셋업이 필요합니다. 고정 위치 그라운드 기반 GPS 유닛을 *Base* 라고 부르고 공중에 있는 유닛을 *Rover* 라고 부릅니다.
Base 유닛을 *QGroundControl* 에 연결하고(USB사용) RTCM 보정을 비행체로 보내기 위해서 데이터링크를 사용합니다.(MAVLink `GPS_RTCM_DATA` 메시지 사용)
autopilot에서 MAVLink 패킷을 풀고 Rover 유닛으로 전송하며 여기서 RTK 솔루션을 얻기 위해 처리되는 지점입니다.

데이터링크는 일반적으로 uplink 속도는 초당 300 bytes로 처리할 수 있습니다.(상세한 내용은 아래 [Uplink Datarate](#uplink-datarate)을 참고하세요)

## RTK GPS 모듈 지원

PX4는 RTK로 현재 단일 주파수 (L1) u-blox M8P 기반 GNSS 리시버만 지원합니다.

많은 제조사가 이 리시버를 이용해서 제품을 생산하고 있습니다. 테스트한 장치 목록은 [사용자 가이드](https://docs.px4.io/en/advanced_features/rtk-gps.html#supported-rtk-devices)를 참고하세요.

> **Note** u-blox은 2가지 M8P 칩 계열을 가지고 있습니다. M8P-0과 M8P-2입니다. M8P-0은 Rover로만 사용이 가능하며 M8P-2는 Rove나 Base로 사용이 가능합니다.


## 자동 설정

PX4 GPS 스택은 자동으로 u-blox M8P 모듈을 설정하여 UART나 USB를 통해 보정 메시지를 주고 받습니다. 해당 모듈은 *QGroundControl* 나 autopilot에 연결에 따라 연결방법이 달라집니다.

autopilot이 `GPS_RTCM_DATA` mavlink 메시지를 수신하자마자, 자동으로 RTCM 데이터를 연결된 GPS 모듈로 포워딩합니다.

> **Note** U-Center RTK 모듈 설정 도구를 사용하지 않습니다!

<span></span>
> **Note** *QGroundControl* 와 autopilot 펌웨어 모두 동일한 [PX4 GPS driver stack](https://github.com/PX4/GpsDrivers)를 공유합니다. 실제로 이 말은 새로운 프로토콜 메시지를 지원하기 위해서는 한 곳에 추가하기만 하면 된다는 뜻입니다.


### RTCM messages

QGroundControl는 다음 RTCM3.2 프레임 출력으로 RTK base station을 설정합니다.각각은 1Hz로 :

- **1005** - Station coordinates XYZ for antenna reference point (Base position).
- **1077** - Full GPS pseudo-ranges, carrier phases, Doppler and signal strength (high resolution).
- **1087** - Full GLONASS pseudo-ranges, carrier phases, Doppler and signal strength (high resolution).


## Uplink 데이터 속도

base에서 수신한 원래 RTCM 메시지는 MAVLink `GPS_RTCM_DATA` 메시지로 패킹되어 있고 datalink로 전송됩니다. 각 MAVLink message의 최대 길이는 182 bytes입니다. RTCM message에 따르면 MAVLink message는 거의 완전히 채워지지는 않습니다.

RTCM base position 메시지 (1005)는 길이가 22 bytes입니다. 반면에 다른 메시지는 위성의 숫자나 위성으로부터의 신호 숫자에(M8P와 같은 L1 유닛에는 1만) 따라서 가변길이입니다. 주어진 시간에서 단일 constellation으로부터 탐지 가능한 위성의 _최대_ 수는 12이고 이론적으로 uplink 속도는 300 B/s면 충분합니다.

*MAVLink 1* 을 사용하면, 182-byte `GPS_RTCM_DATA` 메시지는 모든 RTCM 메시지에서 전송되며 길이와는 상관이 없습니다. 결과적으로 대략 uplink 요구사항이 초당 700+ bytes이 됩니다. 이 경우 낮은 대역폭의 half-duplex 텔레메트리 모듈에서 링크 포화가 될 수 있습니다.(예로 3DR 텔레메트리 라디오)

*MAVLink 2* 를 사용하면 `GPS_RTCM_DATA message`에서 빈공간을 제거하게 됩니다. uplink 요구사항은 결과적으로 이론적인 값과 동일하게 됩니다.(초당 ~300 bytes)

> **Tip** GCS와 텔레메트리 모듈이 MAVLink 2를 지원하면 PX4는 자동으로 MAVLink 2로 스위치됩니다.

양질의 RTK 성능을 위해서 MAVLink 2는 반드시 낮은 대역폭 링크에서 사용해야만 합니다. 텔레메트리 체인이 MAVLink 2 출력을 내는지 각별한 주의가 필요합니다. 시스템 콘솔에서 `mavlink status` 명령을 사용함으로 프로토콜 버전을 검증합니다. :

```
nsh> mavlink status
instance #0:
        GCS heartbeat:  593486 us ago
        mavlink chan: #0
        type:           3DR RADIO
        rssi:           219
        remote rssi:    219
        txbuf:          94
        noise:          61
        remote noise:   58
        rx errors:      0
        fixed:          0
        flow control:   ON
        rates:
        tx: 1.285 kB/s
        txerr: 0.000 kB/s
        rx: 0.021 kB/s
        rate mult: 0.366
        accepting commands: YES
        MAVLink version: 2
        transport protocol: serial (/dev/ttyS1 @57600)
```
