# 📑 지홍 담당 BG2 물류 분류 시스템 DEBUGGING 및 개발 이력 통합 보고서

> [!IMPORTANT]
> **문서 목적**: 본 문서는 협동3 프로젝트에서 지홍이 담당한 Isaac Sim BG2 물류 분류 시스템의 개발 과정, 구조 변경, 핵심 문제, 디버깅 과정, 최종 해결 방법을 기록한 문서입니다.  
> **최종 기준 버전**: `Step4_Stage1_01_finalfac1_two_robot_sort_v18.py`  
> **핵심 범위**: 컨베이어 기반 상자 공급, QR 인식, CSV `route_zone` 날짜 분류, BG2 이중 로봇 분류, Final Gate 도착 판정, ROS2 `TransitPackage` 서비스 연동, DDS 환경 정합성 확보

---

## 📅 Part 1. 세부 개발 변경 이력 (Timeline & Changelog)

### 📅 Step1. Isaac Sim 물리 환경 및 컨베이어 기본 검증

* **컨베이어벨트 동작 방식 분석**
  - Isaac Sim의 `Conveyor Belt Utility`와 `ConveyorNode`가 실제 물체 이동에 관여하는 구조를 파악했다.
  - 단순 텍스처 애니메이션과 실제 rigid body 접촉 기반 이동의 차이를 구분했다.
  - 상자가 공중에 떠 있거나 컨베이어 위에서 움직이지 않는 문제를 collision, rigid body, mass, spawn z-offset 관점에서 분석했다.

* **finalfac/finalfac1 USD 환경 정리**
  - 최종 통합용 USD 파일을 `finalfac1.usd` 기준으로 정리했다.
  - 실행 경로를 `/home/rokey/dev_ws/isaac_sim/isaac_step4` 기준으로 통일했다.
  - 다른 PC 이관을 고려해 `/home/jihongkim` 경로와 `/home/rokey` 경로 차이로 발생할 수 있는 실행 오류를 정리했다.

---

### 📅 Step2. BG2 로봇 제어 및 분류 모션 안정화

* **Robot1 / Robot2 역할 고정**
  - `robot1 = /World/FFW_BG2/ffw_bg2_follower/joints`
  - `robot2 = /World/FFW_BG2_01/ffw_bg2_follower/joints`
  - 두 로봇의 역할이 바뀌지 않도록 고정 매핑했다.

* **분류 시나리오 확정**
  - `today` 상자는 Robot1이 1차 분류 후 Robot2를 건너뛴다.
  - `day2`, `day3` 상자는 Robot1이 `not_today` 방향으로 보내고, Robot2가 2차 분류한다.
  - Robot2는 QR을 다시 읽지 않고 Robot1에서 판독한 target 정보를 그대로 사용한다.

* **초기 자세 충돌 문제 해결**
  - 초기에는 로봇 팔이 컨베이어 방향으로 쓸리듯 이동해 충돌 위험이 있었다.
  - 이를 해결하기 위해 초기 자세를 `init1 → init2 → init3` 단계로 나눴다.
  - 직접 보간으로 팔이 앞으로 튀어나오는 현상을 줄이고, 안정적인 standby pose를 확보했다.

* **drag-only full motion 구조 확립**
  - 중간에 상자 displacement를 보고 동작을 조기 종료하는 방식은 불안정했다.
  - 최종 구조에서는 로봇이 정해진 경유점을 끝까지 수행하고, 성공 여부는 Final Gate에서 판단하도록 분리했다.

---

### 📅 Step3. QR 인식 및 CSV 기반 날짜 분류 통합

* **QR 부착면 문제 확인**
  - box_assets USD에 QR이 이미 붙어 있었지만, 어느 local face인지 명확하지 않았다.
  - 테스트 결과 QR이 local `-Y` 면에 있음을 확인했다.
  - 스폰 시 `--box-qr-face-up y_neg`를 기본값으로 적용하여 QR이 위로 오도록 고정했다.

* **Robot1 QR 카메라 구성**
  - Robot1 작업대 상단에 QR 판독용 카메라 prim을 생성했다.
  - 기본값을 zoom-in 구조로 조정했다.
    - `focal_length=55`
    - `horizontal_aperture=18`
    - `resolution=1024,1024`
  - OpenCV `QRCodeDetector`로 실제 이미지 QR decode를 시도했다.

* **CSV route_zone 기반 분류 구조로 전환**
  - 기존 방식은 `QR_20260607_002` 같은 문자열에서 날짜를 직접 추출했다.
  - 하지만 최종 구조에서는 QR 문자열은 단순한 `qr_id`로 정의했다.
  - 실제 출고/분류 날짜는 CSV의 `route_zone` 컬럼에서 읽도록 변경했다.

* **Strict Spawn 적용**
  - v17에서는 CSV에 없는 QR도 legacy fallback으로 spawn될 수 있었다.
  - v18에서는 `package_csv`에 match된 `qr_id`만 catalog에 들어가도록 변경했다.
  - 결과적으로 작업 지시서에 없는 물류는 시뮬레이션에 공급되지 않는다.

---

### 📅 Step4. ROS2 팀 통합 및 최종 v18 완성

* **Final Gate 도착 판정 개선**
  - 초기에는 상자 중심점이 목적지 영역 안에 들어와야만 도착으로 인정했다.
  - 실제 물류 흐름에서는 상자가 영역에 걸치거나 통과해도 도착으로 봐야 하므로 bbox overlap 방식으로 변경했다.
  - `--final-gate-max-wait-sec 1.0`을 적용하여 다음 상자 공급이 과도하게 지연되지 않게 했다.

* **Despawn 및 분류 완료 로그 구현**
  - Final Gate 도착 후 3초 대기한다.
  - `stage.RemovePrim(path)`로 상자를 hard delete한다.
  - `[ARRIVED BOX LOG]`, `[ARRIVED BOX DESPAWN]`, `[CYCLE DONE]` 로그로 사이클을 추적한다.

* **TransitPackage 서비스 연동**
  - `/sim/transit_package` 서비스로 후속 공정에 상자 이송 이벤트를 전달한다.
  - target별 매핑은 다음과 같다.

| target | target_line |
| :--- | :--- |
| `today` | `sg2_in_01` |
| `day2` | `sg2_in_02` |
| `day3` | `sg2_in_03` |

* **Isaac Python / ROS Humble ABI 문제 해결**
  - Isaac Sim Python은 3.11이고 ROS Humble rclpy는 3.10 빌드여서 직접 import가 실패했다.
  - Isaac 내부에서 `rclpy.create_client()`를 직접 사용하는 구조를 폐기했다.
  - 외부 `/usr/bin/python3` helper를 실행하고, helper가 rclpy client를 생성하는 구조로 변경했다.

* **DDS 환경 정합성 확보**
  - 수동 터미널에서는 `/sim/transit_package`가 보이지만 helper에서는 보이지 않는 문제가 있었다.
  - 원인은 `RMW_IMPLEMENTATION`, `CYCLONEDDS_URI`가 helper subprocess로 전달되지 않았기 때문이었다.
  - v16 이후 helper subprocess에 다음 환경변수를 명시적으로 전달했다.

```bash
ROS_DOMAIN_ID=119
ROS_LOCALHOST_ONLY=0
RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml
```

---

## ⚠️ Part 2. 핵심 문제점 및 해결 방법

### 1. QR이 보이는 면이 틀어지는 문제

#### 문제
상자 asset에 QR이 이미 포함되어 있었지만, 스폰 후 QR이 카메라를 향하지 않거나 위로 오지 않아 실제 decode가 실패했다.

#### 원인
상자 USD의 QR 부착면이 local `-Y` 면인데, 기본 스폰 자세에서는 이 면이 위를 향하지 않았다.

#### 해결
`QR_FACE_UP_QUAT_WXYZ` 매핑을 정의하고, `y_neg`를 기본 face-up 값으로 적용했다.

```text
--box-qr-face-up y_neg
```

#### 결과
Robot1 overhead camera에서 QR이 시야에 들어오고, 실제 OpenCV decode 성공률을 확보했다.

---

### 2. 로봇 팔이 컨베이어와 충돌할 위험

#### 문제
로봇 초기 자세에서 분류 자세로 바로 이동하면 팔이 컨베이어 벨트 쪽으로 크게 휘어 나오며 충돌 위험이 있었다.

#### 원인
초기 pose와 작업 pose 사이의 직접 보간 경로가 물리적으로 안전하지 않았다.

#### 해결
초기 자세를 3단계로 분리했다.

```text
init1 -> init2 -> init3
```

그리고 분류 동작은 drag-only full phase 방식으로 정리했다.

#### 결과
로봇 팔이 컨베이어를 치는 위험을 줄이고, 반복 실행 가능한 분류 모션을 확보했다.

---

### 3. 최종 도착 판정이 너무 늦어지는 문제

#### 문제
상자가 목적지 라인을 지나가고 있는데도 Final Gate 판정이 되지 않아 다음 상자 spawn이 지연됐다.

#### 원인
기존 판정은 상자 bbox 중심점이 gate bbox 내부에 들어와야만 성공이었다.

#### 해결
Final Gate 판정을 `center` 기준에서 `overlap` 기준으로 변경했다.

```bash
--final-gate-hit-mode overlap
--final-gate-max-wait-sec 1.0
```

#### 결과
상자가 최종 라인에 걸치거나 통과하면 빠르게 도착 처리되고, 다음 사이클이 지연되지 않았다.

---

### 4. Isaac 내부 rclpy import 실패

#### 문제 로그

```text
ModuleNotFoundError: No module named 'rclpy._rclpy_pybind11'
The C extension ... cpython-311 ... isn't present on the system
```

#### 원인
Isaac Sim Python은 3.11이고, ROS Humble rclpy는 시스템 Python 3.10 기준으로 빌드되어 있었다. 따라서 Isaac 내부에서 rclpy C extension을 로드할 수 없었다.

#### 실패한 접근
- Isaac 코드에서 직접 `import rclpy`
- Isaac 내부에서 `node.create_client()`
- `ros2 service call` subprocess

#### 해결
외부 helper 방식으로 전환했다.

```text
Isaac Python
  -> subprocess
    -> /usr/bin/python3 transit_package_client_helper.py
      -> rclpy.create_client(TransitPackage, "/sim/transit_package")
```

#### 결과
Isaac Python ABI를 피하면서도 실제 ROS2 서비스 client를 사용할 수 있게 되었다.

---

### 5. helper에서 `/sim/transit_package` 서비스가 보이지 않는 문제

#### 문제 로그

```text
[HELPER WAIT FAIL] service=/sim/transit_package timeout=5.0s
```

#### 원인
수동 터미널에서는 다음 환경이 잡혀 있었지만, helper subprocess에는 일부 환경변수가 전달되지 않았다.

```text
RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml
```

DDS discovery는 domain만 같다고 되는 것이 아니라, RMW와 CycloneDDS 설정 파일까지 맞아야 했다.

#### 해결
v16에서 helper subprocess 환경에 다음 값을 전달하도록 수정했다.

```python
"RMW_IMPLEMENTATION": rmw_impl,
"CYCLONEDDS_URI": cyclonedds_uri,
```

실행 시에도 다음을 명시한다.

```bash
--transit-rmw-implementation rmw_cyclonedds_cpp
--transit-cyclonedds-uri file:///home/rokey/.ros/cyclonedds_wifi.xml
```

#### 결과
helper가 `/sim/transit_package` 서비스를 정상 discovery하고, package transit 요청을 보낼 수 있게 되었다.

---

### 6. QR 문자열을 날짜로 오해한 문제

#### 문제
`QR_20260607_002` 같은 문자열을 보고 `2026-06-07`을 출고일처럼 해석하면, 실제 물류 CSV와 분류 기준이 어긋난다.

#### 원인
QR은 물류 상자의 식별자일 뿐, 실제 출고일은 별도의 CSV `route_zone`이 결정해야 한다.

#### 해결
v17에서 CSV route DB를 추가했다.

```python
qr_id -> CSV row -> route_zone -> target
```

분류 기준은 다음과 같다.

| 기준일 | route_zone | target |
| :--- | :--- | :--- |
| 2026-06-08 | 2026-06-08 | today |
| 2026-06-08 | 2026-06-09 | day2 |
| 2026-06-08 | 2026-06-10 | day3 |

#### 결과
QR은 식별자로만 사용하고, 실제 분류는 작업 지시 데이터인 CSV 기준으로 수행한다.

---

### 7. CSV에 없는 상자가 spawn되는 문제

#### 문제
v17에서는 CSV에 match되지 않은 QR도 legacy fallback으로 target이 정해져 spawn될 수 있었다.

#### 원인
`parse_package_asset_name()`에서 CSV miss를 허용하고, QR 내부 날짜 fallback으로 catalog에 추가하는 구조가 남아 있었다.

#### 해결
v18에서 strict spawn을 적용했다.

```python
if strict_package_csv_spawn and not csv_rec:
    return None
```

#### 결과
`package_csv`에 존재하는 `qr_id`만 spawn 가능하다. 즉 작업 지시서 없는 상자는 공급되지 않는다.

---

## 🛠️ Part 3. 실행 및 점검 명령어

### 1. 서비스 서버 확인

```bash
source /opt/ros/humble/setup.bash
source /home/rokey/cobot3_ws/install/setup.bash

export ROS_DOMAIN_ID=119
export ROS_LOCALHOST_ONLY=0
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml

ros2 service list | grep transit
ros2 service type /sim/transit_package
```

정상 출력:

```text
/sim/transit_package
cobot3_interfaces/srv/TransitPackage
```

### 2. helper 단독 테스트

```bash
cd /home/rokey/dev_ws/isaac_sim/isaac_step4

/usr/bin/python3 transit_package_client_helper.py \
  --service /sim/transit_package \
  --package-id PKG_HELPER_TEST_001 \
  --target-line sg2_in_01 \
  --domain-id 119 \
  --localhost-only 0 \
  --timeout-sec 5.0
```

정상 출력:

```text
[HELPER ENV] ROS_DOMAIN_ID=119 ROS_LOCALHOST_ONLY=0 RMW_IMPLEMENTATION=rmw_cyclonedds_cpp CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml service=/sim/transit_package
[HELPER REQUEST] package_id=PKG_HELPER_TEST_001 target_line=sg2_in_01
[HELPER RESPONSE] success=True message=...
```

### 3. Isaac 실행

```bash
cd /home/rokey/dev_ws/isaac_sim/isaacsim/_build/linux-x86_64/release

ROS_DOMAIN_ID=119 \
ROS_LOCALHOST_ONLY=0 \
RMW_IMPLEMENTATION=rmw_cyclonedds_cpp \
CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml \
./python.sh /home/rokey/dev_ws/isaac_sim/isaac_step4/Step4_Stage1_01_finalfac1_two_robot_sort_v18.py \
  --usd /home/rokey/dev_ws/isaac_sim/isaac_step4/finalfac1.usd \
  --max-boxes 60 \
  --spawn-interval 1 \
  --robot1-start-delay-after-spawn 3.0 \
  --stage2-delay-after-robot1 2.0 \
  --final-gate-hit-mode overlap \
  --final-gate-max-wait-sec 1.0 \
  --despawn-after-final-gate-sec 3.0 \
  --package-csv /home/rokey/dev_ws/isaac_sim/isaac_step4/packages_2026-06-08.csv \
  --today-date 2026-06-08 \
  --transit-service-enabled \
  --transit-service-name /sim/transit_package \
  --transit-ros-domain-id 119 \
  --transit-ros-localhost-only 0 \
  --transit-rmw-implementation rmw_cyclonedds_cpp \
  --transit-cyclonedds-uri file:///home/rokey/.ros/cyclonedds_wifi.xml \
  --transit-service-timeout-sec 5.0 \
  --robot1-camera-enabled \
  --qr-real-decode \
  --no-sort-start-trigger-visual
```

---

## 🔍 Part 4. 로그별 진단표

| 로그 | 의미 | 조치 |
| :--- | :--- | :--- |
| `[PACKAGE CSV][WARN] not found` | CSV 파일 경로가 없음 | `--package-csv` 경로 확인 |
| `[PACKAGE CSV ROUTE][MISS]` | QR이 CSV에 없음 | CSV에 `qr_id` 추가 또는 strict spawn 확인 |
| `[QR CATALOG] total_spawnable=0` | 스폰 가능한 상자가 없음 | CSV와 box_assets의 QR ID 일치 확인 |
| `[QR CAMERA DECODE][REAL FAIL]` | 카메라 이미지에서 QR decode 실패 | debug image 확인, 카메라 위치/조명 조정 |
| `[HELPER IMPORT FAIL]` | system Python에서 rclpy import 실패 | ROS setup source 및 cobot3_ws build 확인 |
| `[HELPER WAIT FAIL]` | 서비스 discovery 실패 | ROS_DOMAIN_ID, RMW, CYCLONEDDS_URI 확인 |
| `[TRANSIT PACKAGE HELPER RETURN] code=0` | 서비스 호출 성공 | 정상 |
| `[ARRIVED BOX DESPAWN] removed=1` | 상자 hard delete 성공 | 정상 |

---

## ✅ Part 5. 최종 검증 체크리스트

- [x] `finalfac1.usd` 로드 성공
- [x] Robot1 / Robot2 joint root 고정
- [x] QR box asset spawn 성공
- [x] QR face-up `y_neg` 적용
- [x] Robot1 camera 생성
- [x] OpenCV QR decode 또는 fallback 작동
- [x] CSV `qr_id` match 확인
- [x] `route_zone` 기준 target 계산
- [x] CSV miss 상자 spawn 차단
- [x] Robot1 today/not_today 분류
- [x] Robot2 day2/day3 분류
- [x] Final Gate overlap 도착 판정
- [x] 도착 후 3초 대기
- [x] `stage.RemovePrim()` despawn
- [x] `TransitPackage` helper 호출
- [x] ROS2 `/sim/transit_package` 서비스 성공 응답
- [x] `package_id`, `target_line` 후속 공정 전달

---

## 🧠 최종 결론

최종 v18은 단순 시뮬레이션 스크립트가 아니라, **실제 물류 작업 지시 데이터와 Isaac Sim 물리 분류 시스템을 연결하는 통합 물류 분류 파이프라인**이다.

개발 과정에서 가장 큰 난점은 세 가지였다.

1. 시뮬레이션 물리 세계에서 상자가 안정적으로 이동하고 로봇이 충돌 없이 분류하게 만드는 것
2. QR 문자열과 실제 물류 출고일을 분리하여 CSV 기반 데이터 판단 구조로 바꾸는 것
3. Isaac Sim과 ROS2 Humble 사이의 Python ABI 및 DDS discovery 문제를 우회해 팀 통합 서비스까지 연결하는 것

이 문제들을 해결한 결과, v18에서는 다음 흐름이 안정적으로 완성되었다.

```text
CSV match 상자 spawn
-> QR decode
-> route_zone target 계산
-> Robot1/Robot2 물리 분류
-> Final Gate 도착 확인
-> Despawn
-> TransitPackage 서비스 보고
```

따라서 본 파트는 협동3 전체 시스템에서 **입고 물류를 날짜별 보관 라인으로 나누고, 후속 부서에 물류 분류 상태를 전달하는 자동 분류 게이트웨이**로 기능한다.
