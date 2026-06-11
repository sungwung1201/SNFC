# 📡 통합 워크스페이스 병합을 위한 관제탑 필수 노드 및 컴포넌트 연동 가이드

본 문서는 현재 개발된 **로봇-AMR 지능형 물류창고 관제 시스템(Control Tower)**의 핵심 노드들과 통신 인터페이스, 그리고 데이터베이스 사양을 타 팀원 또는 다른 워크스페이스에 통합(Merge)할 수 있도록 정리한 **Handoff & Integration 가이드**입니다.

---

## 🗂️ 1. 전체 패키지 및 노드 토폴로지 구조

통합 시 대상 워크스페이스의 `src/` 디렉토리 하위에 다음 2개의 ROS 2 패키지가 그대로 이식되어야 합니다.

```text
src/
├── cobot3_interfaces/               # ROS 2 커스텀 인터페이스 정의 패키지 (C++ CMake)
│   ├── action/                      # 액션 파일 (.action)
│   ├── msg/                         # 메시지 파일 (.msg)
│   ├── srv/                         # 서비스 파일 (.srv)
│   ├── CMakeLists.txt
│   └── package.xml
│
└── cobot3/                          # 관제 코어 및 모의 디바이스 노드 패키지 (Python)
    ├── cobot3/                      # 실제 소스코드 폴더
    │   ├── control_tower_node.py    # [필수 1] 중앙 관제 스케줄러 노드
    │   ├── sim_sync_node.py         # [필수 2] 분산 시뮬레이션 상자 동기화 브릿지 노드
    │   ├── mock_amr_node.py         # [가상 1] 4대 AMR 주행 및 Action 구동 에뮬레이터 노드
    │   ├── mock_sg2_node.py         # [가상 2] 입고적재(sg2_in) 및 출고포장(sg2_out) 로봇 통합 에뮬레이터 노드
    │   └── mock_sg2_out_node.py     # [가상 3] 출고포장 로봇 단독 에뮬레이터 노드 (선택적 사용)
    ├── setup.py
    └── package.xml
```

---

## 🧩 2. 필수 ROS 2 커스텀 인터페이스 (`cobot3_interfaces`)

이 패키지는 모든 노드의 통신 규격을 담고 있으므로, **가장 먼저 컴파일(`colcon build --packages-select cobot3_interfaces`)되어야 합니다.**

### ① 서비스 (Services)
*   **`CheckWarehouseStatus.srv`**
    *   **용도**: 입고 시 동일 상자가 AMR로 이미 창고 직송 처리되었는지 여부 조회 및 패키지 DB 등록 검증
    *   **데이터**: `string customer_name`, `string package_id`, `string qr_id` ➔ `bool is_already_in_warehouse`
*   **`ReportInboundProgress.srv`**
    *   **용도**: 적재 로봇(sg2_in)이 작업대 슬롯에 상자를 올릴 때마다 관제탑에 보고하여 DB 상태 갱신 및 JIT 인터로킹 트리거링
    *   **데이터**: `string workstation_id`, `string robot_id`, `int32 filled_slots_count`, `string package_id` 등 ➔ `bool success`
*   **`TransitPackage.srv`**
    *   **용도**: 분산 시뮬레이터 간 컨베이어 끝에서 상자가 소멸 및 반대편 소환될 때 정보 전달 브릿지
    *   **데이터**: `string package_id`, `string destination_line` ➔ `bool success`
*   **`GetDailyPackageList.srv`**
    *   **용도**: 금일 처리해야 할 전체 물량 리스트를 DB로부터 질의
    *   **데이터**: `string request_date` ➔ `string[] package_ids`

### ② 액션 (Actions)
*   **`ManageWorkstation.action`**
    *   **용도**: 관제탑이 AMR에게 작업대 이송 명령 하향 (`DEPLOY`, `RETRIEVE`, `ROTATE` 등)
    *   **데이터**: `string workstation_id`, `string command_type`, `string target_location` 등 ➔ `bool success` ➔ `float32 distance_remaining` (피드백)
*   **`MovePackage.action`**
    *   **용도**: 관제탑이 AMR에게 단일 긴급 패키지 직송 명령 하향
    *   **데이터**: `string package_id`, `string destination_zone` ➔ `bool success` ➔ `string current_position`, `float32 progress` (피드백)
*   **`StartPackaging.action`**
    *   **용도**: 관제탑이 출고 포장 로봇(sg2_out)에게 작업대 패키지 포장 및 래핑 명령 하향
    *   **데이터**: `string workstation_id`, `string today_date` ➔ `bool success`, `string[] final_output_ids` ➔ `int32 completed_slots` (피드백)

### ③ 메시지 (Messages)
*   **`WorkstationSimTrigger.msg`**
    *   **용도**: Isaac Sim 환경과의 작업대 생성/삭제 연동을 위한 시뮬레이터 브릿지 메시지
    *   **데이터**: `string workstation_id`, `string location`, `string action` (`Spawn` or `Despawn`)

---

## ⚙️ 3. 필수 노드별 상세 명세

### 1️⃣ `control_tower` (`control_tower_node.py`)
*   **역할**: 전체 공정의 **메인 오케스트레이터(Brain)**. PostgreSQL DB와 Redis 우선순위 큐를 실시간 폴링하여 로봇 제어 명령(Action/Service)을 발행하고 JIT 안전 일시정지 제어 수행.
*   **구현 특징**:
    *   **MultiThreadedExecutor**를 사용하여 ROS 콜백 병렬 처리.
    *   **Thread-Safe DB Connection Pool** 및 `threading.RLock()` 뮤텍스 락 설계로 커서 충돌 원천 방지.
    *   Redis Sorted Set(ZSET) 우선순위 큐에서 명령을 가져와 Fleet 제어.
    *   AMR 액션 서버 통신 단절 시 1.0초 타임아웃 예외 처리 및 DB 상태 롤백 복구 엔진 탑재.
*   **인터페이스 토폴로지**:
    *   **Action Client**: `manage_workstation` (ManageWorkstation), `move_package` (MovePackage), `start_packaging` (StartPackaging)
    *   **Service Server**: `report_inbound_progress` (ReportInboundProgress), `check_warehouse_status` (CheckWarehouseStatus), `get_daily_package_list` (GetDailyPackageList)
    *   **Publisher**: `/{robot_id}/pause_status` (std_msgs/Bool) -> 로봇 암 인터로킹 일시정지 제어용
    *   **Publisher**: `/sim/sg2_workstation_trigger` (WorkstationSimTrigger) -> Isaac Sim 시뮬레이터 연동용

### 2️⃣ `sim_sync_node` (`sim_sync_node.py`)
*   **역할**: **분산 시뮬레이션 환경 브릿지**. 물리적으로 쪼개진 시뮬레이션 PC 사이에서 컨베이어 이동 중인 상자 소멸/소환 트리거링 처리.
*   **인터페이스 토폴로지**:
    *   **Service Server**: `transit_package` (TransitPackage)
    *   **Publisher**: `/sim/sg2_spawn_trigger` (std_msgs/String) -> Isaac Sim 상자 소환 트리거 발행

### 3️⃣ [가상 에뮬레이터] `mock_amr` (`mock_amr_node.py`)
*   **역할**: 실제 AMR 주행 장비나 Isaac Sim 없이도 로컬에서 4대 가상 AMR(`amr_01` ~ `amr_04`)을 에뮬레이션하여 관제탑 명령 주행 시나리오 모의 검증.
*   **구현 특징**: 1.5m 그리드 기반으로 실시간으로 위치(X, Y)를 계산하여 **Redis Hash (`amr:<name>`)에 30Hz 주기로 업데이트**함으로써 웹 대시보드상에서 AMR 아이콘 주행 렌더링 지원.
*   **인터페이스 토폴로지**:
    *   **Action Server**: `manage_workstation` (ManageWorkstation), `move_package` (MovePackage)
    *   **Subscriber**: `/fleet/task_events` (std_msgs/String) -> 관제탑이 매핑한 배정 태스크 정보 모니터링

### 4️⃣ [가상 에뮬레이터] `mock_sg2` (`mock_sg2_node.py`)
*   **역할**: 실제 협동 로봇 암 없이도 입고라인 로봇(`sg2_in_01`/`02`/`03`)의 상자 적재 및 출고라인 로봇(`sg2_out_00`)의 포장 동작을 멀티스레드로 모의 시뮬레이션.
*   **구현 특징**: 관제탑이 발행하는 `/{robot_id}/pause_status` 토픽을 실시간으로 반영하여 적재/포장 동작 중단 및 180도 회전/작업대 교체 완료 후 자동 재개 인터로킹 시나리오 수행.
*   **인터페이스 토폴로지**:
    *   **Action Server**: `start_packaging` (StartPackaging)
    *   **Service Client**: `check_warehouse_status` (CheckWarehouseStatus), `report_inbound_progress` (ReportInboundProgress)
    *   **Subscriber**: `/{robot_id}/pause_status` (std_msgs/Bool)

---

## 🗄️ 4. 백엔드 및 데이터 레이어 요구사항 (Data Layer)

합쳐진 워크스페이스가 정상 작동하려면 ROS 2 노드뿐만 아니라 **아래 데이터베이스 및 대시보드 구성 요소가 기동되어야 합니다.**

### ① PostgreSQL 15 데이터베이스 스키마
*   **`floor_qr_map`**: 창고 바닥 13.5m x 20m 크기의 1.5m 간격 격자 좌표(143개 노드) 및 주차 스팟/대기 영역 정의 테이블 (X, Y 좌표 Resolution 용도)
*   **`robots`**: AMR 기기명(`amr_01` ~ `amr_04`) 및 로봇 상태 관리 테이블
*   **`workstations`**: 작업대 상태(`EMPTY`, `WAITING`, `_A_ROTATING` 등) 및 현재 배치 좌표 테이블
*   **`packages`**: 개별 패키지 ID, 배송예정일, 로봇/작업대/슬롯 외래키 관계 테이블 (**1:N 정규화 스키마**)
*   **`warehouse_locations`**: 12개 보관/주차 스팟 상태 테이블

### ② Redis 7.0 캐시 스토리지
*   **`system:today_date`**: WMS 영업일 날짜 (FastAPI 및 관제탑 스케줄러 동기화 기준)
*   **`system:inbound_started`**: `true`/`false` (입고 라벨링 기동 플래그)
*   **`amr:<amr_name>`**: AMR 실시간 위치(`current_qr_id`), 배터리, 상태(`NAVIGATING`/`IDLE`)를 담고 있는 Hash 구조 (대시보드 실시간 브로드캐스트용)
*   **`fleet_tasks`**: Redis Sorted Set(ZSET) 자료구조. 관제탑이 우선순위 스코어(Score)와 UUID 기반으로 명령 큐 적재.

### ③ 웹 대시보드 서버 (`dashboard_server.py`)
*   FastAPI 기반 웹소켓 양방향 변경분 브로드캐스트 서버 (포트: `8009`).
*   HTML5/CSS absolute positioning 기법을 사용하여 DOM 연산 오버헤드 95% 감축.

---

## 🚀 5. 통합 후 원클릭 환경 구동 프로세스

워크스페이스 통합이 성공적으로 끝나면, 아래의 순서로 시스템을 검증할 수 있습니다.

### Step 1. 인프라 컨테이너 실행
```bash
docker compose -f docker/docker-compose.yml up -d
```

### Step 2. 데이터베이스 스키마 & Redis 데이터 초기화
```bash
python3 scratch/reset_db.py
```

### Step 3. 워크스페이스 빌드 및 환경 로드
```bash
colcon build --symlink-install
source install/setup.bash
export ROS_DOMAIN_ID=119
```

### Step 4. 개별 컴포넌트 터미널 실행
```bash
# 터미널 1: 웹 대시보드 서버 실행
python3 scratch/dashboard_server.py

# 터미널 2: 관제 코어 노드 구동
ros2 run cobot3 control_tower

# 터미널 3: 가상 로봇/AMR 에뮬레이터 구동
ros2 run cobot3 mock_amr
ros2 run cobot3 mock_sg2
```

---

## ❓ 6. 병합 시 주의 사항 및 질문 사항

> [!WARNING]
> **1. ROS_DOMAIN_ID 및 localhost 환경 변수 통일**
> 이종 PC 연동(예: Isaac Sim 구동 PC와 백엔드 관제 PC가 다를 경우) 시, `ROS_LOCALHOST_ONLY` 환경 변수를 반드시 `0`으로 세팅하고 `cyclonedds_wifi_config.xml` 파일을 로드해야 원활한 DDS 통신이 이뤄집니다.
>
> **2. PostgreSQL/Redis 포트 충돌 방지**
> 로컬 환경에 이미 구동 중인 Postgres(5432)나 Redis(6379) 데몬이 있다면 Docker 컨테이너 실행 시 충돌이 날 수 있습니다. 호스트 포트 바인딩 설정을 미리 조율해야 합니다.

### 💬 Handoff를 위한 질문 사항 (체크 필요)
1.  **합치게 될 원본 워크스페이스의 ROS 2 버전이 Humble이 맞습니까?** (버전이 다를 경우 interfaces 빌드 방식에 호환성 체크 필요)
2.  **데이터베이스 서버(PostgreSQL, Redis)를 단일 중앙 서버에 올리나요, 아니면 개별 로컬 환경에서 구동하나요?** (IP 주소 및 환경 변수 `POSTGRES_HOST`, `REDIS_HOST` 세팅 필수 조율)
3.  **상대방 측에서 로봇 팔(Manipulator)이나 AMR 구동 코드를 제공하나요?** 제공한다면 본 가이드라인의 `mock_amr_node`와 `mock_sg2_node`를 제거하고 그 자리에 실제 장비 제어 노드를 얹어 통신을 직접 바인딩해야 합니다.
