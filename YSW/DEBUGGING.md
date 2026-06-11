# DEBUGGING.md

# 1. 협동3 디버깅 및 구현 정리

> 담당자: 윤성웅
> 역할: 팀장 / 프로젝트 기획 / 전체 시나리오 설계 / 팀원 조율 / GitHub 문서 작업 / PPT 제작 / 발표 / ROS2·IsaacSim 연동 디버깅 / AMR Fleet 주행 안정화 / 네트워크 통신 문제 해결
> 기간: 2026년 5월 29일 ~ 2026년 6월 12일
> 발표일: 2026년 6월 12일
> 대상 프로젝트: 협동3 IsaacSim 기반 AMR Fleet 물류 자동화 시뮬레이션


최종 문서 구조는 다음과 같다.

```text
1. 전체 개요
2. 시스템 구조
3. ROS2 Bridge 상세 구조
4. bridge_queue 파일 연동 방식
5. Global Arbiter 상세 구조
6. 디버깅 원칙
7. 날짜별 진행 타임라인
8. 계층별 디버깅
9. 주요 이슈별 상세 디버깅 카드
10. 적용한 패치 정리
11. 검증 방법
12. 최종 결과
13. 성과 및 기여
```

---

# 2. 프로젝트 전체 개요

협동3 프로젝트는 IsaacSim 환경에서 AMR 5대가 작업대를 픽업하고, 지정된 SG 또는 stage 구역으로 운반한 뒤, 작업 완료 후 복귀하는 물류 자동화 시뮬레이션이다.

단순히 AMR을 하나씩 움직이는 데모가 아니라, 외부 PC 또는 Control Tower에서 ROS2 Action 명령을 내리고, bridge가 해당 명령을 JSON command로 변환하며, IsaacSim controller가 이를 읽어 실제 stage 안의 AMR과 작업대를 제어하는 구조로 구성하였다.

전체 흐름은 다음과 같다.

```text
상대 PC 또는 Control Tower
→ ROS2 Action Goal 전송
→ Fleet Manager Bridge 수신
→ bridge_queue/commands/CMD_xxx.json 생성
→ IsaacSim Controller가 command JSON 읽음
→ AMR 작업 수행
→ status/result JSON 작성
→ Bridge가 ROS2 feedback/result 반환
```

---

# 3. 성웅 역할

성웅은 협동3 프로젝트에서 팀장 역할을 맡아 다음 작업을 담당하였다.

```text
1. 프로젝트 전체 주제 기획
2. IsaacSim 기반 AMR Fleet 시나리오 설계
3. 실제 창고 물류 흐름을 가정한 작업 시퀀스 구성
4. 팀원별 구현 방향 조율
5. GitHub README 문서 작업
6. 발표용 PPT 제작
7. 전체 시스템 시나리오와 발표 흐름 구성
8. ROS2 bridge 구조 분석 및 디버깅
9. IsaacSim AMR controller 구조 분석 및 디버깅
10. CycloneDDS 통신 문제 해결
11. 중복 명령 및 목적지 충돌 문제 분석
12. bridge admission guard 패치 적용
13. cost-aware global reroute 구조 적용
14. 최종 발표 준비 및 발표
```

이번 프로젝트에서 성웅의 역할은 단순 코드 작성자가 아니라, 전체 시스템 방향을 잡고, 문제 발생 시 원인을 계층별로 분리하고, 팀원들이 구현한 부분이 하나의 시나리오로 연결되도록 조율하는 팀장 역할이었다.

---

# 4. 최종 시스템 구조

## 4.1 전체 구성

```text
[상대 PC / Control Tower]
        |
        | ROS2 Action Goal
        v
[ROS2 Fleet Manager Bridge]
        |
        | JSON Command
        v
[bridge_queue/commands]
        |
        v
[IsaacSim AMR Controller]
        |
        | AMR 주행 / 작업대 운반 / QR 위치 인식
        v
[IsaacSim Stage]
        |
        | status/result JSON
        v
[bridge_queue/status, results]
        |
        v
[ROS2 Action Feedback / Result]
```

---

## 4.2 주요 파일

### Fleet Manager Bridge

```text
fleet_manager_bridge_node_gpu_v42_per_amr_actions.py
fleet_manager_bridge_node_gpu_v43_guarded_actions.py
```

역할:

```text
1. ROS2 Action Server 실행
2. /manage_workstation 제공
3. /amr_01/manage_workstation ~ /amr_05/manage_workstation 제공
4. Action goal을 command JSON으로 변환
5. IsaacSim controller의 status/result JSON을 읽어 feedback/result 반환
6. v43에서 중복 명령 차단 admission guard 추가
```

---

### IsaacSim AMR Controller

```text
amr_live_existing_stage_true8_qr_camera_controller_gpu.py
amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py
```

역할:

```text
1. 기존 IsaacSim stage의 AMR_01~AMR_05 prim 제어
2. RACK_01~RACK_10 또는 WS_01~WS_10 작업대 제어
3. bridge_queue/commands 폴더의 command JSON 읽기
4. target_location을 target_xy, target_cell로 변환
5. AMR 배정
6. 작업대 픽업, 운반, 배치, 복귀 수행
7. QR 기반 위치 인식
8. 8-way Time A* 경로계획
9. Reservation Table 기반 충돌 회피
10. Local Macro Route 기반 SG 진입/탈출
11. cost-aware global reroute 판단
12. 결과를 status/result JSON으로 작성
```

---

## 4.3 bridge_queue 구조

```text
bridge_queue/
├── commands/
├── status/
├── results/
├── cancel/
└── done/
```

### commands

bridge가 생성하는 명령 JSON이 저장된다.

예시:

```json
{
  "command_id": "CMD_f9bb42e729e8",
  "workstation_id": "WS07",
  "start_location": "",
  "target_location": "sg2_in_01_B",
  "target_x": 0.0,
  "target_y": 0.0,
  "target_yaw": 0.0,
  "preferred_amr_name": "AMR_05",
  "preferred_amr": "AMR_05",
  "require_preferred_amr": true,
  "created_at": 1781167628.6863856
}
```

### status

controller가 작업 진행 상태를 기록한다.

```json
{
  "command_id": "CMD_f9bb42e729e8",
  "status": "NAVIGATING",
  "distance_remaining": 4.5,
  "updated_at": 1781167635.4
}
```

### results

작업 성공 또는 실패 결과를 기록한다.

```json
{
  "command_id": "CMD_f9bb42e729e8",
  "success": true,
  "status": "COMPLETED",
  "message": "task completed",
  "finished_at": 1781167655.5
}
```

---

## 4.4 ROS2 Fleet Manager Bridge 상세 구조

Bridge는 협동3 시스템에서 외부 명령 계층과 IsaacSim 제어 계층을 분리하는 역할을 한다. 즉, Bridge 자체가 AMR의 경로를 계산하거나 stage 안의 prim을 직접 움직이는 것이 아니라, ROS2 Action Goal을 controller가 이해할 수 있는 JSON command로 바꾸고, controller가 작성한 status/result를 다시 ROS2 Action feedback/result로 반환한다.

### Bridge의 핵심 책임

```text
1. ROS2 Action Server 실행
2. /manage_workstation 전역 Action 제공
3. /amr_01/manage_workstation ~ /amr_05/manage_workstation per-AMR Action 제공
4. Action Goal 수신 후 command_id 생성
5. command JSON을 bridge_queue/commands에 저장
6. controller가 작성한 status JSON을 읽어 Action feedback 반환
7. controller가 작성한 result JSON을 읽어 Action result 반환
8. cancel 요청을 bridge_queue/cancel에 기록
9. 완료된 command를 done 처리
10. 중복 명령을 admission guard에서 reject
```

### Bridge가 직접 하지 않는 일

```text
1. AMR 경로계획을 직접 수행하지 않음
2. target_location을 최종 주행 경로로 직접 변환하지 않음
3. IsaacSim stage의 AMR prim을 직접 이동시키지 않음
4. 작업대 lifting/placing 물리 동작을 직접 수행하지 않음
5. 다중 AMR 이동 충돌을 직접 승인하지 않음
```

이 역할 분리가 중요한 이유는 ROS2 통신 문제와 IsaacSim 제어 문제를 분리해서 디버깅할 수 있기 때문이다. Action Server가 보이지 않으면 Bridge 또는 DDS 문제로 보고, command JSON은 생성되지만 AMR이 움직이지 않으면 controller 또는 stage 문제로 볼 수 있다.

### ROS2 Action과 JSON command 변환 흐름

```text
[Action Goal]
workstation_id = WS05
target_location = sg2_in_03_B
preferred_amr_name = AMR_04
require_preferred_amr = true
        |
        v
[Bridge]
command_id 생성
입력 필드 정규화
중복 명령 검사
JSON command 작성
        |
        v
[bridge_queue/commands/CMD_xxx.json]
```

예시 command JSON은 다음과 같은 의미를 가진다.

```json
{
  "command_id": "CMD_f9bb42e729e8",
  "workstation_id": "WS07",
  "target_location": "sg2_in_01_B",
  "target_x": 0.0,
  "target_y": 0.0,
  "target_yaw": 0.0,
  "preferred_amr_name": "AMR_05",
  "preferred_amr": "AMR_05",
  "require_preferred_amr": true,
  "created_at": 1781167628.6863856
}
```

| 필드 | 의미 |
| --- | --- |
| `command_id` | 명령 추적용 고유 ID |
| `workstation_id` | 이동시킬 작업대 ID |
| `target_location` | SG, stage 등 논리 목적지 이름 |
| `target_x`, `target_y`, `target_yaw` | 좌표 기반 명령을 사용할 때의 목표 pose |
| `preferred_amr_name` | 특정 AMR을 우선 또는 강제로 배정할 때 사용하는 이름 |
| `require_preferred_amr` | true이면 해당 AMR이 아니면 작업을 수행하지 않음 |
| `created_at` | 명령 생성 시각 |

### Bridge Lifecycle

Bridge는 command를 생성한 뒤 끝나는 것이 아니라 command의 생명주기를 추적한다.

```text
RECEIVED
→ ADMITTED
→ COMMAND_WRITTEN
→ RUNNING
→ FEEDBACK_PUBLISHING
→ RESULT_RECEIVED
→ DONE
```

실패 또는 취소가 발생하면 다음 흐름으로 정리된다.

```text
RECEIVED
→ ADMITTED
→ COMMAND_WRITTEN
→ CANCEL_REQUESTED 또는 TIMEOUT
→ RESULT_FAILED
→ CLEANUP
```

### Admission Guard

Bridge v43에서 추가한 admission guard는 잘못된 명령이 controller로 내려가기 전에 차단하는 방어 계층이다.

```text
새 command 수신
→ workstation key 생성
→ target key 생성
→ AMR key 생성
→ active registry와 비교
→ 중복 없음: command JSON 작성
→ 중복 있음: Action result를 failed로 반환하고 reject
```

관리하는 active registry는 다음과 같다.

```text
_active_commands      : command_id 기준 전체 active command
_active_workstations  : workstation_id 기준 중복 작업대 방지
_active_targets       : target_location, target_cell, target_qr_id 기준 목적지 중복 방지
_active_amrs          : preferred_amr 기준 AMR 중복 작업 방지
```

reject 기준은 다음과 같다.

```text
DUPLICATE_WORKSTATION : 같은 작업대가 이미 다른 명령에서 사용 중
DUPLICATE_TARGET      : 같은 목적지 또는 같은 target_cell이 이미 사용 중
DUPLICATE_AMR         : 같은 preferred AMR이 이미 다른 작업 수행 중
INVALID_GOAL          : 필수 필드가 부족하거나 해석 불가능한 goal
```

이 구조의 핵심은 “controller가 해결할 수 없는 명령은 controller에 보내지 않는다”는 것이다. 같은 목적지에 작업대 2개를 넣는 문제는 경로계획이나 Arbiter로 해결할 수 없기 때문에 Bridge에서 reject하는 것이 맞다.

### Bridge Queue의 파일 의미

| 폴더 | 작성 주체 | 읽는 주체 | 의미 |
| --- | --- | --- | --- |
| `commands/` | Bridge | Controller | 실행할 작업 명령 |
| `status/` | Controller | Bridge | 진행 중인 작업 상태 |
| `results/` | Controller | Bridge | 성공/실패 결과 |
| `cancel/` | Bridge | Controller | 취소 요청 |
| `done/` | Bridge 또는 Controller | Bridge/Controller | 처리 완료 표시 |

---

## 4.5 Global Arbiter 상세 구조

Global Arbiter는 다중 AMR 환경에서 각 AMR의 이동을 중앙에서 승인하는 계층이다. 각 AMR이 개별적으로 A* 경로를 계산하더라도, 동시에 움직이면 같은 cell에 들어가거나 서로 자리를 맞바꾸는 충돌이 생길 수 있다. 이를 막기 위해 매 tick마다 모든 AMR의 다음 이동 후보를 모아 승인/거부를 결정한다.

### Global Arbiter가 필요한 이유

AMR이 1대라면 경로계획 결과를 그대로 실행해도 큰 문제가 없다. 하지만 AMR 5대가 동시에 움직이면 다음 문제가 발생한다.

```text
1. 두 AMR이 같은 tick에 같은 cell로 이동하려는 same-cell 충돌
2. 두 AMR이 서로의 현재 위치를 맞바꾸려는 edge-swap 충돌
3. 작업대를 들고 있는 AMR의 footprint가 다른 AMR 또는 작업대와 겹치는 문제
4. 좁은 SG 진입 구간에서 여러 AMR이 동시에 진입하는 병목
5. 한 AMR이 기다리기만 하고 우회하지 못하는 starvation 문제
```

Global Arbiter는 이러한 충돌을 AMR 개별 planner가 아니라 전체 fleet 관점에서 검사한다.

### Tick 단위 승인 흐름

```text
1. 각 AMR이 현재 phase와 target_cell 기준으로 다음 이동 후보를 계산한다.
2. 각 AMR은 next_cell, current_cell, carry 여부, priority, planned_path를 Arbiter에 제출한다.
3. Arbiter는 모든 AMR의 후보를 한 번에 모은다.
4. 같은 cell을 요구하는 AMR이 있는지 검사한다.
5. 서로 current_cell과 next_cell을 맞바꾸는 edge-swap이 있는지 검사한다.
6. 작업대 운반 중인 AMR의 rack footprint가 다른 AMR이나 작업대와 겹치는지 검사한다.
7. Reservation Table과 비교해 시간축 충돌을 검사한다.
8. 승인 가능한 AMR만 이동시킨다.
9. 거부된 AMR은 WAIT 상태가 되거나 cost-aware reroute 판단으로 넘어간다.
```

### Arbiter 판단 의사코드

```text
for each tick:
    proposals = collect_next_cell_from_all_amrs()

    for proposal in sort_by_priority(proposals):
        if target_cell_already_approved(proposal.next_cell):
            reject(proposal, reason="SAME_CELL")
            continue

        if edge_swap_with_approved_move(proposal):
            reject(proposal, reason="EDGE_SWAP")
            continue

        if rack_footprint_conflict(proposal):
            reject(proposal, reason="FOOTPRINT_CONFLICT")
            continue

        if reservation_table_conflict(proposal):
            reject(proposal, reason="RESERVED_CELL")
            continue

        approve(proposal)
```

### 주요 충돌 차단 규칙

| 규칙 | 설명 | 처리 |
| --- | --- | --- |
| same-cell block | 두 AMR이 같은 next_cell을 요구 | 우선순위 높은 AMR 승인, 나머지 reject |
| edge-swap block | A가 B 위치로, B가 A 위치로 이동 | 둘 중 하나 reject |
| footprint block | 작업대 운반 AMR의 점유 영역이 다른 객체와 겹침 | 운반 AMR 또는 후순위 AMR reject |
| static blocker block | future route에 정적 작업대가 있음 | 해당 route reject |
| reservation block | 시간 예약 테이블에서 이미 점유된 cell | wait 또는 reroute |

### 작업대 운반 중 별도 제약

작업대를 들고 있는 AMR은 빈 AMR보다 더 보수적으로 처리했다.

```text
1. 대각선 이동 제한 또는 강한 penalty 적용
2. rack footprint 포함 충돌 검사
3. 좁은 SG 진입 구간에서 local macro route 우선 사용
4. 우회 경로가 너무 길면 WAIT 선택
5. 회전 또는 placing 중에는 QR cell 갱신 차단
```

이유는 작업대를 들고 있는 상태에서는 AMR 자체보다 점유 면적이 커지고, 회전 또는 대각선 이동 시 주변 작업대와 충돌할 가능성이 높기 때문이다.

### Convoy Following과 Tail Release

같은 방향으로 이동하는 AMR들이 한 줄로 움직이는 경우에는 앞 AMR이 빠져나간 cell을 뒤 AMR이 따라가는 방식이 가능하다. 이를 완전히 막으면 불필요한 대기가 증가한다.

```text
앞 AMR: cell A → cell B
뒤 AMR: cell C → cell A
```

이 경우 앞 AMR이 먼저 이동하고 tail cell이 release되는 구조라면 뒤 AMR의 진입을 허용할 수 있다. 단, 반대 방향으로 맞바꾸는 edge-swap과는 구분해야 한다.

### Priority Aging

특정 AMR이 계속 reject되면 wait_steps가 증가한다. 이때 단순히 매번 같은 우선순위로 검사하면 특정 AMR이 계속 밀릴 수 있다. 따라서 오래 기다린 AMR에는 aging 점수를 부여해 우선순위를 점진적으로 높이는 구조가 필요하다.

```text
priority_score = base_priority + wait_steps * aging_weight
```

작업대를 들고 있는 AMR은 통로를 계속 막을 수 있기 때문에 상황에 따라 더 높은 우선순위를 줄 수 있다. 반대로 좁은 구간에서는 작업대 운반 AMR의 무리한 우회를 막기 위해 carry penalty를 적용한다.

### Cost-aware Reroute와의 연결

Global Arbiter가 next_cell을 reject하면 controller는 다음 두 선택지를 비교한다.

```text
1. WAIT: 현재 위치에서 기다린다.
2. REROUTE: reject된 cell을 임시 blocked cell로 보고 우회 경로를 다시 계산한다.
```

판단 흐름은 다음과 같다.

```text
Arbiter reject
→ rejected next_cell 확인
→ 해당 cell을 temporary blocked cell로 설정
→ detour A* 실행
→ wait_cost 계산
→ detour_cost 계산
→ wait_cost <= detour_cost 이면 WAIT
→ detour_cost < wait_cost 이면 REROUTE
```

따라서 Global Arbiter는 단순히 충돌을 막는 계층이고, Cost-aware Reroute는 Arbiter가 거부한 상황에서 기다릴지 돌아갈지를 판단하는 계층이다.

---

---

# 5. 날짜별 진행 타임라인

## 5월 29일

```text
- IsaacSim 실행 환경 확인
- Warehouse Creator 및 stage 구성 방식 확인
- IsaacSim에서 AMR/작업대를 배치하는 기준 조사
- 프로젝트 방향을 AMR Fleet 기반 물류 자동화 시뮬레이션으로 설정
```

## 5월 30일

```text
- 실제 창고 자동화 시나리오 구상
- AMR 5대 사용 구조 정리
- 작업대 한 면 4칸, 두 면 8칸 기준 작업 완료 후 이동하는 운영 방식 설계
- AMR이 작업대를 SG 또는 stage로 운반하는 전체 흐름 구상
```

## 6월 1일

```text
- 팀원별 구현 방향 조율
- 내 PC와 상대 PC 역할 분리 방향 정리
- 내 PC는 IsaacSim 실행, 상대 PC는 ROS2 명령 송신 구조로 설계
- 전체 시스템에서 bridge와 controller 역할 분리
```

## 6월 4일

```text
- GitHub README 구조 설계
- 프로젝트 개요, 목적, 시스템 구성, 실행 방법 정리
- 발표 자료에 들어갈 시스템 아키텍처 흐름 정리
```

## 6월 5일

```text
- ROS2 bridge 구조 확인
- IsaacSim controller가 기존 stage의 AMR/RACK prim을 제어하는 구조 확인
- 외부 명령이 command JSON으로 변환되는 흐름 확인
```

## 6월 6일

```text
- /manage_workstation Action Server 확인
- /amr_01/manage_workstation ~ /amr_05/manage_workstation per-AMR Action 구조 확인
- preferred_amr_name, require_preferred_amr 기반 특정 AMR 지정 구조 확인
- target_location과 target_cell 변환 구조 정리
```

## 6월 7일

```text
- bridge_queue commands/status/results 구조 정리
- AMR 작업 phase 구조 문서화
- QR 기반 위치 인식 구조 확인
- QR ID와 grid cell 매핑 구조 확인
```

## 6월 8일

```text
- Time A* 기반 경로계획 구조 분석
- Reservation Table 기반 time-indexed cell 예약 구조 분석
- edge swap 충돌 방지 구조 확인
- Global Arbiter가 tick 단위로 모든 AMR의 next_cell을 승인/거부하는 구조 분석
- same-cell, edge-swap, footprint 충돌 차단 기준 정리
- bridge 실행 및 ros2 action list 검증
```

## 6월 9일

```text
- AMR 초기 위치, 작업대 위치, SG 좌표 매핑 확인
- sg2_in_01_B, sg2_in_02_B, sg2_in_03_B, sg2_out_00_A 좌표 정리
- 작업대 운반 phase와 carry 상태 변화 확인
- 발표용 PPT 제작 및 슬라이드 흐름 구성
```

## 6월 10일

```text
- CycloneDDS 통신 문제 분석
- 상대 PC에서 action server가 보이지 않는 문제 확인
- thunderbolt0으로 고정된 DDS interface 문제 발견
- Wi-Fi interface wlp128s20f3 기준 수정 방향 정리
- Local Macro Entry/Exit 구조 분석
- wait/no_path 증가 원인 분석
```

## 6월 11일

```text
- current_run_full_log.txt 기반 tick별 AMR 상태 분석
- WS05와 WS06이 같은 target_location으로 들어가는 문제 확인
- sg2_in_03_B target_cell=(4,-5) 중복 문제 분석
- bridge v43 admission guard 패치 적용
- DUPLICATE_WORKSTATION / DUPLICATE_TARGET / DUPLICATE_AMR reject 구조 반영
- Bridge active registry와 result/cancel/timeout cleanup 흐름 정리
- Global Arbiter reject 이후 wait vs detour cost 판단 필요성 도출
- controller v42 cost-aware global reroute 패치 적용
- COST_DECISION 로그 구조 적용
- 다중 AMR 시나리오 테스트 완료
```

## 6월 12일

```text
- GitHub README 최종 정리
- PPT 최종 정리
- 발표 흐름 최종 확인
- 협동3 최종 발표 진행
- 질의응답 대응
```

---

# 6. 디버깅 원칙

협동3 디버깅은 다음 원칙으로 진행하였다.

```text
1. 증상을 먼저 확인한다.
2. bridge 문제인지 controller 문제인지 네트워크 문제인지 분리한다.
3. 로그에서 AMR state, cell, target, carry, wait, no_path를 확인한다.
4. 명령 문제가 먼저인지 경로계획 문제가 먼저인지 구분한다.
5. 정상 작동 중인 구조는 유지하고, 문제가 발생한 계층만 수정한다.
6. 임시 수정이 아니라 재발 방지 구조를 만든다.
7. 패치 후 py_compile, ros2 action list, IsaacSim 실행 로그로 검증한다.
8. README와 발표 자료에 문제 원인과 해결 결과를 함께 반영한다.
```

특히 협동3에서는 다음 구분이 중요했다.

```text
bridge 문제:
- Action server가 안 보임
- command JSON이 생성되지 않음
- 중복 명령이 controller로 들어감

controller 문제:
- target_cell 변환 오류
- AMR phase 전환 오류
- LOCAL_ENTRY 병목
- wait/no_path 증가
- cost 판단 부족

네트워크 문제:
- 상대 PC에서 action discovery 안 됨
- CycloneDDS interface가 잘못 잡힘
- ROS_DOMAIN_ID 또는 RMW 설정 불일치

명령 시나리오 문제:
- 같은 target_location 중복
- 같은 workstation_id 중복
- 같은 AMR에 동시에 명령
```

Global Arbiter 문제:
```text
- 각 AMR의 개별 A* 경로는 존재하지만 첫 next_cell이 계속 reject됨
- same-cell 또는 edge-swap 충돌로 tick 단위 이동 승인이 거부됨
- 작업대 운반 footprint 때문에 일반 AMR보다 더 많은 cell이 막힘
- wait_steps가 증가하지만 우회 판단이 부족함
```

Bridge Lifecycle 문제:
```text
- command JSON은 생성되었지만 result가 회수되지 않음
- 완료된 command가 active registry에서 해제되지 않아 다음 명령이 계속 reject됨
- cancel 또는 timeout 후 active_workstations, active_targets, active_amrs 정리가 필요함
```


---

# 7. 주요 이슈별 상세 디버깅

---

# ISSUE 1. 상대 PC에서 ROS2 Action Server가 보이지 않는 문제

## 1. 증상

내 PC에서는 bridge node와 action server가 정상적으로 보였다.

```bash
ros2 node list
```

예상 출력:

```text
/fleet_manager_bridge_node
```

```bash
ros2 action list | grep manage_workstation
```

예상 출력:

```text
/amr_01/manage_workstation
/amr_02/manage_workstation
/amr_03/manage_workstation
/amr_04/manage_workstation
/amr_05/manage_workstation
/manage_workstation
```

하지만 상대 PC에서는 action server가 보이지 않거나 일부만 보이는 문제가 있었다.

## 2. 영향 범위

```text
- 상대 PC에서 AMR 명령을 보낼 수 없음
- Control Tower와 IsaacSim controller가 연결되지 않음
- bridge는 정상 실행 중이지만 외부 명령 연동이 불가능함
```

## 3. 원인 분석

처음에는 bridge node 문제처럼 보였지만, 내 PC에서는 action server가 정상적으로 보였기 때문에 bridge 자체는 살아 있었다.

따라서 문제를 다음처럼 분리했다.

```text
bridge 실행 문제인가? → 아님
Action Server 생성 문제인가? → 아님
상대 PC discovery 문제인가? → 가능성 높음
CycloneDDS interface 문제인가? → 확인 필요
```

CycloneDDS XML을 확인한 결과, NetworkInterface가 thunderbolt0으로 고정되어 있었다.

```xml
<NetworkInterface name="thunderbolt0"/>
<Peer address="192.168.100.20"/>
```

하지만 실제 통신하려던 네트워크는 Wi-Fi 대역이었다.

```text
내 PC Wi-Fi IP = 192.168.10.40
상대 PC Wi-Fi IP = 192.168.10.x
```

즉 DDS discovery가 Wi-Fi가 아니라 Thunderbolt 쪽으로만 나가고 있었기 때문에 상대 PC에서 action server를 발견하지 못한 것이다.

## 4. 해결

CycloneDDS XML의 interface를 Wi-Fi 장치로 변경했다.

```xml
<NetworkInterface name="wlp128s20f3"/>
```

peer도 같은 Wi-Fi 대역으로 맞춰야 한다.

```xml
<Peers>
  <Peer address="192.168.10.40"/>
  <Peer address="192.168.10.41"/>
</Peers>
```

양쪽 PC에서 환경변수도 동일하게 맞춰야 한다.

```bash
export ROS_DOMAIN_ID=119
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI=file://$HOME/.ros/cyclonedds_thunderbolt.xml
```

bridge 재시작:

```bash
pkill -f fleet_manager_bridge_node_gpu_v42_per_amr_actions.py
pkill -f run_bridge_gpu.sh

cd ~/isaaclab_ws/isaac_aruco/amr
./run_bridge_gpu.sh
```

## 5. 검증

```bash
ros2 action list | grep manage_workstation
```

정상 기준:

```text
/amr_01/manage_workstation
/amr_02/manage_workstation
/amr_03/manage_workstation
/amr_04/manage_workstation
/amr_05/manage_workstation
/manage_workstation
```

## 6. 최종 결과

상대 PC에서 AMR Action Server를 discovery할 수 있는 구조로 정리되었다.

## 7. 재발 방지

```text
1. ROS_DOMAIN_ID 양쪽 동일 확인
2. RMW_IMPLEMENTATION 양쪽 동일 확인
3. CycloneDDS XML의 NetworkInterface 확인
4. Wi-Fi 사용 시 thunderbolt0 고정 금지
5. action list가 안 보이면 bridge보다 DDS interface를 먼저 확인
```

---

# ISSUE 2. set -u 때문에 ROS setup.bash가 실패한 문제

## 1. 증상

명령 시퀀스 스크립트를 실행했을 때 다음 오류가 발생했다.

```text
/opt/ros/humble/setup.bash: line 8: AMENT_TRACE_SETUP_FILES: unbound variable
```

## 2. 원인

스크립트에서 `set -u`가 활성화되어 있었다.

`set -u`는 정의되지 않은 변수를 사용하면 바로 오류를 발생시키는 옵션이다. ROS2의 `setup.bash`는 내부에서 아직 정의되지 않은 환경변수를 참조할 수 있는데, 이때 `set -u` 때문에 setup 과정이 중단되었다.

## 3. 해결

ROS setup을 source할 때만 `set +u`로 unbound variable 검사를 잠시 끄고, 이후 다시 `set -u`를 켜는 방식으로 수정하였다.

```bash
set +u
source /opt/ros/humble/setup.bash
set -u
```

## 4. 검증

수정 후 `send_business_open_sequence_v1.sh`가 ROS 환경을 정상적으로 불러오고, action goal 전송 단계까지 진행되었다.

## 5. 최종 결과

ROS setup 단계에서 스크립트가 중단되는 문제가 해결되었다.

## 6. 재발 방지

ROS2 setup 파일을 source하는 shell script에서는 `set -u`를 직접 적용하지 않거나, setup 구간만 `set +u`로 감싸야 한다.

---

# ISSUE 3. 같은 목적지로 작업대 2개가 들어가는 문제

## 1. 증상

다중 AMR 테스트 중 AMR이 특정 위치에서 멈추고 wait/no_path가 계속 증가했다.

로그 분석 결과 다음 명령이 동시에 존재했다.

```text
WS05 → sg2_in_03_B → target_cell=(4,-5)
WS06 → sg2_in_03_B → target_cell=(4,-5)
```

즉 서로 다른 작업대가 같은 목적지로 들어가도록 명령이 생성되었다.

## 2. 영향 범위

```text
- AMR_04가 WS05를 먼저 target_cell=(4,-5)에 배치
- AMR_03이 WS06을 들고 같은 cell로 진입 시도
- 이미 목적지가 점유되어 AMR_03이 멈춤
- wait/no_path 증가
- 전체 시나리오가 지연됨
```

## 3. 처음 의심한 원인

처음에는 다음 가능성을 의심했다.

```text
1. A* 경로계획 실패
2. Reservation Table 오류
3. LOCAL_ENTRY route 문제
4. QR 위치 인식 오류
5. 작업대 footprint 충돌
```

하지만 로그를 보면 AMR이 막힌 이유는 “길이 없어서”가 아니라, 목적지 자체가 이미 다른 작업대로 점유된 것이었다.

## 4. 근본 원인

근본 원인은 경로계획이 아니라 명령 생성 문제였다.

```text
같은 target_location이 동시에 두 번 들어옴
→ 같은 target_cell로 변환됨
→ controller가 두 작업을 모두 받아들임
→ 물리적으로 불가능한 배치 시도 발생
```

즉 이 문제는 A*로 해결할 수 있는 문제가 아니다.

```text
돌아가도 목적지는 이미 점유됨
기다려도 목적지가 비워지지 않음
따라서 REROUTE가 아니라 REJECT가 맞음
```

## 5. 해결 방향

controller가 명령을 수행하기 전에 bridge 단계에서 잘못된 명령을 먼저 막도록 admission guard를 추가했다.

차단 조건:

```text
1. 같은 workstation_id 중복
2. 같은 target_location 중복
3. 같은 target_qr_id 중복
4. 같은 target_x/y 중복
5. 같은 preferred AMR 중복
```

## 6. 적용 파일

```text
fleet_manager_bridge_node_gpu_v43_guarded_actions.py
```

## 7. 적용 구조

bridge 내부에 active command registry를 추가했다.

```text
_active_commands
_active_workstations
_active_targets
_active_amrs
```

명령 수신 시 먼저 검사한다.

```text
새 명령 수신
→ workstation key 생성
→ target key 생성
→ AMR key 생성
→ active registry와 비교
→ 중복이면 reject
→ 중복이 아니면 command JSON 생성
```

## 8. 기대 로그

중복 target인 경우:

```text
ManageWorkstation rejected by admission guard |
reason=DUPLICATE_TARGET active_command=CMD_xxx key=location:sg2_in_03_B
```

중복 workstation인 경우:

```text
ManageWorkstation rejected by admission guard |
reason=DUPLICATE_WORKSTATION active_command=CMD_xxx key=workstation:WS05
```

중복 AMR인 경우:

```text
ManageWorkstation rejected by admission guard |
reason=DUPLICATE_AMR active_command=CMD_xxx key=amr:AMR_03
```

## 9. 최종 결과

동일한 목적지에 두 작업대가 동시에 배정되는 문제가 bridge에서 사전에 차단되는 구조가 되었다.

## 10. 성과

```text
기존:
잘못된 명령도 controller로 들어감
→ AMR이 실제 주행 중 멈춤
→ 로그 분석 후 원인 파악 필요

개선:
잘못된 명령을 bridge에서 즉시 reject
→ controller가 불가능한 작업을 받지 않음
→ 다중 AMR 테스트 안정성 증가
```

---

# ISSUE 4. 같은 workstation_id가 중복 명령으로 들어가는 문제

## 1. 증상

어떤 작업대가 이미 작업 중인데, 같은 workstation_id로 새로운 명령이 다시 들어오는 문제가 있었다.

예시:

```text
CMD_1: WS05 → sg2_in_03_B
CMD_2: WS05 → sg2_in_02_B
```

## 2. 문제점

같은 작업대는 동시에 두 개의 목적지로 이동할 수 없다.

만약 controller가 두 명령을 모두 받으면 다음 문제가 발생할 수 있다.

```text
1. rack.assigned 상태 충돌
2. carried_by 상태 충돌
3. 두 AMR이 같은 작업대를 가지러 감
4. 먼저 잡은 AMR과 나중 명령이 충돌
5. result/status 반환이 꼬임
```

## 3. 해결

bridge admission guard에서 workstation key를 만들어 active registry에 등록한다.

```text
workstation:WS05
```

이미 active 상태이면 새 명령을 reject한다.

## 4. 최종 결과

작업대 단위 중복 명령이 차단되었다.

## 5. 성과

작업대 상태가 꼬이는 문제를 사전에 막을 수 있게 되었다.

---

# ISSUE 5. 특정 AMR에 중복 명령이 들어가는 문제

## 1. 증상

per-AMR action을 사용할 경우, 특정 AMR에 직접 명령을 내릴 수 있다.

예시:

```text
/amr_03/manage_workstation
```

이 구조는 시연에는 편리하지만, AMR_03이 이미 작업 중인데 다시 AMR_03으로 명령이 들어가면 문제가 된다.

## 2. 원인

per-AMR action은 command JSON에 다음 값을 추가한다.

```json
{
  "preferred_amr_name": "AMR_03",
  "preferred_amr": "AMR_03",
  "require_preferred_amr": true
}
```

이때 bridge가 AMR_03의 active 상태를 관리하지 않으면, 같은 AMR에 여러 명령이 동시에 들어갈 수 있다.

## 3. 해결

bridge admission guard에서 AMR key를 추가했다.

```text
amr:AMR_03
```

해당 AMR이 active command에 등록되어 있으면 새 명령을 reject한다.

## 4. 최종 결과

특정 AMR에 중복 명령이 들어가는 문제가 차단되었다.

## 5. 성과

per-AMR action을 사용하면서도 AMR 단위 작업 충돌을 줄일 수 있게 되었다.

---

# ISSUE 6. LOCAL_ENTRY에서 AMR이 정적 작업대 때문에 막히는 문제

## 1. 증상

AMR이 작업대를 들고 SG 구역으로 진입하는 LOCAL_ENTRY 단계에서 특정 경로를 따라가다가 막히는 문제가 있었다.

예상 route에는 문제가 없어 보였지만, 실제로는 future route에 정적 작업대가 존재하는 경우가 있었다.

## 2. 원인

기존 local macro route는 시작 cell 또는 바로 다음 cell 위주로 검사했다.

하지만 route 전체 중간에 이미 정적 작업대가 있는 경우, AMR이 출발한 뒤 나중에 막히는 문제가 발생할 수 있었다.

## 3. 문제 예시

```text
AMR_02가 LOCAL_ENTRY route를 선택
route 중간에 WS07 또는 다른 static rack이 존재
처음에는 출발 가능
중간에 도착하면 더 이상 진행 불가
```

## 4. 해결 방향

local macro route를 선택할 때 future route에 있는 static blocker를 사전에 확인하도록 개선했다.

개선 개념:

```text
route 후보 생성
→ route 전체 QR valid 확인
→ route 중간 static rack 여부 확인
→ static blocker가 있으면 해당 route reject
→ 다른 route 후보 선택
```

## 5. 로그 방향

```text
LOCAL_MACRO ROUTE REJECT |
blockers=[...]
route=[...]
```

또는:

```text
LOCAL_MACRO B DETOUR SELECT |
route=[...]
```

## 6. 최종 결과

local macro route가 정적 작업대를 관통하는 경로를 선택하지 않도록 개선되었다.

## 7. 성과

SG 진입 구간의 안정성이 증가했고, AMR이 작업대 근처에서 불필요하게 멈추는 상황을 줄일 수 있게 되었다.

---


# ISSUE 6-1. Global Arbiter 구조가 문서에 충분히 드러나지 않는 문제

## 1. 증상

기존 문서에는 Time A*, Reservation Table, edge swap, cost-aware reroute는 언급되어 있었지만, 실제 다중 AMR 이동을 최종 승인하는 Global Arbiter의 역할이 별도 구조로 충분히 설명되지 않았다.

이 때문에 GitHub 문서만 읽으면 다음 구분이 모호해질 수 있었다.

```text
A* Planner가 하는 일
Reservation Table이 하는 일
Global Arbiter가 하는 일
Cost-aware Reroute가 하는 일
Bridge가 하는 일
```

## 2. 문제점

Global Arbiter 설명이 부족하면 다중 AMR이 왜 멈추는지, 왜 wait/no_path가 증가하는지, 왜 특정 상황에서 reroute가 필요한지 설명하기 어렵다.

예를 들어 AMR이 움직이지 않는 상황은 단순히 “경로가 없음”이 아니라 다음 중 하나일 수 있다.

```text
1. A*가 실제로 경로를 찾지 못함
2. 경로는 있지만 첫 next_cell이 Arbiter에서 reject됨
3. 같은 목적지로 명령이 중복되어 물리적으로 갈 수 없음
4. 작업대 footprint 때문에 통로가 막힘
5. Local Macro route 중간에 static blocker가 있음
```

## 3. 정리한 구조

Global Arbiter는 다음 순서로 정리하였다.

```text
AMR별 next_cell 후보 생성
→ 후보 전체 수집
→ same-cell 검사
→ edge-swap 검사
→ rack footprint 검사
→ reservation table 검사
→ 승인/거부 결정
→ 거부된 AMR은 WAIT 또는 REROUTE 판단
```

## 4. 문서 보강 내용

```text
1. Global Arbiter의 목적 추가
2. tick 단위 이동 승인 흐름 추가
3. same-cell / edge-swap / footprint 충돌 차단 기준 추가
4. Convoy Following과 Tail Release 설명 추가
5. Priority Aging 설명 추가
6. Cost-aware Reroute와의 연결 구조 추가
```

## 5. 최종 결과

GitHub 문서에서 다중 AMR 주행 안정화 구조가 다음처럼 계층적으로 설명되도록 개선되었다.

```text
Bridge: 잘못된 명령 사전 차단
A*: 개별 AMR 경로 후보 생성
Reservation Table: 시간축 cell 예약
Global Arbiter: tick 단위 전체 AMR 이동 승인
Cost-aware Reroute: reject 발생 시 WAIT/REROUTE 판단
```

---

# ISSUE 7. AMR이 막혔을 때 기다림과 우회를 판단하지 못하는 문제

## 1. 증상

AMR이 막혔을 때 기존 구조는 대부분 기다리는 방식이었다.

```text
A* 경로 계산
→ 첫 이동 cell 제안
→ Global Arbiter가 전체 AMR 이동 후보와 비교
→ same-cell / edge-swap / footprint / reservation 충돌이면 reject
→ wait 증가
→ 다음 tick에서 다시 시도
```

이 방식은 안전하지만 효율적이지 않았다.

예를 들어 다음 상황이 있었다.

```text
현재 경로:
3초 기다리면 통과 가능

우회 경로:
2칸 돌아가면 바로 통과 가능
```

기존 구조는 이런 상황에서 “기다릴지, 돌아갈지”를 cost로 비교하지 못했다.

## 2. 영향 범위

```text
- AMR이 불필요하게 오래 대기
- 병목 구간에서 wait_steps 증가
- 전체 작업 완료 시간이 길어짐
- 5대 동시 작업 시 혼잡이 누적됨
```

## 3. 설계 판단

처음에는 LOCAL_ENTRY에만 cost 판단을 넣는 방식을 고려했다. 하지만 실제로는 다음 모든 phase에서 막힘이 발생할 수 있다.

```text
TO_RACK
LOCAL_ENTRY
TO_TARGET
LOCAL_EXIT
RETURN_HOME
RANDOM
```

따라서 각 phase에 개별적으로 cost 코드를 넣는 방식은 적합하지 않았다.

잘못된 방식:

```text
TO_RACK 함수에 cost 추가
TO_TARGET 함수에 cost 추가
LOCAL_ENTRY 함수에 cost 추가
RETURN_HOME 함수에 cost 추가
```

이 방식은 코드 중복이 많고 유지보수가 어렵다.

최종 결정:

```text
모든 주행 phase가 공통으로 거치는 global planning/approval 계층에 cost-aware reroute를 추가한다.
```

## 4. 해결 구조

적용한 구조:

```text
1. 기존 A*로 기본 경로 계산
2. global arbiter가 첫 이동을 reject
3. rejected first cell을 temporary blocked cell로 지정
4. 해당 cell을 피하는 detour A*를 다시 계산
5. wait_cost 계산
6. detour_cost 계산
7. wait_cost <= detour_cost 이면 WAIT
8. detour_cost < wait_cost 이면 REROUTE
```

## 5. cost 기준

### wait cost

```text
wait_cost =
기본 대기 비용
+ wait_steps 기반 aging 비용
+ 오래 기다린 AMR에 대한 강제 우회 보정
```

### detour cost

```text
detour_cost =
우회 경로 길이
+ 기존 경로 대비 추가 거리
+ 회전 비용
+ 작업대 운반 penalty
+ 혼잡 cell 회피 효과
```

## 6. 작업대 운반 중 penalty

작업대를 들고 있는 AMR은 빈 AMR보다 우회에 더 보수적이어야 한다.

이유:

```text
1. 작업대 footprint가 큼
2. 회전 시 충돌 위험 증가
3. 대각선 이동 제한
4. 좁은 통로에서 우회가 오히려 위험할 수 있음
```

따라서 carry 상태에서는 우회 허용 범위를 더 작게 설정했다.

예시:

```bash
export AMR_COST_AWARE_REROUTE_MAX_EXTRA_CELLS_CARRY=3.0
export AMR_COST_AWARE_REROUTE_MAX_EXTRA_CELLS_EMPTY=5.0
```

## 7. 적용 파일

```text
amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py
```

## 8. 기대 로그

우회 선택:

```text
COST_DECISION | AMR_03 decision=REROUTE
wait_cost=12.4
detour_cost=8.7
blocked=(3,-5)
new_next=(2,-5)
```

대기 선택:

```text
COST_DECISION | AMR_04 decision=WAIT
wait_cost=2.1
detour_cost=9.5
reason=wait_cheaper
```

## 9. 최종 결과

AMR이 막혔을 때 단순히 기다리는 방식에서 벗어나, 기다림과 우회 비용을 비교하는 구조가 추가되었다.

## 10. 성과

```text
기존:
막히면 wait 증가

개선:
막히면 WAIT/REROUTE 판단

기술적 의미:
단순 충돌 회피에서 비용 기반 주행 판단으로 개선
```

---

# ISSUE 8. QR MISS 및 위치 동기화 문제

## 1. 증상

AMR 하단 카메라가 QR을 읽지 못하거나, 읽은 QR 위치가 현재 AMR 위치와 맞지 않는 문제가 발생할 수 있었다.

로그 유형:

```text
QR MISS
QR REJECTED
QR UNMAPPED
QR LOCALIZED SAFE
```

## 2. 원인 가능성

```text
1. 카메라가 QR을 제대로 바라보지 못함
2. 렌더링 프레임이 늦음
3. AMR 이동 중 QR을 읽음
4. 인접 QR 또는 앞쪽 QR을 잘못 읽음
5. transform 위치와 QR 위치가 일시적으로 불일치
```

## 3. 기존 문제

QR을 읽었다고 바로 current_cell을 바꾸면 AMR 위치가 갑자기 튈 수 있다.

예시:

```text
기존 cell=(-4,-5)
감지 QR cell=(-2,-5)
```

이런 식으로 2칸 이상 jump가 발생하면 경로계획이 꼬일 수 있다.

## 4. 해결 구조

QR safety gate를 적용했다.

조건:

```text
1. AMR이 이동 중이면 QR cell 갱신 금지
2. LIFTING / PLACING / ROTATING 중이면 QR cell 갱신 금지
3. 기존 cell과 감지 cell의 jump가 너무 크면 reject
4. world 위치와 QR 위치 차이가 너무 크면 reject
5. 같은 QR을 일정 횟수 이상 안정적으로 읽었을 때만 accept
```

## 5. 로그 개선

디버깅을 쉽게 하기 위해 QR MISS 로그에는 다음 정보가 필요하다.

```text
AMR 이름
camera path
현재 state
moving 여부
현재 cell
target cell
carry 상태
마지막 QR
마지막 rejected reason
```

## 6. 최종 결과

QR을 읽지 못하거나 잘못 읽었을 때도 planner의 current_cell이 갑자기 튀지 않도록 안정화되었다.

## 7. 성과

QR 기반 위치 인식의 신뢰성을 높였고, 위치 오인식으로 인한 잘못된 경로계획 가능성을 줄였다.

---

# ISSUE 9. target_location과 target_cell 매핑 문제

## 1. 증상

사용자가 IsaacSim에서 본 실제 위치와 코드상 target_location 매핑이 다르게 느껴지는 문제가 있었다.

예를 들어 사용자는 다음 위치를 확인했다.

```text
sg2_out은 IsaacSim에서 대략 (-4.5, 7.5) ~ (-4.5, 9.0)
```

또 AMR 초기 위치는 다음과 같이 확인했다.

```text
AMR_01 = (-6.0, -9.0)
AMR_02 = (-6.0, -7.5)
```

## 2. 원인

controller는 target_location을 내부 LOCATION_TARGETS에서 world 좌표로 변환하고, 다시 grid cell로 변환한다.

기준:

```text
GRID_SPACING = 1.5
cell_x = round(world_x / 1.5)
cell_y = round(world_y / 1.5)
```

예시:

```text
world=(6.0, 1.5)
→ cell=(4,1)

world=(6.0, -7.5)
→ cell=(4,-5)

world=(-4.5, 9.0)
→ cell=(-3,6)
```

## 3. 해결

사용자가 IsaacSim에서 직접 본 위치를 기준으로 target_location과 cell을 다시 확인했다.

주요 매핑:

```text
sg2_in_01_B  = world=(6.0, 1.5)   → cell=(4,1)
sg2_in_02_B  = world=(6.0, -3.0)  → cell=(4,-2)
sg2_in_03_B  = world=(6.0, -7.5)  → cell=(4,-5)
sg2_out_00_A = world=(-4.5, 9.0)  → cell=(-3,6)
sg2_out_00_B = world=(-4.5, 7.5)  → cell=(-3,5)
```

## 4. 최종 결과

world 좌표와 grid cell 기준을 README 및 발표 자료에 명확히 정리할 수 있게 되었다.

## 5. 성과

좌표 혼동으로 인해 작업대가 잘못된 위치로 이동하는 문제를 줄일 수 있게 되었다.

---

# ISSUE 10. Redis 상태 publish 구조

## 1. 목적

AMR 상태를 외부 시스템이나 Control Tower에서 확인할 수 있도록 Redis 상태 publish 구조를 사용했다.

## 2. publish 항목

```text
state
current_qr_id
target_qr_id
carrying_workstation_id
battery
```

## 3. 의미

```text
state:
AMR 상태 IDLE / MOVING / ERROR 등

current_qr_id:
현재 AMR 위치

target_qr_id:
AMR 목표 위치

carrying_workstation_id:
현재 운반 중인 작업대

battery:
AMR 배터리 상태
```

## 4. 확인한 구조

controller는 AMR 상태가 바뀌었을 때 Redis hash에 업데이트하도록 구성되어 있었다.

## 5. 최종 결과

외부 시스템에서 AMR의 현재 상태를 추적할 수 있는 기반이 마련되었다.

---

# 8. 적용한 주요 패치 정리

## 8.1 Bridge v43 Admission Guard

### 적용 파일

```text
fleet_manager_bridge_node_gpu_v43_guarded_actions.py
```

### 목적

잘못된 명령이 IsaacSim controller로 들어가기 전에 bridge에서 차단한다.

### 추가된 기능

```text
1. active command registry
2. active workstation tracking
3. active target tracking
4. active AMR tracking
5. duplicate command reject
6. command 완료/실패/취소/timeout 시 active registry 해제
```

### 차단 조건

```text
DUPLICATE_WORKSTATION
DUPLICATE_TARGET
DUPLICATE_AMR
```

### 최종 효과

```text
같은 작업대나 같은 목적지로 중복 명령이 들어와 AMR이 멈추는 문제를 사전에 방지
```

---

## 8.2 Local Macro Static Blocker 개선

### 적용 대상

```text
choose_sg_local_entry_route()
local_macro_cell_is_free()
advance_local_macro_entry()
```

### 목적

LOCAL_ENTRY route가 정적 작업대가 있는 cell을 관통하지 않도록 한다.

### 최종 효과

```text
SG 진입 구간에서 작업대와 route 충돌 가능성 감소
```

---

## 8.3 Cost-aware Global Reroute

### 적용 파일

```text
amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py
```

### 목적

AMR이 막혔을 때 기다리는 것과 우회하는 것을 cost로 비교한다.

### 적용 범위

```text
TO_RACK
LOCAL_ENTRY
TO_TARGET
LOCAL_EXIT
RETURN_HOME
RANDOM
```

### 최종 효과

```text
막힘 상황에서 단순 wait가 아니라 WAIT/REROUTE 판단 가능
```

---

## 8.4 Global Arbiter 승인 레이어 정리

### 목적

여러 AMR이 동시에 움직일 때 각 AMR의 개별 경로계획 결과를 그대로 실행하지 않고, 전체 fleet 기준으로 다음 이동을 승인한다.

### 정리한 기능

```text
1. tick 단위 next_cell 후보 수집
2. same-cell 충돌 차단
3. edge-swap 충돌 차단
4. rack footprint 충돌 차단
5. Reservation Table 기반 시간축 충돌 차단
6. convoy following과 tail-release 허용 조건 구분
7. reject 발생 시 wait 또는 cost-aware reroute 판단으로 연결
```

### 최종 효과

```text
다중 AMR이 동시에 주행할 때 개별 A*만으로 막기 어려운 순간 충돌을 중앙에서 차단
```

---

## 8.5 Bridge Queue 및 Active Registry 정리

### 목적

ROS2 Action 명령과 IsaacSim controller 실행 상태를 command_id 기준으로 추적하고, 중복 명령을 controller에 전달하기 전에 차단한다.

### 정리한 기능

```text
1. command_id 기반 command/status/result 매칭
2. active_workstations 관리
3. active_targets 관리
4. active_amrs 관리
5. cancel/timeout/result 이후 cleanup
6. DUPLICATE_WORKSTATION / DUPLICATE_TARGET / DUPLICATE_AMR reject
```

### 최종 효과

```text
잘못된 외부 명령이 실제 AMR 주행 오류로 번지기 전에 Bridge 단계에서 차단
```

---

# 9. 실행 및 검증 명령어

## 9.1 bridge 실행

```bash
cd ~/isaaclab_ws/isaac_aruco/amr
./run_bridge_gpu.sh
```

## 9.2 bridge 확인

```bash
ros2 node list
ros2 action list | grep manage_workstation
```

정상 출력:

```text
/fleet_manager_bridge_node
/amr_01/manage_workstation
/amr_02/manage_workstation
/amr_03/manage_workstation
/amr_04/manage_workstation
/amr_05/manage_workstation
/manage_workstation
```

## 9.3 IsaacSim controller 실행

IsaacSim Script Editor:

```python
exec(open('/home/rokey/isaaclab_ws/isaac_aruco/amr/amr_live_existing_stage_true8_qr_camera_controller_gpu.py', encoding='utf-8').read())
```

## 9.4 logger 실행

```python
exec(open('/home/rokey/isaaclab_ws/isaac_aruco/amr/amr_timeline_full_logger_stop_reset.py', encoding='utf-8').read())
```

## 9.5 business 시퀀스 실행

```bash
cd ~/isaaclab_ws/isaac_aruco/amr
./send_business_open_sequence_v1.sh
```

## 9.6 command/status/result 확인

```bash
ls -al ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/commands
ls -al ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/status
ls -al ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/results
```


## 9.7 Bridge active command 검증 관점

중복 명령을 테스트할 때는 다음 순서로 확인한다.

```bash
# commands 생성 여부
ls -al ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/commands

# status 갱신 여부
ls -al ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/status

# result 반환 여부
ls -al ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/results

# cancel 요청 여부
ls -al ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/cancel

# 완료 처리 여부
ls -al ~/isaaclab_ws/isaac_aruco/amr/bridge_queue/done
```

중복 명령 차단이 정상이라면 Bridge 로그에서 다음 유형을 확인할 수 있어야 한다.

```text
DUPLICATE_WORKSTATION
DUPLICATE_TARGET
DUPLICATE_AMR
```

## 9.8 Global Arbiter 및 Cost Decision 로그 검증 관점

AMR이 멈췄을 때는 단순히 no_path만 볼 것이 아니라 Arbiter reject와 cost decision을 같이 확인한다.

```text
1. AMR 현재 cell
2. AMR target cell
3. 제안된 next_cell
4. Global Arbiter reject reason
5. wait_steps
6. wait_cost
7. detour_cost
8. 최종 decision = WAIT 또는 REROUTE
```

기대 로그 예시는 다음과 같다.

```text
GLOBAL_ARBITER_REJECT | amr=AMR_03 reason=SAME_CELL blocked=(4,-5)
COST_DECISION | AMR_03 decision=REROUTE wait_cost=12.4 detour_cost=8.7
```

---

# 10. 핵심 디버깅 요약표

| 구분           | 문제                              | 원인                                      | 해결                                    | 최종 상태 |
| ------------ | ------------------------------- | --------------------------------------- | ------------------------------------- | ----- |
| ROS2 Network | 상대 PC에서 action server가 안 보임     | CycloneDDS interface가 thunderbolt0으로 고정 | Wi-Fi interface wlp128s20f3 기준으로 수정   | 완료    |
| Shell Script | setup.bash unbound variable 오류  | set -u와 ROS setup.bash 충돌               | source 구간만 set +u 적용                  | 완료    |
| Bridge       | 같은 target_location 중복 명령        | 외부 명령 생성 시 slot 중복                      | admission guard에서 DUPLICATE_TARGET 차단 | 완료    |
| Bridge       | 같은 workstation_id 중복 명령         | 작업대 active 상태 관리 부족                     | DUPLICATE_WORKSTATION 차단              | 완료    |
| Bridge       | 같은 AMR 중복 명령                    | per-AMR action 중복 사용 가능                 | DUPLICATE_AMR 차단                      | 완료    |
| Controller   | LOCAL_ENTRY 중간 route 막힘         | future route의 static blocker 미검사        | local macro route blocker 검사 반영       | 완료    |
| Controller   | wait만 하고 우회 판단 부족               | Global Arbiter reject 후 wait_cost/detour_cost 비교 없음 | cost-aware global reroute 적용          | 완료    |
| QR           | QR MISS 또는 cell jump 가능성        | 이동 중 오인식, 인접 QR 인식                      | QR safety gate 적용                     | 완료    |
| 좌표계          | target_location과 실제 stage 위치 혼동 | world/grid 변환 기준 불명확                    | 1.5m grid 기준 좌표 정리                    | 완료    |
| 문서화          | 구현 내용이 흩어짐                      | 코드/로그/발표 자료가 분리됨                        | README/PPT/DEBUGGING.md 구조화           | 완료    |

---

# 11. 최종 구현 상태

최종적으로 협동3 시스템은 다음 구조로 정리되었다.

```text
1. 상대 PC 또는 Control Tower에서 ROS2 Action 명령 전송
2. bridge가 goal을 JSON command로 변환
3. IsaacSim controller가 command를 읽고 AMR 배정
4. AMR이 작업대 위치로 이동
5. 작업대 LIFTING
6. target_location에 따라 SG 또는 stage로 이동
7. LOCAL_ENTRY/TO_TARGET/PLACING 수행
8. 작업대 배치
9. AMR LOCAL_EXIT 또는 RETURN_HOME 수행
10. status/result JSON 작성
11. bridge가 ROS2 feedback/result 반환
```

최종 적용된 주요 개선은 다음과 같다.

```text
1. per-AMR Action Server 구조
2. bridge_queue 기반 JSON 연동 구조
3. QR 기반 위치 인식
4. 8-way Time A* 경로계획
5. Reservation Table 기반 충돌 회피
6. Local Macro Route 기반 SG 진입 안정화
7. bridge admission guard
8. Global Arbiter 기반 tick 단위 이동 승인 구조
9. cost-aware global reroute
10. CycloneDDS Wi-Fi 통신 설정
11. README/PPT/DEBUGGING 문서화
```

---

# 12. 최종 성과

## 12.1 기술적 성과

```text
1. IsaacSim에서 AMR 5대와 작업대 다수를 활용한 물류 시뮬레이션 구축
2. ROS2 Action 기반 외부 명령 수신 구조 구현
3. /manage_workstation 및 /amr_01~05/manage_workstation 구조 확보
4. bridge_queue 기반 command/status/result 연동 구조 구현
5. QR 기반 AMR 위치 인식 구조 적용
6. Time A* + Reservation Table 기반 다중 AMR 충돌 회피 적용
7. SG 진입을 위한 Local Macro Route 구조 적용
8. 중복 명령 문제를 bridge admission guard로 차단
9. Global Arbiter 기반으로 다중 AMR의 tick 단위 이동 승인 구조 정리
10. wait vs detour cost-aware global reroute 구조 적용
11. CycloneDDS interface 문제 해결로 PC 간 ROS2 통신 구조 안정화
```

---

## 12.2 팀장 역할 성과

```text
1. 프로젝트 주제와 방향 직접 기획
2. 실제 창고 자동화를 가정한 AMR Fleet 시나리오 설계
3. 팀원별 구현 방향 조율
4. 내 PC와 상대 PC 역할 분리 구조 정리
5. 전체 시스템 시나리오 구성
6. GitHub README 문서 작업
7. 발표용 PPT 제작
8. 발표 흐름 및 팀원별 발표 내용 조율
9. 발표 당일 최종 발표 수행
10. 디버깅 결과를 문서화하여 프로젝트 완성도 향상
```

---

## 12.3 문제 해결 성과

### 기존 문제

```text
1. 상대 PC에서 ROS2 action discovery가 안 됨
2. 같은 목적지에 작업대 2개가 들어감
3. 같은 작업대가 중복 명령을 받음
4. AMR이 막혔을 때 wait만 증가함
5. LOCAL_ENTRY route가 정적 작업대를 고려하지 못함
6. QR MISS 또는 위치 jump 가능성이 있음
7. 좌표계와 실제 stage 위치가 혼동됨
```

### 개선 후

```text
1. CycloneDDS Wi-Fi 설정으로 통신 구조 정리
2. bridge admission guard로 중복 target 차단
3. bridge admission guard로 중복 workstation 차단
4. bridge admission guard로 중복 AMR 차단
5. cost-aware global reroute로 WAIT/REROUTE 판단 가능
6. Local Macro route blocker 검사로 SG 진입 안정화
7. QR safety gate로 위치 인식 안정화
8. grid/world 좌표 변환 기준 문서화
```

---

# 13. 최종 결론

협동3 프로젝트의 디버깅은 단순히 오류를 하나씩 수정하는 과정이 아니라, AMR Fleet 시스템 전체를 안정적인 구조로 바꿔가는 과정이었다.

초기에는 IsaacSim에서 AMR과 작업대가 정상적으로 움직이는지 확인하는 수준이었지만, 프로젝트가 진행되면서 외부 PC에서 ROS2 Action 명령을 내리고, bridge가 이를 JSON으로 변환하고, IsaacSim controller가 실제 stage의 AMR을 움직이며, 결과를 다시 반환하는 전체 파이프라인을 구성하였다.

디버깅 과정에서 가장 중요한 판단은 문제를 계층별로 분리한 것이다. 상대 PC에서 action server가 안 보이는 문제는 controller 문제가 아니라 CycloneDDS interface 문제였고, AMR이 특정 위치에서 멈추는 문제는 단순 A* 문제가 아니라 중복 target command 문제였다. 또한 AMR이 막혔을 때 계속 기다리는 문제는 충돌 회피 실패가 아니라 wait와 detour를 비교하는 cost planner가 부족한 구조적 한계였다.

이러한 문제를 해결하기 위해 bridge에는 admission guard를 추가하여 중복 workstation, 중복 target, 중복 AMR 명령을 사전에 차단했고, controller에는 Global Arbiter 기반 tick 단위 이동 승인 구조와 cost-aware global reroute 구조를 정리하여 AMR이 막혔을 때 충돌을 막으면서 기다릴지 우회할지 판단할 수 있도록 개선하였다. 또한 Local Macro Route와 QR safety gate, CycloneDDS Wi-Fi 설정까지 정리하여 전체 시스템의 안정성을 높였다.

최종적으로 협동3 프로젝트는 IsaacSim 기반 AMR Fleet 물류 자동화 시뮬레이션으로 완성되었으며, 성웅은 팀장으로서 프로젝트 기획, 전체 시나리오 설계, 팀원 조율, GitHub 문서화, PPT 제작, 최종 발표를 담당했다. 기술적으로는 ROS2와 IsaacSim을 연결하는 핵심 구조와 다중 AMR 주행 안정성 개선을 주도하였다.

---

# 14. 한 줄 요약

협동3 프로젝트에서는 IsaacSim 기반 AMR Fleet 환경에서 ROS2 Action 명령을 받아 작업대를 픽업·운반·배치하는 전체 파이프라인을 구축했고, 중복 명령 차단과 cost-aware reroute를 통해 다중 AMR 주행 안정성과 효율성을 개선하였다.
