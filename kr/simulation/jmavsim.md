# jMAVSim (SITL)

jMAVSim은 간편한 멀티로터/쿼드 시뮬레이터로 PX4가 실행되는 *copter* 타입의 비행체를 시뮬레이션 환경에서 비행시켜볼 수 있습니다. 셋업하기 쉽고 이륙, 비행, 착륙 및 여러 실패 상황에서 제대로 동작하는지를 테스트하는데 사용할 수 있습니다.(예로 GPS 실패 상황)

<strong>지원하는 비행체:</strong>

* 쿼드(Quad)

여기서는 PX4의 SITL 버전에 연결하기 위해 jMAVSim 설정하는 방법에 대해서 알아봅니다.

> **Tip** jMAVSim은 HITL 시뮬레이션에서도 사용할 수 있습니다. ([여기 참조](../simulation/hitl.md#using-jmavsim-quadrotor))


## 시뮬레이션 환경

SITL 시뮬레이션은 호스트 머신에서 완전한 시스템을 실행시켜 autopilot을 시뮬레이션할 수 있습니다. 로컬 네트워크를 통해 시뮬레이터에 연결할 수 있습니다. 설정은 다음과 같습니다:

{% mermaid %}
graph LR;
  Simulator-->MAVLink;
  MAVLink-->SITL;
{% endmermaid %}

## SITL 실행하기

[simulation 사전준비](../setup/dev_env.md)가 시스템에 제대로 설치된 것을 확인한 후에 실행만 해주면 됩니다: 편리하게 make로 POSIX 호스트 빌드와 시뮬레이션을 실행할 수 있습니다.

```sh
make posix_sitl_default jmavsim
```

PX4 쉘이 나타납니다.:

```sh
[init] shell id: 140735313310464
[init] task name: px4

______  __   __    ___
| ___ \ \ \ / /   /   |
| |_/ /  \ V /   / /| |
|  __/   /   \  / /_| |
| |     / /^\ \ \___  |
\_|     \/   \/     |_/

Ready to fly.


pxh>
```

## 주요 파일

  * startup 스크립트는 [posix-configs/SITL/init](https://github.com/PX4/Firmware/tree/master/posix-configs/SITL/init) 폴더에 있는 `rcS_SIM_AIRFRAME` 형태의 이름을 갖고 있습니다. 기본은 `rcS_jmavsim_iris` 입니다.
  * 루트 파일 시스템(`/`로 나타나는)은 빌드 디렉토리 내부에 위치하고 있습니다.: `build_posix_sitl_default/src/firmware/posix/rootfs/`

## 하늘로 날리기(Taking it to the Sky)

[jMAVSim](http://github.com/PX4/jMAVSim.git) 시뮬레이터의 3D 뷰를 가지는 윈도우:

![](../../assets/sim/jmavsim.png)

일단 초기화가 끝나면 시스템은 home position을 출력합니다.(`telem> home: 55.7533950, 37.6254270, -0.00`) 다음과 같이 입력하면 날릴 수 있습니다.:

```sh
pxh> commander takeoff
```

> **Info** 조이스틱 지원은 QGroundControl(QGC)를 통해서 가능합니다. 수동 입력을 사용하기 위해서는 메뉴얼 비행 모드로(POSCTL, position control) 시스템을 설정합니다. QGC preferences 메뉴에서 조이스틱을 활성화시킬 수 있습니다.

## Wifi 드론 시뮬레이팅

로컬 네트워크에서 Wifi를 통해 연결한 드론을 시뮬레이션하기 위한 특별한 타겟은 다음과 같습니다.:

```sh
make broadcast jmavsim
```

시뮬레이터는 실제 드론이 하는 것처럼 로컬 네트워크에 자신의 주소를 브로드캐스팅합니다.

## 확장과 커스터마이징

시뮬레이션 인터페이스를 확장하고 커스터마이징하기 위해서 **Tools/jMAVSim** 폴더에 있는 파일을 수정합니다. 코드는 Github의 [jMAVSim repository](https://github.com/px4/jMAVSim)를 참고하세요.

> **Info** 빌드 시스템은 시뮬레이터를 포함해서 의존하는 모든 모듈의 올바른 버전인지를 확인해야 합니다. 디렉토리에 있는 파일의 변경을 덮어쓰기를 하지는 않습니다. 하지만 변경 사항이 서브모듈에 커밋되는 경우 새로운 commit hash로 Firmware repo에 등록되어야 합니다. 이렇게 하기 위해서 `git add Tools/jMAVSim`와 변경을 커밋합니다. 이것은 시뮬레이터의 GIT hash를 업데이트합니다.

## ROS에 인터페이스
시뮬레이션은 실제 비행체의 온보드와 동일한 방식으로 [ROS에 대한 인터페이스](../simulation/ros_interface.md)가 될 수 있습니다.
