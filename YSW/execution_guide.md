# execution_guide.md

# 협동3 AMR Fleet 실행 순서

## 1. 실행 개요

최종 시스템은 크게 두 부분으로 실행된다.

```text
1. IsaacSim Controller
   - AMR 5대와 작업대 움직임 제어
   - Time A*, Reservation Table, Global Arbiter, Cost-aware Reroute 수행
   - bridge_queue/commands 폴더의 명령 JSON을 읽음

2. ROS2 Bridge
   - ROS2 Action Server 제공
   - 외부 PC 또는 Control Tower에서 들어오는 ManageWorkstation goal 수신
   - command JSON을 bridge_queue/commands에 저장
   - 중복 명령을 admission guard로 사전 차단
```

---

## 2. 최종 파일 구성

프로젝트 폴더:

```bash
cd ~/isaaclab_ws/isaac_aruco/amr
```

필수 파일:

```text
amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py
fleet_manager_bridge_node_gpu_v43_guarded_actions.py
run_bridge_gpu.sh
```

Bridge queue 폴더:

```bash
mkdir -p bridge_queue/{commands,status,results,cancel,done}
```

---

## 3. Wi-Fi ROS2 설정

### 3.1 현재 IP 확인

```bash
ip -br addr
```

예시:

```text
내 PC: 192.168.10.40
상대 PC: 192.168.10.19
Wi-Fi interface: wlp128s20f3
```

### 3.2 CycloneDDS Wi-Fi XML 생성

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

### 3.3 환경 변수 적용

```bash
export ROS_DOMAIN_ID=119
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml
```

---

## 4. run_bridge_gpu.sh 설정

`run_bridge_gpu.sh`가 최종 bridge 파일을 실행하도록 맞춘다.

```bash
cd ~/isaaclab_ws/isaac_aruco/amr

python3 - <<'PY'
from pathlib import Path
import re

p = Path("run_bridge_gpu.sh")
s = p.read_text()

# ROS_DOMAIN_ID
if "ROS_DOMAIN_ID" in s:
    s = re.sub(r'^\s*export\s+ROS_DOMAIN_ID=.*$', 'export ROS_DOMAIN_ID=119', s, flags=re.M)
else:
    s = 'export ROS_DOMAIN_ID=119\n' + s

# RMW
if "RMW_IMPLEMENTATION" in s:
    s = re.sub(r'^\s*export\s+RMW_IMPLEMENTATION=.*$', 'export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp', s, flags=re.M)
else:
    s = 'export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp\n' + s

# CYCLONEDDS_URI
if "CYCLONEDDS_URI" in s:
    s = re.sub(
        r'^\s*#?\s*export\s+CYCLONEDDS_URI=.*$',
        'export CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml',
        s,
        flags=re.M
    )
else:
    s = 'export CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml\n' + s

# 실행 파일
s = s.replace("fleet_manager_bridge_node_gpu.py", "fleet_manager_bridge_node_gpu_v43_guarded_actions.py")
s = s.replace("fleet_manager_bridge_node_gpu_v42_per_amr_actions.py", "fleet_manager_bridge_node_gpu_v43_guarded_actions.py")

p.write_text(s)
print("patched:", p)
PY

grep -n "ROS_DOMAIN_ID\|RMW_IMPLEMENTATION\|CYCLONEDDS_URI\|python3\|fleet_manager" run_bridge_gpu.sh
```

---

## 5. 실행 전 queue 초기화

이전 테스트 명령이 남아 있으면 잘못된 동작이 재현될 수 있다.

```bash
cd ~/isaaclab_ws/isaac_aruco/amr

rm -f bridge_queue/commands/*.json \
      bridge_queue/status/*.json \
      bridge_queue/results/*.json \
      bridge_queue/cancel/*.json \
      bridge_queue/done/*.json 2>/dev/null
```

---

## 6. IsaacSim Controller 실행

IsaacSim을 켠 뒤 Script Editor에서 실행한다.

```python
exec(open('/home/rokey/isaaclab_ws/isaac_aruco/amr/amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py', encoding='utf-8').read())
```

또는 실제 실행 파일명을 기존 이름으로 맞춘 경우:

```python
exec(open('/home/rokey/isaaclab_ws/isaac_aruco/amr/amr_live_existing_stage_true8_qr_camera_controller_gpu.py', encoding='utf-8').read())
```

권장: 최종 버전 파일명을 그대로 실행하면 제출 파일과 추적이 쉽다.

---

## 7. Bridge 실행

새 터미널에서 실행:

```bash
cd ~/isaaclab_ws/isaac_aruco/amr

export ROS_DOMAIN_ID=119
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml

./run_bridge_gpu.sh
```

정상 로그 예시:

```text
AMR Fleet Bridge V42 ActionServer ready: /manage_workstation
AMR Fleet Bridge V42 ActionServer ready: /amr_01/manage_workstation -> preferred AMR_01
...
Bridge queue dir: /home/rokey/isaaclab_ws/isaac_aruco/amr/bridge_queue
Admission guard enabled: True
```

---

## 8. Action Server 확인

다른 터미널:

```bash
export ROS_DOMAIN_ID=119
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml

ros2 action list | grep manage_workstation
```

정상 출력:

```text
/amr_01/manage_workstation
/amr_02/manage_workstation
/amr_03/manage_workstation
/amr_04/manage_workstation
/amr_05/manage_workstation
/manage_workstation
```

---

## 9. 상대 PC에서 확인

상대방 PC도 같은 조건이어야 한다.

```bash
export ROS_DOMAIN_ID=119
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml
```

상대방 XML에는 자신의 IP와 내 PC IP가 모두 들어가야 한다.

예시:

```xml
<Peers>
  <Peer address="192.168.10.19"/>
  <Peer address="192.168.10.40"/>
</Peers>
```

상대방 확인:

```bash
ros2 action list | grep manage_workstation
```

---

## 10. 주요 기능 실행 순서

### 10.1 정상 작업대 이동

흐름:

```text
외부 PC / Control Tower
→ ROS2 ManageWorkstation Action goal
→ Bridge v43 admission guard 검사
→ command JSON 생성
→ Controller v42 command scan
→ AMR 배정
→ Time A* 경로계획
→ Global Arbiter 승인
→ AMR 이동
→ 작업대 pickup
→ SG target 이동
→ placement
→ result JSON 작성
→ Bridge action result 반환
```

### 10.2 중복 명령 차단

중복 target 예시:

```text
CMD_1: WS05 → sg2_in_03_B
CMD_2: WS06 → sg2_in_03_B
```

Bridge v43 동작:

```text
CMD_1 ACCEPT
CMD_2 REJECT: DUPLICATE_TARGET
```

중복 workstation 예시:

```text
CMD_1: WS05 → sg2_in_03_B
CMD_2: WS05 → sg2_in_02_B
```

Bridge v43 동작:

```text
CMD_1 ACCEPT
CMD_2 REJECT: DUPLICATE_WORKSTATION
```

### 10.3 Global Arbiter 충돌 회피

```text
각 AMR이 다음 이동 후보 제출
→ Global Arbiter가 cell 충돌, edge swap, 작업대 footprint 충돌 검사
→ 안전한 이동만 승인
→ 위험하면 WAIT
```

### 10.4 Cost-aware Reroute

```text
Global Arbiter가 이동 reject
→ 해당 cell을 임시 blocked cell로 처리
→ detour A* 재계산
→ wait_cost와 detour_cost 비교
→ WAIT 또는 REROUTE 선택
```

### 10.5 Local Macro Route

```text
SG 진입부 접근
→ 일반 A* 대신 deterministic local macro route 사용
→ future route static blocker 검사
→ cell-by-cell 진입
→ placement
```

---

## 11. 로그 확인 포인트

### Bridge

```text
ManageWorkstation received
ManageWorkstation rejected by admission guard
DUPLICATE_TARGET
DUPLICATE_WORKSTATION
ManageWorkstation completed
ManageWorkstation timeout
```

### Controller

```text
COMMAND ACCEPTED
LOCAL_MACRO ENTRY START
LOCAL_MACRO ENTRY STEP
LOCAL_MACRO ROUTE REJECT
COST_DECISION
decision=WAIT
decision=REROUTE
DELIVERY_DONE
```

---

## 12. 종료

Bridge 종료:

```bash
Ctrl + C
```

남은 프로세스 정리:

```bash
pkill -f fleet_manager_bridge_node_gpu
pkill -f run_bridge_gpu.sh
```

queue 초기화:

```bash
rm -f ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/commands/*.json \
      ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/status/*.json \
      ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/results/*.json \
      ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/cancel/*.json \
      ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/done/*.json 2>/dev/null
```
