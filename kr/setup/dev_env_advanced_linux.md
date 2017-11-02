# 고급 Linux

이 주제는 고급 설치와 관련된 내용을 다룹니다.

## USB Device 설정

Linux 사용자는 명시적으로 JTAG 프로그래밍 아답터용 USB bus에 접근이 허용됩니다.

> **Note** Archlinux의 경우: 아래 명령에서 plugdev 그룹을 uucp로 변경


단순히 `sudo` 모드에서 `ls`를 실행하여 아래와 같이 명령이 동작하는지 확인:

```sh
sudo ls
```

`sudo` 권한을 가진 경우 다음 명령을 실행:

```sh
cat > $HOME/rule.tmp <<_EOF
# All 3D Robotics (includes PX4) devices
SUBSYSTEM=="usb", ATTR{idVendor}=="26AC", GROUP="plugdev"
# FTDI (and Black Magic Probe) Devices
SUBSYSTEM=="usb", ATTR{idVendor}=="0483", GROUP="plugdev"
# Olimex Devices
SUBSYSTEM=="usb",  ATTR{idVendor}=="15ba", GROUP="plugdev"
_EOF
sudo mv $HOME/rule.tmp /etc/udev/rules.d/10-px4.rules
sudo /etc/init.d/udev restart
```

해당 사용자를 **plugdev** 그룹에 추가:

```sh
sudo usermod -a -G plugdev $USER
```
