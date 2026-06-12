# troubleshooting.md

# 협동3 AMR Fleet 실행 오류 해결 정리

## 1. v43 bridge 파일 없음

### 증상

```text
/usr/bin/python3: can't open file
fleet_manager_bridge_node_gpu_v43_guarded_actions.py
No such file or directory
```

### 원인

`run_bridge_gpu.sh`는 v43 파일을 실행하도록 되어 있지만, 해당 파일이 현재 폴더에 없다.

### 해결

```bash
cd ~/isaaclab_ws/isaac_aruco/amr

cp ~/Downloads/fleet_manager_bridge_node_gpu_v43_guarded_actions.py \
   ./fleet_manager_bridge_node_gpu_v43_guarded_actions.py

python3 -m py_compile fleet_manager_bridge_node_gpu_v43_guarded_actions.py && echo "V43 BRIDGE OK"
```

---

## 2. thunderbolt0 인터페이스 오류

### 증상

```text
thunderbolt0: does not match an available interface
```

### 원인

CycloneDDS XML에 `thunderbolt0`가 지정되어 있는데 현재 Wi-Fi로 실행 중이다.

### 해결

`~/.ros/cyclonedds_wifi.xml`을 만들고 Wi-Fi interface를 지정한다.

```bash
ip -br addr
```

Wi-Fi 이름이 `wlp128s20f3`이면:

```xml
<NetworkInterface name="wlp128s20f3"/>
```

---

## 3. CycloneDDS XML parser error

### 증상

```text
parser error open/close tag mismatch
```

### 원인

CycloneDDS XML 태그가 깨졌다.

### 해결

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

---

## 4. Action Server가 상대 PC에서 안 보임

### 확인

양쪽 PC에서:

```bash
echo $ROS_DOMAIN_ID
echo $RMW_IMPLEMENTATION
echo $CYCLONEDDS_URI
ip -br addr
```

조건:

```text
ROS_DOMAIN_ID=119
RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
같은 Wi-Fi 대역
Peer address에 서로의 IP 포함
```

### 확인 명령

Bridge PC:

```bash
ros2 action list | grep manage_workstation
```

상대 PC:

```bash
ros2 action list | grep manage_workstation
```

상대 PC에도 `/manage_workstation`과 `/amr_01/manage_workstation` 등이 보여야 한다.

---

## 5. Participant index 오류

### 증상

```text
Failed to find a free participant index
```

### 해결

CycloneDDS XML에 다음 값 추가:

```xml
<ParticipantIndex>auto</ParticipantIndex>
<MaxAutoParticipantIndex>300</MaxAutoParticipantIndex>
```

---

## 6. AMR이 멈췄는데 no_path=0

### 의미

```text
no_path=0:
경로는 존재

wait 증가:
Global Arbiter가 현재 tick 이동을 보류
```

즉 A* 실패가 아니라 충돌 회피 판단이다.

---

## 7. wait/no_path가 계속 증가

### 원인 후보

```text
1. target cell 자체가 이미 점유됨
2. 같은 target_location에 중복 명령이 들어옴
3. LOCAL_ENTRY route의 다음 cell이 static workstation으로 막힘
4. 작업대 footprint가 corridor에서 충돌
5. 이전 command JSON이 queue에 남아 있음
```

### 우선 확인

```bash
ls -l bridge_queue/commands
ls -l bridge_queue/status
ls -l bridge_queue/results
```

queue 초기화:

```bash
rm -f bridge_queue/commands/*.json \
      bridge_queue/status/*.json \
      bridge_queue/results/*.json \
      bridge_queue/cancel/*.json \
      bridge_queue/done/*.json 2>/dev/null
```

---

## 8. 중복 target 문제

### 실제 사례

```text
CMD_fa1baa6f3e0b:
WS05 → sg2_in_03_B

CMD_3bdcdced7901:
WS06 → sg2_in_03_B
```

결과:

```text
AMR_04가 WS05를 먼저 target=(4,-5)에 배치
AMR_03은 WS06을 들고 (3,-5)에서 정지
wait/no_path 증가
```

### 해결

Bridge v43 admission guard 사용:

```bash
export AMR_BRIDGE_ADMISSION_GUARD=1
```

기대 로그:

```text
ManageWorkstation rejected by admission guard
reason=DUPLICATE_TARGET
```

---

## 9. LOCAL_ENTRY static blocker 문제

### 실제 사례

```text
AMR_02:
state=LOCAL_ENTRY
cell=(-1,-1)
target=(-1,0)
carry=WS05

문제:
(-1,0)에 WS07 존재
```

### 의미

LOCAL_ENTRY route의 다음 cell이 static workstation으로 막혀 진행 불가.

### 해결 방향

```text
future route static blocker 검사
LOCAL_MACRO ROUTE REJECT
대체 route 선택
```

---

## 10. 최종 정상 실행 체크리스트

```bash
cd ~/isaaclab_ws/isaac_aruco/amr

# 파일 확인
ls -l fleet_manager_bridge_node_gpu_v43_guarded_actions.py
ls -l amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py

# bridge 문법 확인
python3 -m py_compile fleet_manager_bridge_node_gpu_v43_guarded_actions.py && echo "V43 BRIDGE OK"

# 네트워크 설정 확인
grep -n "NetworkInterface\|Peer\|ParticipantIndex" ~/.ros/cyclonedds_wifi.xml

# bridge 실행
./run_bridge_gpu.sh

# action 확인
ros2 action list | grep manage_workstation
```
