# requirements.md

# 협동3 AMR Fleet / IsaacSim 실행 의존성 정리

## 1. 프로젝트 기준

본 문서는 협동3 AMR Fleet 시뮬레이션을 실행하기 위한 의존성, 환경 변수, 네트워크 설정, 실행 전 점검 항목을 정리한 문서이다.

최종 실행 기준 파일은 다음 2개이다.

```text
AMR Controller:
amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py

Bridge:
fleet_manager_bridge_node_gpu_v43_guarded_actions.py
```

실제 프로젝트 폴더 기준 위치:

```text
~/isaaclab_ws/isaac_aruco/amr/
```

---

## 2. 운영체제 / 기본 환경

| 항목 | 기준 |
|---|---|
| OS | Ubuntu 22.04 |
| ROS | ROS2 Humble |
| Python | Python 3.10 |
| DDS/RMW | CycloneDDS / rmw_cyclonedds_cpp |
| ROS_DOMAIN_ID | 119 |
| IsaacSim | IsaacSim 실행 환경 필요 |
| GPU | NVIDIA GPU 권장 |
| 네트워크 | Wi-Fi 기반 ROS2 통신 |

---

## 3. ROS2 의존성

필수 ROS2 패키지:

```bash
sudo apt update
sudo apt install -y \
  ros-humble-desktop \
  ros-humble-rmw-cyclonedds-cpp \
  python3-colcon-common-extensions
```

필수 커스텀 인터페이스:

```text
cobot3_interfaces.action.ManageWorkstation
```

선택 인터페이스:

```text
cobot3_interfaces.action.MovePackage
```

Bridge 파일은 다음 Action을 사용한다.

```python
from cobot3_interfaces.action import ManageWorkstation
```

따라서 실행 전에 `cobot3_interfaces`가 ROS2 워크스페이스에 빌드되어 있어야 한다.

확인:

```bash
ros2 interface show cobot3_interfaces/action/ManageWorkstation
```

정상이라면 action 필드가 출력된다.

---

## 4. Python 의존성

### 4.1 Bridge 쪽

Bridge는 대부분 Python 표준 라이브러리와 ROS2 Python API를 사용한다.

필수:

```text
rclpy
cobot3_interfaces
```

사용 모듈:

```text
json
os
time
uuid
threading
pathlib
typing
rclpy
```

### 4.2 IsaacSim Controller 쪽

Controller는 IsaacSim 내부 Python 환경에서 실행된다.

필수 모듈:

```text
numpy
opencv-python 또는 IsaacSim 내 cv2
pxr
omni.usd
omni.timeline
omni.kit.app
omni.replicator.core
```

Controller import 기준:

```python
import cv2
import numpy
import omni.usd
import omni.timeline
import omni.kit.app
import omni.replicator.core
from pxr import ...
```

Ubuntu 일반 Python이 아니라 IsaacSim Script Editor 또는 IsaacSim Python 환경에서 실행해야 한다.

---

## 5. 폴더 구조 의존성

Bridge와 Controller는 파일 기반 queue를 공유한다.

필수 경로:

```text
/home/rokey/isaaclab_ws/isaac_aruco/amr/bridge_queue/
├── commands/
├── status/
├── results/
├── cancel/
└── done/
```

없으면 생성:

```bash
mkdir -p ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/{commands,status,results,cancel,done}
```

---

## 6. 최종 파일 배치

프로젝트 폴더:

```bash
cd ~/isaaclab_ws/isaac_aruco/amr
```

최종 파일 배치:

```bash
cp ~/Downloads/amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py \
   ~/isaaclab_ws/isaac_aruco/amr/

cp ~/Downloads/fleet_manager_bridge_node_gpu_v43_guarded_actions.py \
   ~/isaaclab_ws/isaac_aruco/amr/
```

문법 검사:

```bash
cd ~/isaaclab_ws/isaac_aruco/amr

python3 -m py_compile fleet_manager_bridge_node_gpu_v43_guarded_actions.py && echo "BRIDGE OK"
```

Controller는 IsaacSim 모듈을 사용하므로 일반 Python에서 완전 실행 검사는 불가능할 수 있다. 단순 문법 검사만 필요한 경우:

```bash
python3 -m py_compile amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py
```

---

## 7. ROS2 네트워크 설정

현재 프로젝트는 Wi-Fi 기반으로 ROS2 Action 통신을 한다.

기준 값:

```bash
export ROS_DOMAIN_ID=119
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
```

Wi-Fi 인터페이스 확인:

```bash
ip -br addr
```

사용자 PC의 Wi-Fi 인터페이스 예시:

```text
wlp128s20f3
```

상대방 IP가 `192.168.10.19`인 경우 CycloneDDS 설정 예시:

```bash
mkdir -p ~/.ros

cat > ~/.ros/cyclonedds_wifi.xml <<'XML'
<CycloneDDS>
  <Domain id="any">
    <General>
      <Interfaces>
        <NetworkInterface name="wlp128s20f3"/>
      </Interfaces>
      <AllowMulticast>true</AllowMulticast>
    </General>
    <Discovery>
      <ParticipantIndex>auto</ParticipantIndex>
      <MaxAutoParticipantIndex>300</MaxAutoParticipantIndex>
      <Peers>
        <Peer address="192.168.10.40"/>
        <Peer address="192.168.10.19"/>
      </Peers>
    </Discovery>
  </Domain>
</CycloneDDS>
XML
```

적용:

```bash
export CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml
```

주의:

```text
- NetworkInterface 이름은 실제 Wi-Fi 인터페이스명과 같아야 한다.
- 상대방 IP가 바뀌면 Peer address도 바꿔야 한다.
- Thunderbolt를 쓰지 않는 경우 thunderbolt0가 XML에 들어가면 안 된다.
```

---

## 8. run_bridge_gpu.sh 필수 설정

`run_bridge_gpu.sh` 안에 다음 값이 들어가야 한다.

```bash
export ROS_DOMAIN_ID=119
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml

exec /usr/bin/python3 fleet_manager_bridge_node_gpu_v43_guarded_actions.py
```

확인:

```bash
cd ~/isaaclab_ws/isaac_aruco/amr
grep -n "ROS_DOMAIN_ID\|RMW_IMPLEMENTATION\|CYCLONEDDS_URI\|python3\|fleet_manager" run_bridge_gpu.sh
```

---

## 9. 주요 환경 변수

### Bridge

| 환경 변수 | 기본값 | 설명 |
|---|---:|---|
| `AMR_BRIDGE_EXECUTOR_THREADS` | `2` | Bridge MultiThreadedExecutor thread 수 |
| `AMR_BRIDGE_ADMISSION_GUARD` | `1` | 중복 명령 차단 기능 활성화 |

Admission Guard 비활성화 테스트:

```bash
export AMR_BRIDGE_ADMISSION_GUARD=0
```

일반 실행에서는 비활성화하지 않는다.

### Controller

| 환경 변수 | 기본값 | 설명 |
|---|---:|---|
| `AMR_GPU_ENABLED` | `1` | GPU 사용 |
| `AMR_QR_GPU_PREPROCESS_ENABLED` | `1` | QR 전처리 GPU 사용 |
| `AMR_QR_CUDA_DEVICE_ID` | `0` | CUDA device |
| `AMR_OPENCV_CPU_THREADS` | `1` | OpenCV CPU thread 수 |
| `AMR_RESERVATION_HORIZON` | `35` | Reservation Table 시간 범위 |
| `AMR_COST_AWARE_GLOBAL_REROUTE_ENABLED` | `1` | Cost-aware reroute 활성화 |
| `AMR_COST_AWARE_REROUTE_MIN_WAIT_STEPS` | `3` | reroute 판단 시작 wait 기준 |
| `AMR_COST_AWARE_REROUTE_FORCE_AFTER_WAIT_STEPS` | `18` | 강제 reroute 검토 wait 기준 |
| `AMR_COST_AWARE_REROUTE_MAX_EXTRA_CELLS_EMPTY` | `5.0` | 빈 AMR 우회 허용 추가 거리 |
| `AMR_COST_AWARE_REROUTE_MAX_EXTRA_CELLS_CARRY` | `3.0` | 운반 AMR 우회 허용 추가 거리 |
| `AMR_SG_LOCAL_MACRO_ROUTE_ENABLED` | `1` | SG local macro route 활성화 |
| `AMR_SG_LOCAL_MACRO_LOG_ENABLED` | `1` | SG local macro 로그 출력 |

권장 실행값:

```bash
export AMR_COST_AWARE_GLOBAL_REROUTE_ENABLED=1
export AMR_COST_AWARE_REROUTE_MIN_WAIT_STEPS=3
export AMR_COST_AWARE_REROUTE_FORCE_AFTER_WAIT_STEPS=18
export AMR_SG_LOCAL_MACRO_ROUTE_ENABLED=1
export AMR_SG_LOCAL_MACRO_LOG_ENABLED=1
```

---

## 10. Redis 상태 발행

Controller에는 Redis 상태 발행 코드가 포함되어 있다.

기본 설정:

```text
REDIS_STATUS_ENABLED = True
REDIS_HOST = 192.168.100.20
REDIS_PORT = 6379
REDIS_KEY_PREFIX = amr:
```

현재 Wi-Fi 통신만 사용할 경우 Redis PC IP가 다르면 코드 또는 환경 설정을 수정해야 한다.

Redis가 꺼져 있어도 Controller는 실행되지만, 상태 발행 로그에 연결 대기 메시지가 나올 수 있다.

---

## 11. 실행 전 점검 명령

```bash
cd ~/isaaclab_ws/isaac_aruco/amr

# 파일 존재 확인
ls -l amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py
ls -l fleet_manager_bridge_node_gpu_v43_guarded_actions.py
ls -l run_bridge_gpu.sh

# Bridge 문법 확인
python3 -m py_compile fleet_manager_bridge_node_gpu_v43_guarded_actions.py && echo "BRIDGE OK"

# ROS 환경 확인
echo $ROS_DOMAIN_ID
echo $RMW_IMPLEMENTATION
echo $CYCLONEDDS_URI

# Wi-Fi 인터페이스 확인
ip -br addr

# XML 확인
grep -n "NetworkInterface\|Peer\|AllowMulticast\|ParticipantIndex" ~/.ros/cyclonedds_wifi.xml
```

---

## 12. 흔한 오류

### 12.1 thunderbolt0 에러

```text
thunderbolt0: does not match an available interface
```

원인:

```text
CycloneDDS XML이 thunderbolt0를 지정하고 있는데 현재 Wi-Fi로 실행 중임.
```

해결:

```text
NetworkInterface name을 Wi-Fi 인터페이스명으로 변경.
예: wlp128s20f3
```

### 12.2 XML parser error

```text
parser error open/close tag mismatch
```

원인:

```text
CycloneDDS XML 태그가 깨짐.
```

해결:

```bash
cat > ~/.ros/cyclonedds_wifi.xml <<'XML'
<CycloneDDS>
  <Domain id="any">
    <General>
      <Interfaces>
        <NetworkInterface name="wlp128s20f3"/>
      </Interfaces>
      <AllowMulticast>true</AllowMulticast>
    </General>
    <Discovery>
      <ParticipantIndex>auto</ParticipantIndex>
      <MaxAutoParticipantIndex>300</MaxAutoParticipantIndex>
      <Peers>
        <Peer address="192.168.10.40"/>
        <Peer address="192.168.10.19"/>
      </Peers>
    </Discovery>
  </Domain>
</CycloneDDS>
XML
```

### 12.3 v43 파일 없음

```text
can't open file fleet_manager_bridge_node_gpu_v43_guarded_actions.py
```

해결:

```bash
cp ~/Downloads/fleet_manager_bridge_node_gpu_v43_guarded_actions.py \
   ~/isaaclab_ws/isaac_aruco/amr/
```

---

## 13. 최종 실행 성공 기준

Bridge 실행 후 다른 터미널에서 확인:

```bash
ros2 action list | grep manage_workstation
```

정상 출력 예시:

```text
/amr_01/manage_workstation
/amr_02/manage_workstation
/amr_03/manage_workstation
/amr_04/manage_workstation
/amr_05/manage_workstation
/manage_workstation
```
