# 장거리 비디오 스트리밍 QGroundControl

여기서는 UAV에서 그라운드 컴퓨터로 비디오 스트림을 전송해서 *QGroundControl* 에서 출력될 수 있도록 카메라(Logitech C920나 RaspberryPi camera) 장착한 컴패니온 컴퓨터를 셋업하는 방법에 대해서 알아봅시다. 여기서 셋업은 비연결(broadcast) 모드와 [Wifibroadcast 프로젝트](https://github.com/svpcom/wifibroadcast/wiki)의 소프트웨어 기반의 WiFi를 사용합니다.


## Wifibroadcast 개요

*Wifibroadcast 프로젝트* 는 WiFi 라디오를 사용할때, 아날로그 링크를 이용해서 HD 비디오(와 다른 종류) 데이터를 전송할때의 장점을 가질 수 있게 하는 것을 목표로 합니다. 신호가 약하거나 거리가 멀어지는 경우 비디오 품질이 갑자기 이상이 생기지 않도록 하는 것이 하나의 예제가 될 수 있습니다.

> **Note** *Wifibroadcast*을 사용하기 전에 여러분 지역에서 WiFi 사용 규제를 확인해 보세요.

*Wifibroadcast*의 상위 레벨에서의 이득 :

- 단일 WiFi (IEEE80211) 패킷과 즉각적인 송신(byte stream으로 직렬화되지 않는 경우)에 대해서 모든 수신 RTP 패킷을 인코딩할때 최소 지연
- Smart FEC 지원 (gap이 없는 FEC 파이프라인인 경우에 패킷을 비디오 디코더로 즉각 넘겨주기)
- 스트림 암호화와 인증 ([libsodium](https://download.libsodium.org/doc/))
- 분산 처리. 다른 호스트의 카드로부터 데이터를 모을 수 있고 따라서 대역폭을 단일 USB bus로 제약하지 않아야 함.
- MAVlink 패킷들 합치기. 매 MAVLink 패킷을 위해서 WiFi 패킷을 보내지 않음.
- [강화된 OSD for Raspberry Pi](https://github.com/svpcom/wifibroadcast_osd) (Pi Zero에서 10% CPU 자원 사용).

추가 정보는 아래 [FAQ](#faq)를 확인하세요.


## 하드웨어 셋업

다음 부품들로 하드웨어 셋업 :

TX (UAV)쪽:
* [NanoPI NEO2](http://www.friendlyarm.com/index.php?route=product/product&product_id=180) (Pi 카메라를 사용한다면 Raspberry Pi).
* [Logitech camera C920](https://www.logitech.com/en-us/product/hd-pro-webcam-c920?crid=34) 혹은 [Raspberry Pi camera](https://www.raspberrypi.org/products/camera-module-v2/).
* WiFi 모듈  [ALPHA AWUS051NH v2](https://www.alfa.com.tw/products_show.php?pc=67&ps=241).

RX (ground station 쪽):
* Linux가 돌아가는 아무 컴퓨터 (Fedora 25 x86-64에서 테스트).
* Ralink RT5572 칩셋이 장착된 WiFi 모듈 ([CSL 300Mbit Sticks](https://www.amazon.co.uk/high-performance-gold-plated-technology-Frequency-adjustable/dp/B00RTJW1ZM) 혹은 [GWF-4M02](http://en.ogemray.com/product/product.php?t=4M02)). OEM 모듈은 저렴하지만 중국에서 주문해야함. CSL 스틱은 비싸지만 ebay에서 구입가능. 지원하는 모듈에 관한 정보는 [wifibroadcast wiki > WiFi hardware](https://github.com/svpcom/wifibroadcast/wiki/WiFi-hardware) 참고.


## 하드웨어 수정

Alpha WUS051NH는 전송하는 동안 너무 많은 전류를 끌어다 씁니다. USB로 전원을 공급받는 경우, 대부분 ARM 보드에서 포트를 리셋시킬 수 있습니다. 따라서 5V BEC에 직접 연결하도록 합니다. 다음과 같이 2가지 방법으로 처리할 수 있습니다.:

1. 커스텀 USB 케이블 만들기 [USB 플러그에서 ``+5V`` 전선을 잘라서 BEC에 연결하기](https://electronics.stackexchange.com/questions/218500/usb-charge-and-data-separate-cables)
2. PCB에 ``+5V`` 전선을 잘라서 BEC에 연결하기. 잘 모르면 따라하지 마세요. 커스텀 케이블을 사용하세요!
   전원과 그라운드 사이에 전압 스파크가 발생하지 않도록 470uF low ESR 캐패시터(ESC에 달려있는 것처럼)을 추가하는 것을 추천합니다. 여러개 그라운드 전선을 사용하는 경우 [ground loop](https://en.wikipedia.org/wiki/Ground_loop_(electricity))을 참고하세요.


## 소프트웨어 셋업

1. **libpcap**와 **libsodium** 개발 lib를 설치
2. [wifibroadcast sources](https://github.com/svpcom/wifibroadcast) 다운로드
3. 커널 [Patch](https://github.com/svpcom/wifibroadcast/wiki/Kernel-patches)하기. TX쪽에 커널만 패치하면 됨(CRDA에 따라서 여러분의 지역에서 사용할 수 없는 WiFi 채널을 사용하는 경우는 제외)

### 암호화 키 생성

```
make
keygen
```
``rx.key``를 RX 호스트에 복사하고 ``tx.key``는 TX 호스트에 복사하세요.

### UAV 셋업 (TX)

1. RTP 스트림을 출력할 수 있도록 카메라 셋업:

   a. Logitech camera C920 카메라:
      ```
      gst-launch-1.0 uvch264src device=/dev/video0 initial-bitrate=6000000 average-bitrate=6000000 iframe-period=1000 name=src auto-start=true \
               src.vidsrc ! queue ! video/x-h264,width=1920,height=1080,framerate=30/1 ! h264parse ! rtph264pay ! udpsink host=localhost port=5600
      ```
   b. RaspberryPi 카메라:
      ```
      raspivid --nopreview --awb auto -ih -t 0 -w 1920 -h 1080 -fps 30 -b 4000000 -g 30 -pf high -o - | gst-launch-1.0 fdsrc ! h264parse !  rtph264pay !  udpsink host=127.0.0.1 port=5600
      ```

2. TX 모드에서 *Wifibroadcast* 셋업하기:

   ```
   git clone https://github.com/svpcom/wifibroadcast
   cd wifibroadcast
   make
   ./scripts/tx_standalone.sh wlan1   # where wlan1 is your WiFi TX interface
   ```
   이렇게 하면 149 WiFi 채널(5GHz 밴드)에서 `MCS #1: QPSK 1/2 40MHz Short GI` 모듈레이션(30 Mbit/s)을 사용하고 UDP 포트 5600로 입력 데이터를 listening하는 wifibroadcast를 설정할 수 있습니다.

### Ground Station 셋업 (RX)

1. RX 모드로 *Wifibroadcast* 셋업하기:
   ```
   git clone https://github.com/svpcom/wifibroadcast
   cd wifibroadcast
   make
   ./scripts/rx_standalone.sh wlan1  # your WiFi interface for RX
   ```

2. 비디오 디코딩을 위해서 qgroundcontrol 실행하기
   ```
   gst-launch-1.0 udpsrc port=5600 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264' \
             ! rtph264depay ! avdec_h264 ! clockoverlay valignment=bottom ! autovideosink fps-update-interval=1000 sync=false
   ```

## RX 안테나 배열, FPV 고글과 OSD를 위한 셋업

[wiki](https://github.com/svpcom/wifibroadcast/wiki/enhanced-setup) 글을 참고하세요.
위에(TX로 ALPHA AWUS051NH v2) RX 셋업을 사용하면 어떠한 copter 피치/롤 각도라도 1-2km 떨어진 거리에서도 안정적인 1080p 비디오 수신이 가능했습니다.



## FAQ


#### 장거리 비디오 전송에 일반 WiFi를 사용하는 경우의 제약사항은?

일반 WiFi를 장거리 비디오 전송에 사용하는 경우 다음과 같은 문제가 있습니다:

- **연결(Association):** 비디오 트랜스미터와 리시버는 연결되어 있어야합니다. 만약 하나의 장치가 연결을 끊어지면(신호 강도가 약한 경우) 비디오 전송이 바로 멈추게 됩니다.

- **Two-way 통신:** 소스에서 데이터를 전송하더라도 WiFi를 사용하려면 양방향 data flow가 필요합니다. 이런 이유는 WiFi 리시버가 수신한 패킷에 대한 정보를 알려줘야 하기 떄문입니다. 만약 트랜스미터가 ack르 받지 못한 경우 연결을 끊게 됩니다. 따라서 비행체와 그라운드 스테이션 모두에게 강한 신호의 트랜스미터와 안테나가 필요합니다. 비행체에서는 단방향 안테나를 사용하고 지상 장치는 gain이 높은 안테나로 구성하는 경우 일반 WiFi와 궁합이 맞지 않는다.

- **Rate control:** 일반 WiFi 연결시 신호 강도가 너무 약하면 자동으로 낮은 전송 속도로 변환됩니다. 결국 신호가 너무 약해서 비디오 데이터를 전송하지 못하는 경우 자동으로 전송속도를 선택하는 것입니다. 이렇게 되면 해당 데이터는 큐에 쌓이고 몇 초까지 이상 지연 현상이 생길 수 있습니다.

- **1:1 전송:** broadcast 프레임이나 유사한 기술을 사용하지 않는다면, 일반 WiFi data flow는 1:1 연결입니다. 제3자가 스트림을 볼려고 "채널"을 추적하는 시나리오는(아날로그 비디오 전송에서는 가능) 기존 WiFi을 사용을 통해 달성하는 것이 쉽지 않다.

- Limited diversity: 일반 WiFi의 경우 사용하는 WiFi 카드가 제공하는 다이버스티 스트림의 수로 제한됩니다.

#### Wifibroadcast가 위와 같은 제약을 극복하는 방식

*Wifibroadcast* 는 WiFi 카드를 모니터 모드로 설정합니다. 이 모드에서는 연결이 되지 않아도 임의의 패킷을 주고받기가 가능합니다. 이 방식은 실제로 단방향 연결로 설정되어 아날로그 링크의 장점을 사용할 수 있습니다. 다음과 같은 기능 :

- 트랜스미터는 연결된 리시버에 상관없이 데이터를 전송합니다. 따라서 연결이 끊어지는 영향으로 비디오가 갑자기 멈추는 위험이 없습니다.
- 리시버는 트랜스미터가 허용하는 범위에서 비디오 수신이 가능합니다. 범위 밖으로 벗어나는 경우 천천히 화질이 떨어지게 됩니다.
- 기존 개념 “1개 broadcaster – 여러 리시버”은 기본 기능입니다. 만약 제 3자가 그들의 장치로 비디오 스트림을 볼려고 하는 경우 그냥 "채널 맞추기"만 하면 됩니다.
- *Wifibroadcast* 를 사용하면 값싼 리시버를 병렬로 사용하고 데이터를 모아서 올바른 데이터 수신의 가능성을 높입니다. 소프트웨어 다이버시티로 상호보완 및 신뢰성을 높이기 위해서 동일한 리시버를 사용할 수 있습니다.(360° 단뱡향 안테나로 된 리시버와 여러 양방향 안테나로 모두 장거리용 병렬로 동작)
- *Wifibroadcast* 는 FEC(Forward Error Correction)을 사용해서 낮은 대역에서도 높은 신뢰성을 가집니다. 리시버에서 잃어버리거나 오류가 생긴 패킷을 복구하는 것이 가능합니다.

#### 새로운 Wifibroadcast가 원래 프로젝트와 다른 점은?

[ wifibroadcast의 원래 버전](https://befinitiv.wordpress.com/wifibroadcast-analog-like-transmission-of-live-video-data/)은 [현재 project](https://github.com/svpcom/wifibroadcast/wiki)와 동일한 이름을 공유하지만 코드를 가지고 온 것은 아닙니다.

원래 버전은 입력으로 byte-stream을 사용했고 고정 길이의 패킷으로 분해합니다.(기본적으로 1024) 라디오 패킷을 잃어버리고 FEC로 복구가 되지 않으면 스트림에서 임의의 (예상치 못한) 곳에 구멍이 생길 수 있다. 만약 데이터 프로토콜이 이런 임의의 삭제에 대한 대비가 없다면 좋지 않은 결과를 얻게 됩니다.

새로운 버전은 데이터 소스로 DUP를 사용하기 위해서 재작성했고 하나의 소스 UDP 패킷을 하나의 라디오 패킷으로 패킹합니다. 라디오 패킷은 이제 payload 길이에 따라서 가변 길이로 패킹됩니다. 이렇게 하면 비디오 지연을 많이 줄일 수 있습니다.

#### Wifibroadcast을 사용해서 전송하는 데이터의 타입은?

패킷 사이즈가 <= 1466인 어떤 UDP라도 가능합니다. 예제로 RTP나 MAVLink 내부에 x264.

#### 전송 보장이란?

*Wifibroadcast* uses Forward Error Correction (FEC) which can recover 4 lost packets from a 12 packet block with default settings.
You can tune both TX and RX (simultaneously) to fit your needs.

#### 프레임 드롭과 `XX packets lost` 메시지의 원인은?

다음과 같은 원인:
1. 신호 세기가 너무 낮은 경우. 높은 전원이 필요한 카드나 높은 gain이 필요한 안테나를 사용하세요. RX쪽에서 방향성 안테나를 사용하세요. 다이버시티를(wlan2, wlan3, ...를 RX 프로그램에 추가) 위해서 추가 RX 카드를 사용하세요.
2. 신호 세기가 너무 낮은 경우. 특히 인도어에서 30dBm TX를 사용하는 경우라면 TX 파워를 줄이세요(예제로 커널 내부에 CRDA 데이터베이스를 해킹하고 전원 10dBm과 20dBm 제약으로 여러 구역을 만들 수 있음)
3. 다른 WiFi와 간섭. WiFi 채널이나 밴드 변경.

   > **Caution** RC TX가 동작하는 밴드는 사용하지 마세요! 모델 loss를 회피하기 위해서 적절하게 RTL을 셋업하세요.

지연이 증가하는 대신 FEC 블록 사이즈를(기본적으로 8/12 - 8 data blocks과 4 FEC blocks) 증가시킬 수 있습니다. 다이버시티를(wlan2, wlan3을 RX 프로그램에 추가) 위해서 추가 RX 카드르 사용합니다.


#### UAV에 추천하는 ARM 보드는?

Board | Pros  | Cons
--- | --- | ---
[Raspberry Pi Zero](https://www.raspberrypi.org/products/raspberry-pi-zero/) | - Huge community<br>- Camera support<br>- HW video encoder/decoder with OMX API. | - Hard to buy outside US (shipping costs >> its price)<br>- Slow CPU<br>- Only one USB bus<br>- 512MB SDRAM
[Odroid C0](http://www.hardkernel.com/main/products/prdt_info.php?g_code=G145326484280) | - Fast CPU<br>- EMMC<br>- 1GB SDRAM | - Very sensitive to radio interference<br>- Doesn't supported by mainline kernel<br>- High cost<br>- HW video encoder is broken<br>- Bad PCB quality (too thin, ground pins without [thermal relief](https://en.wikipedia.org/wiki/Thermal_relief))
[NanoPI NEO2](http://www.friendlyarm.com/index.php?route=product/product&product_id=180) | - ARM 64-bit CPU<br>- Very cheap<br>- Supported by mainline kernel<br>- 3 independent USB busses<br>- 1Gbps Ethernet port<br>- 3 UARTs<br>- Very small form-factor<br>- Resistant to radio interference | - Small community<br>- 512MB SDRAM<br>- No camera interface

여기서는 카메라 보드로 Pi Zero를 사용하고 메인 UAV 보드로(wifibroadcast, MAVLink 텔레메트리 등) NEO2를 사용합니다.


## TODO

1. 미리 빌드한 패키지 만들기. PR 환영.
2. 다른 카드/안테나로 비행 테스트하기
3. CRDA 해킹하지 않고 TX 전원설정하는 방법 조사
4. 라디오 링크 RSSI를 가지는 패킷을 MAVLink stream에 추가
