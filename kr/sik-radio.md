# SiK Radio

SiK 라디오용 하드웨어는 여러 스토어에서 구매가 가능합니다.

### 벤더

* [3DR Radio](https://store.3drobotics.com/products/3dr-radio-set) \(small\)
* [HK Radio](http://www.hobbyking.com/hobbyking/store/uh_viewitem.asp?idproduct=55559) \(small\)
* [RFD900u](http://rfdesign.com.au/products/rfd900u-modem/) \(small\)
* [RFD900](http://rfdesign.com.au/products/rfd900-modem/) \(장거리\)

![](../../assets/sik_radio.jpg)

## 빌드 방법

PX4 툴체인은 기본적으로 필요한 8051 컴파일러를 설치하지 않습니다.

### Mac OS

툴체인 설치

```
brew install sdcc
```

표준 SiK 라디오 / 3DR Radio용 이미지 빌드하기:

```sh
git clone https://github.com/LorenzMeier/SiK.git
cd SiK/Firmware
make install
```

이를 라디오로 업로드하기  \(**serial 포트 이름은 변경하세요**\):

```
tools/uploader.py --port /dev/tty.usbserial-CHANGETHIS dst/radio~hm_trp.ihx
```

## 설정 방법

라디오는 설정을 위해서 AT 명령을 지원합니다.

```
screen /dev/tty.usbserial-CHANGETHIS 57600 8N1
```

다음으로 명령 모드를 구동시킵니다:

> **Note** DO NOT TYPE ANYTHING ONE SECOND BEFORE AND AFTER

```
+++
```

현재 셋팅 목록:

```
ATI5
```

다음으로 net ID를 설정하고 셋팅을 write하고 라디오를 리부팅 시킵니다:

```
ATS3=55
AT&W
ATZ
```

> **Note** 2번째 라디오로 연결하기 위해서는 라디오의 전원을 다시 끄고켜기를 해야할 수도 있습니다.
