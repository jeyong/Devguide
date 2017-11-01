# 짐벌 제어 셋업(Gimbal Control Setup)

비행체에 카메라를 장착한 짐벌을 제어하고 싶다면, 어떻게 제어할 것인지 또 PX4가 명령을 어떻게 줄 것인지에 대한 설정이 필요합니다. 여기서는 셋업 방법을 설명합니다.

PX4는 서로 다른 입력/출력 방식을 가진 일반적인 마운트/짐벌 제어 드라이버가 포함되어 있습니다. 입력은 짐벌을 제어하는 방법을 정의하며 RC나 MAVLink 명령을 통해서 가능합니다.(mission이나 surveys에서 설명) 출력은 짐벌이 어떻게 연결되어 있는지 정의합니다. : 일부는 MAVLink 명령을 지원하고 이외에는 PWM을 사용합니다.(아래 AUX 출력에서 설명) 어떤 형태의 입력이든 선택한 드라이버로 출력이 가능합니다. 모두 파라미터를 통해 설정해야만 합니다.

## 파라미터(Parameters)
[이 파라미터](../advanced/parameter_reference.md#mount)는 마운트 드라이버를 셋업하는데 사용합니다. 가장 중요한 것은 입력(MNT_MODE_IN)과 출력(MNT_MODE_OUT) 모드입니다. 기본적으로 입력은 비활성화되어 있고 드라이버도 실행되지 않습니다. 입력 모드를 선택하고 나서 비행체를 리부팅하면 마운트 드라이버가 시작됩니다.

입력 모드를 `AUTO`로 설정하면, 모드는 자동으로 마지막 입력을 기반으로 자동으로 전환됩니다. mavlink에서 RC로 변환시키기 위해서는 큰 스틱 모션이 필요합니다.

## AUX 출력

출력 모드가 `AUX`로 되어 있다면, 믹서 파일은 출력 핀에 대해서 매핑되어 있어야 하고 [mount mixer](https://github.com/PX4/Firmware/blob/master/ROMFS/px4fmu_common/mixers/mount.aux.mix)는 자동으로 선택되어 있어야 합니다.(airframe 설정으로 aux 믹서를 사용하지 못하게 함)

할당된 출력은 다음과 같습니다:
- **AUX1**: Pitch
- **AUX2**: Roll
- **AUX3**: Yaw
- **AUX4**: Shutter/retract

### 믹서 설정 커스터마이징
> **Note** 믹서가 어떻게 동작하는지 그리고 믹서 파일의 포맷에 대해서는 [Mixing and Actuators](../concept/mixing.md)을 참고하세요.

출력은 SD 카드에 있는 `etc/mixers/mount.aux.mix` 파일을 [믹서 파일 생성하기](../advanced/system_startup.md#starting-a-custom-mixer)로 커스터마이징할 수 있습니다.

마운트에 대해서 기본 믹서 설정은 다음과 같습니다.

```
# roll
M: 1
O:      10000  10000      0 -10000  10000
S: 2 0  10000  10000      0 -10000  10000

# pitch
M: 1
O:      10000  10000      0 -10000  10000
S: 2 1  10000  10000      0 -10000  10000

# yaw
M: 1
O:      10000  10000      0 -10000  10000
S: 2 2  10000  10000      0 -10000  10000
```


## SITL

Typhoon H480 모델은 설정된 시뮬레이션 짐벌에 포함되어 있습니다. 이를 실행하기 위해서 다음을 사용합니다:
```
make posix gazebo_typhoon_h480
```

다른 모델이나 시뮬레이터에 있는 마운트 드라이버를 테스트하기 위해서, `vmount start`를 사용해서 드라이버가 실행되었는지 확인하고 파라미터를 설정합니다.  


## 테스팅
드라이버는 간단한 테스트 명령을 제공합니다. - 먼저 `vmount stop`로 정지시킵니다. 다음은 SITL에서 테스팅하는 방법을 설명합니다. 하지만 실제 장치엥서도 동작합니다.

시뮬레이션을 다음과 같이 시작시킵니다(파라미터를 변경하지 않아도 됩니다):
```
make posix gazebo_typhoon_h480
```
arm 상태가 되었는지 확인합니다. `commander takeoff` 명령을 사용할 수 있으며 다음으로 다음 명령으로 짐벌을 제어합니다.
```
vmount test yaw 30
```
시뮬레이션한 짐벌은 자체로 스테빌라이저가 됩니다. mavlink 명령을 보내서 `stabilize` flag가 false로 설정됩니다.

![Gazebo Gimbal Simulation](../../assets/gazebo/gimbal-simulation.png)
