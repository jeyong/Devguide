# Linux용 S.Bus 드라이버

*Linux용 S.Bus 드라이버*는 linux 기반 autopilot이 시리얼 포트를 이용하는 *Futaba S.Bus receiver*로부터 통해 16 채널까지 접근 가능하게 합니다. 드라이버는 S.Bus 프로토콜을 사용하는 다른 리시버와 같이 동작해야만 합니다. 여기에는 FrSky, RadioLink 그리고 S.Bus 인코더가 있습니다.

시그날 인버터 회로는 장치 시리얼 포트가 리시버로부터 데이터를 읽는 것을 가능하게 하는 것이 필요합니다. (아래에서 설명)

> **Note** 해당 드라이버는 Rasbian Linux를 실행하는 Raspberry Pi에서 테스트했습니다. 리시버에 연결할때는 온보드 시리얼 포트를 통하거나 USB to TTY 시리얼 케이블을 이용합니다. 이것은 시리얼 포트를 통해 모든 Linux 버전에서 동작하리라 예상됩니다.


## 시그날 인버터 회로

S.Bus는 *인버터* UART 통신 신호입니다. 많은 시리얼 포트/비행 제어기가 인버팅된 UART 신호를 읽지 못하므로 리시버와 시리얼 포트 사이에 시그날 인터버 회로가 필요합니다. 이 섹션에서는 적절한 회로를 만드는 방법을 소개합니다.

> **Tip** 이 회로는 Raspberry Pi가 S.Bus 리모트 컨트롤 신호를 시리얼 포트나 USB-to-TTY 시리얼 컨버터를 통해서 읽는데 필요합니다. 이것은 다른 비행 제어기에도 필요합니다.

### 필요한 컴포넌트

* 1x NPN transistor (e.g. NPN S9014 TO92)  
* 1x 10K resistor
* 1x 1K resistor

> **Note** 끌어쓰는 전류가 매우 낮으므로 트랜지스터의 어떤 타입/모델도 사용할 수 있습니다.

<span></span>
> **Tip** Raspberry Pi는 단일 시리얼 포트입니다. 만약 이미 사용하고 있다면 대안으로 S.Bus 리시버를 RaPi USB 포트에 연결할 수 있습니다. 이때 USB-to-TTY 시리얼 케이블을 사용합니다.(예로 PL2302 USB-to-TTY 시리얼 컨버터)


### 회로 다이어그램/연결

컴포넌트들을 아래와 같이 연결합니다.(회로 다이어그램 참고):

* S.Bus signal &rarr; 1K resistor &rarr; NPN transistor base
* NPN transistor emit &rarr; GND
* 3.3VCC &rarr; 10K resistor &rarr; NPN transistor collection &rarr; USB-to-TTY rxd
* 5.0VCC &rarr; S.Bus VCC
* GND &rarr; S.Bus GND

![시그날 인버터 회로 다이어그램](../../assets/driver_sbus_signal_inverter_circuit_diagram.png)


### 빵판 이미지

아래 이미지는 빵판에서 연결을 보여줍니다.

![시그날 인버터 빵판](../../assets/driver_sbus_signal_inverter_breadboard.png)


## 소스 코드

* [Firmware/src/drivers/linux_sbus](https://github.com/PX4/Firmware/tree/master/src/drivers/linux_sbus)

## 사용법

명령 문법:

```
linux_sbus start|stop|status -d <device> -c <channel>
```

예제에서는 자동으로 `/dev/ttyUSB0`에 8개 채널을 listening하는 드라이버를 구동시킵니다. 구동 설정 파일에서 아래와 같이 추가합니다.

```
linux_sbus start -d /dev/ttyUSB0 -c 8
```

> **Note** 원래 설정 파일은 **Firmware/posix-configs** 에 있습니다. 공식 문서에 따르면 `make upload`을 마치면 모든 posix 관련 설정 파일은 **/home/pi** 에 위치하게 됩니다. 사용하고 싶으면 해당 파일을 수정할 수 있습니다.
