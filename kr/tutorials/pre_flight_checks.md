# 비행전 센서와 EKF 체크
commander 모듈이 여러 가지 비행전 센서 품질과 EKF 체크를 수행합니다. COM_ARM<> 파라미터로 제어합니다. 이 체크에서 실패하면 모터는 arming을 하지 않고 다음과 같은 에러가 내게 됩니다:

* PREFLIGHT FAIL: EKF HGT ERROR
 * IMU와 높이 측정 데이터가 일치하는 않는 경우에 발생하는 에러입니다.
 * accel과 gyro 칼리브레이션을 수행하고 재시작 시킵니다. 만약 에러가 지속되면 고도 센서 데이터에 문제가 없는지 확인하세요.
 * 이 검사는 COM_ARM_EKF_HGT 파라미터로 제어됩니다.
* PREFLIGHT FAIL: EKF VEL ERROR
 * IMU와 GPS 속도 측정 데이터가 일치하지 않는 경우에 발생하는 에러입니다.
 * 비현실적인 데이터로 점프하는 경우 GPS 속도 데이터를 확인하세요.GPS 품질이 괜찮다면 accel과 gyro 칼리브레이션을 수행하고 재시작 시킵니다.
 * 이 검사는 COM_ARM_EKF_VEL 파라미터로 제어됩니다.
* PREFLIGHT FAIL: EKF HORIZ POS ERROR
 * IMU와 position 측정 데이터가(GPS나 외부 비젼) 일치하는 않는 경우에 발생합니다.
 * 비현실적인 데이터로 점프하는 경우 position 센서 데이터를 확인하세요. 만약 데이터 품질이 괜찮아보인다면 accel과 gyro 칼리브레이션을 수행하고 재시작 시킵니다.
 * 이 검사는 COM_ARM_EKF_POS 파라미터로 제어됩니다.
* PREFLIGHT FAIL: EKF YAW ERROR
 * gyro 데이터로 추측한 yaw 각도와 magnetometer나 외부 비젼 시스템으로 예측한 yaw 각이 일치하지 않는 경우에 발생합니다.
 * yaw rate offset이 큰 경우 IMU 데이터를 체크하고 magnetometer 정렬과 칼리브레이션을 검사합니다.
 * 이 검사는 COM_ARM_EKF_POS 파라미터로 제어됩니다.
* PREFLIGHT FAIL: EKF HIGH IMU ACCEL BIAS
 * EKF로 예측한 IMU accelerometer bias가 과도한 경우에 발생하는 에러입니다.
 * 이 검사는 COM_ARM_EKF_AB 파라미터로 제어됩니다.
* PREFLIGHT FAIL: EKF HIGH IMU GYRO BIAS
 * EKF로 예측한 IMU gyro bias가 과돠한 경우에 발생하는 에러입니다.
 * 이 검사는 COM_ARM_EKF_GB 파라미터로 제어됩니다.
* PREFLIGHT FAIL: ACCEL SENSORS INCONSISTENT - CHECK CALIBRATION
 * 서로 다른 IMU 유닛에서 acceleration 측정 값이 일치하지 않는 경우에 발생합니다.
 * 이 검사는 보드에 2개 이상의 IMU가 장착된 경우에 시행합니다.
 * 이 검사는 COM_ARM_IMU_ACC 파라미터로 제어됩니다.
* PREFLIGHT FAIL: GYRO SENSORS INCONSISTENT - CHECK CALIBRATION
 * 서로 다른 IMU 유닛에서 angular rate 측정값이 일치하지 않는 경우에 발생합니다.
 * 이 검사는 보드에 2개 이상의 IMU가 장착된 경우에 시행합니다.
 * 이 검사는 COM_ARM_IMU_GYR 파라미터로 제어됩니다.

##COM_ARM_WO_GPS
COM_ARM_WO_GPS 파라미터는 GPS 신호 없이도 arming을 허용할지 여부를 결정합니다. GPS 신호 없이 arming이 되도록 설정하기 위해선느 반드시 0으로 설정되어 있어야만 합니다. GPS 신호 없이 arming은 선택한 flight mode가 GPS가 필요하지 않은 경우에만 허용됩니다.
##COM_ARM_EKF_POS
COM_ARM_EKF_POS 파라미터는 EKF inertial 측정과 position 레퍼런스(GPS 혹은 외부 비젼)가 일치하지 않는 경우를 허용하는 최대치를 제어하는 것입니다. 기본 값 0.5는 EKF가 허용하는 최대값의 50% 이하의 차이를 허용하며 비행을 시작할때 에러 증가에 대한 일정 부분 마진을 제공합니다.
##COM_ARM_EKF_VEL
COM_ARM_EKF_VEL 파라미터는 EKF inertial 측정과 GPS 속도 측정 사이에 일치하지 않는 경우를 허용하는 최대치를 제어하는 것입니다. 기본 값 0.5는 EKF가 허용하는 최대값의 50% 이하의 차이를 허용하며 비행을 시작할때 에러 증가에 대한 일정 부분 마진을 제공합니다.
##COM_ARM_EKF_HGT
COM_ARM_EKF_HGT 파라미터는 EKF inertial 측정과 고도 측정(Baro, GPS, range finder나 외부 비젼) 사이에 일치하지 않는 경우를 허용하는 최대치를 제어하는 것입니다. 기본 값 0.5는 EKF가 허용하는 최대값의 50% 이하의 차이를 허용하며 비행을 시작할때 에러 증가에 대한 일정 부분 마진을 제공합니다.
##COM_ARM_EKF_YAW
COM_ARM_EKF_YAW 파라미터는 EKF inertial 측정과 yaw 측정(magnetometer나 외부 비젼) 사이에 일치하지 않는 경우를 허용하는 최대치를 제어하는 것입니다. 기본 값 0.5는 EKF가 허용하는 최대값의 50% 이하의 차이를 허용하며 비행을 시작할때 에러 증가에 대한 일정 부분 마진을 제공합니다.
##COM_ARM_EKF_AB
COM_ARM_EKF_AB 파라미터는 최대로 허용하는 EKF 예측 IMU accelerometer bias를 제어합니다. 기본 값 0.005는 accelerometer bias의 0.5 m/s/s 까지 허용합니다.
##COM_ARM_EKF_GB
COM_ARM_EKF_GB 파라미터는 최대로 허용하는 EKF 예측 IMU gyro bias를 제어합니다. 기본 값 0.00087는 gyro bias에 스위치의 5 deg/sec까지 허용합니다.
##COM_ARM_IMU_ACC
COM_ARM_IMU_ACC 파라미터는 비행 제어에 사용한 기본 IMU와 다른 IMU 유닛이 설치되어 있는 경우 이것들 사이에 acceleration 측정에서 일치하지 않는 경우를 허용하는 최대치를 제어합니다.
##COM_ARM_IMU_GYR
COM_ARM_IMU_GYR 파라미터는 비행 제어에 사용한 기본 IMU와 다른 IMU 유닛이 설치되어 있는 경우 이것들 사이에 angular rate 측정에서 일치하지 않는 경우를 허용하는 최대치를 제어합니다.
