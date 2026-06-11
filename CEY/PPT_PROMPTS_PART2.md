---

## Slide 9 — ReplayController 상태머신

Create a clean slide with white background.

Header strip:
- Title: "ReplayController 상태머신"
- Subtitle: "3대 로봇 독립 병렬 운영 | sh5_bringup_ros2_3robot.py"

Layout: Center area shows state machine flow diagram.

Flow diagram (left to right with arrows):
[IDLE] → [SCANNING] → [WAITING_DB] → [REPLAYING] → [HOMING] → [DONE] → back to [IDLE]

Below each state, small description boxes:
- IDLE: 트리거 대기
- SCANNING: QR 코드 인식 (TopView 카메라)
- WAITING_DB: check_warehouse_status 서비스 응답 대기
- REPLAYING: HDF5 궤적 주입 + HDF5-Guided Snapping
- HOMING: stay.hdf5 안전 자세 복귀
- DONE: report_inbound_progress 보고 후 IDLE

Bottom note:
- "3대 ReplayController 인스턴스가 sg2_in_01/02/03 라인에서 완전 독립 병렬 운영"

---

## Slide 10 — HDF5-Guided Snapping (파지 안정화)

Create a clean slide with white background.

Header strip:
- Title: "HDF5-Guided Snapping 파지 안정화"
- Subtitle: "왼/오른손 자동 선택 + 딜레이 없는 즉시 부착"

Layout: 2 columns.

Left column (50%):
- Header: "기존 문제"
- Red bullet points:
  * 슬롯 2(왼손)에 오른손으로 텔레포트 오류
  * 손 닫힌 후 상자가 딜레이되어 나중에 안착
  * 빠른 이동 시 상자가 공중에 떠서 뒤처짐

- Header: "해결 방법"
- Green bullet points:
  * HDF5 box_trajectory 기준 가장 가까운 링크 자동 선택
  * ATTACH_FACTOR=1.0 → 링크 중심에 즉시 부착
  * GRASP_DIST=0.30m, MAX_BOX_STEP=3.0 (즉시 반응)
  * FINGER_OPEN_THRESH=0.80 → 자연스러운 릴리즈

Right column (50%):
- Parameter table:
  | 파라미터 | 값 | 효과 |
  |---|---|---|
  | ATTACH_FACTOR | 1.0 | 완전 부착 |
  | GRASP_DIST | 0.30m | 빠른 감지 |
  | MAX_BOX_STEP | 3.0 | 즉시 반응 |
  | FINGER_OPEN_THRESH | 0.80 | 자연 릴리즈 |

- Image placeholder below table:
  [파지 로직 플로우차트: 손가락 상태 → 링크 선택 → 부착/릴리즈]

---

## Slide 11 — yo-yo 현상 원인 분석 및 수정

Create a clean slide with white background.

Header strip:
- Title: "yo-yo 현상 완전 해결"
- Subtitle: "kinematic 물리 충돌 원인 규명 및 _write_box_pose() 도입"

Layout: Full width, 3 sections top-to-bottom.

Section 1 — 증상 (red background box):
"상자가 Z축으로 진동하며 손에서 일정 offset 유지, PhysX 에러 1000개 누적 → 시뮬 강제 종료"

Section 2 — 원인 분석 (yellow background, 2 columns):
Left:
- 원인 1: 스폰 시 kinematic=False 설정
  → 중력 ↓ vs write_root_state ↑ 싸움
Right:
- 원인 2: write_root_state_to_sim이 내부적으로
  setLinearVelocity/setAngularVelocity 호출
  → kinematic 바디에서 PhysX 에러

Section 3 — 해결 (green background, 2 columns):
Left:
- 해결 1: kinematic=False 코드 완전 제거
  → 항상 kinematic=True 유지
Right:
- 해결 2: _write_box_pose() 헬퍼 구현
  → write_root_pose_to_sim (velocity 없음)
  → 없으면 USD XFormable 직접 쓰기

---

## Slide 12 — WorkstationManager Spawn/Despawn

Create a clean slide with white background.

Header strip:
- Title: "작업대 실시간 Spawn/Despawn"
- Subtitle: "WorkstationManager | /tmp/sh5_ws_trigger.jsonl"

Layout: 2 columns.

Left column (45%):
- Header: "시나리오"
- Bullets:
  * AMR이 작업대 회수 시작 → 즉시 DESPAWN
  * AMR이 새 작업대 안착 완료 → SPAWN
  * 파일큐 폴링 방식 (실시간 반응)

- Header: "메시지 형식"
- Code block:
  {"workstation_id": "WS02",
   "location": "sg2_in_01_A",
   "action": "DESPAWN"}

- Header: "RACK 매핑"
- Small table:
  | location | RACK |
  |---|---|
  | sg2_out | RACK_01 |
  | sg2_in_01_A | RACK_02 |
  | sg2_in_02_A | RACK_03 |
  | sg2_in_03_A | RACK_04 |

Right column (55%): Image placeholder.
  [Isaac Sim 화면: RACK_02가 사라지는 DESPAWN 장면 / 다시 나타나는 SPAWN 장면]

---

## Slide 13 — ROS 2 브릿지 연동

Create a clean slide with white background.

Header strip:
- Title: "ROS 2 브릿지 연동"
- Subtitle: "ros2_sh5_bridge.py | 관제탑 ↔ Isaac Sim"

Layout: Center flow diagram.

Flow (top to bottom):
[관제탑 WMS] 
  ↓ /sim/sg2_spawn_trigger (ROS2 Topic)
[ros2_sh5_bridge.py]
  ↓ check_warehouse_status (Service)  →  DB 중복 확인
  ↓ /tmp/sh5_queue.jsonl (파일큐)
[sh5_bringup_ros2_3robot.py]
  ↓ 작업 완료
  ↓ /tmp/sh5_report_req.jsonl
[ros2_sh5_bridge.py]
  ↓ report_inbound_progress (Service)
[관제탑 WMS]

Right side: File queue table:
| 파일 | 방향 |
|---|---|
| sh5_queue.jsonl | Bridge→Isaac |
| sh5_qr_req.jsonl | Isaac→Bridge |
| sh5_pause.json | Bridge→Isaac |
| sh5_ws_trigger.jsonl | Bridge→Isaac |

---

## Slide 14 — SlotRegistry 고객 슬롯 관리

Create a clean slide with white background.

Header strip:
- Title: "SlotRegistry — 고객별 슬롯 유지 할당"
- Subtitle: "동일 고객 → 항상 동일 슬롯 보장"

Layout: 2 columns.

Left column (50%):
- Header: "동작 원리"
- Bullets:
  * 고객 ID 기반 슬롯 1~4 자동 배정
  * 재방문 시 동일 슬롯 재할당
  * 슬롯 4칸 만석 시 AMR에 작업대 교체 명령

- Header: "슬롯 구성"
- Table:
  | 슬롯 | 위치 | 손 |
  |---|---|---|
  | 1 | 앞면 우측 상단 | 오른손 |
  | 2 | 앞면 좌측 하단 | 왼손 |
  | 3 | 뒷면 우측 상단 | 오른손 |
  | 4 | 뒷면 좌측 하단 | 왼손 |

Right column (50%): Image placeholder.
  [작업대 슬롯 1~4 위치 다이어그램]

---

## Slide 15 — 3대 로봇 병렬 시연

Create a clean slide with white background.

Header strip:
- Title: "3대 로봇 동시 병렬 시연"
- Subtitle: "sg2_in_01 / 02 / 03 완전 독립 운영"

Layout: 3 equal columns.

Column 1 (sg2_in_01):
- Header: "라인 1"
- Robot position: (7.5, 3.0, -0.18)
- Icon: robot
- Status flow: IDLE → REPLAYING

Column 2 (sg2_in_02):
- Header: "라인 2"
- Robot position: (7.5, -1.5, -0.18)
- Icon: robot
- Status flow: SCANNING → WAITING_DB

Column 3 (sg2_in_03):
- Header: "라인 3"
- Robot position: (7.5, -6.0, -0.18)
- Icon: robot
- Status flow: HOMING → DONE

Bottom note:
"각 ReplayController 인스턴스가 독립 상태머신으로 동작 — 한 라인 대기 중에도 다른 라인 계속 작업"

---

## Slide 16 — 핵심 파라미터 레퍼런스

Create a clean slide with white background.

Header strip:
- Title: "핵심 파라미터 레퍼런스"
- Subtitle: "sh5_bringup_ros2_3robot.py 전역 상수"

Layout: 2 columns, parameter tables.

Left column:
- Header: "재생 제어"
- Table:
  | 파라미터 | 값 | 설명 |
  |---|---|---|
  | PLAYBACK_SPEED | 2 | 2배속 재생 |
  | WARMUP_FRAMES | 30 | 보간 프레임 (~1초) |
  | HOMING_FRAMES | 120 | 호밍 보간 (~4초) |
  | SKIP_FRAMES | 1 | 프레임 스킵 |

Right column:
- Header: "파지 제어"
- Table:
  | 파라미터 | 값 | 설명 |
  |---|---|---|
  | ATTACH_FACTOR | 1.0 | 완전 부착 |
  | GRASP_DIST | 0.30m | 스냅 거리 |
  | FINGER_OPEN_THRESH | 0.80rad | 열림 판정 |
  | MAX_BOX_STEP | 3.0 | 속도 클램프 |

---

## Slide 17 — 개발 이력 타임라인

Create a clean slide with white background.

Header strip:
- Title: "2주간 개발 이력"
- Subtitle: "2026.05.26 ~ 2026.06.12"

Layout: Vertical timeline (center line with left/right events).

Timeline events (chronological):
05/26 — Isaac Sim + SH5 환경 구축, VR 조작 인터페이스
05/28 — HDF5 녹화 파이프라인 + 카메라 통합
06/01 — 슬롯 1~4 총 400+ 에피소드 수집 완료
06/03 — freeze_idle_arms + create_subset 전처리
06/04 — augment_data 증강 파이프라인
06/06 — ACT v2 학습 (Colab A100, 150 epoch)
06/08 — 3대 로봇 ReplayController + ROS 2 브릿지
06/10 — WARMUP 보간 + stay.hdf5 호밍 + frozen_set
06/11 — HDF5-Guided Snapping 파지 안정화
06/12 — yo-yo 수정 + WorkstationManager 완성

---

## Slide 18 — 핵심 전환점 7가지

Create a clean slide with white background.

Header strip:
- Title: "핵심 기술 전환점 7가지"
- Subtitle: "프로젝트를 완성으로 이끈 결정적 순간들"

Layout: 7 numbered cards in grid (4 top + 3 bottom).

Card 1: "HDF5 녹화 파이프라인" — joint/box/image 완전 저장
Card 2: "방해 팔 전처리" — freeze_idle_arms로 학습 품질 개선
Card 3: "3대 로봇 병렬 시연" — ReplayController 독립 운영
Card 4: "워밍업 보간" — 텔레포트 제거, 자연스러운 시작
Card 5: "HDF5-Guided Snapping" — 왼/오른손 자동 선택
Card 6: "yo-yo 현상 해결" — kinematic + _write_box_pose
Card 7: "Spawn/Despawn" — WorkstationManager 실시간 연동

---

## Slide 19 — 알려진 이슈 및 해결 현황

Create a clean slide with white background.

Header strip:
- Title: "알려진 이슈 및 해결 현황"
- Subtitle: "기술적 도전과 대응 결과"

Layout: Table with status indicators.

Table (full width):
| 이슈 | 원인 | 해결 방법 | 상태 |
|---|---|---|---|
| kinematic 박스 velocity 오류 | write_root_state_to_sim | _write_box_pose() 헬퍼 | ✅ 해결 |
| 첫 프레임 텔레포트 | HDF5 재생 시작 | WARMUP_FRAMES=30 | ✅ 해결 |
| 비동작 팔 방해 | 키보드 조작 | frozen_set 사용 | ✅ 해결 |
| 복귀 모션 쓰러짐 | 마지막 손 동작 | stay.hdf5 호밍 | ✅ 해결 |
| 왼/오른손 선택 오류 | 수동 매핑 | HDF5-Guided Snapping | ✅ 해결 |
| GPU PhysX 불안정 | RTX 5080 Blackwell | CPU 모드 우회 | ⚠️ 우회 |
| RACK prim 경로 탐색 | USD 구조 | _find_prim 디버그 | 🔄 진행 |

---

## Slide 20 — 데이터셋 구성 현황

Create a clean slide with white background.

Header strip:
- Title: "데이터셋 구성 현황"
- Subtitle: "수집 → 전처리 → 증강 → 최종 학습셋"

Layout: 2 columns.

Left column (bar chart area):
- Header: "에피소드 수 현황"
- Bar chart placeholder:
  원본 수집: 400 episodes
  방해 팔 제거: 380 episodes
  증강 후: 760+ episodes
  frozen_set (최종): 400 episodes

Right column (table):
- Header: "슬롯별 분포"
- Table:
  | 슬롯 | 위치 | 원본 | 증강 후 |
  |---|---|---|---|
  | 1 | 앞 우측 상단 | 100 | 200 |
  | 2 | 앞 좌측 하단 | 100 | 200 |
  | 3 | 뒤 우측 상단 | 100 | 200 |
  | 4 | 뒤 좌측 하단 | 100 | 200 |

Bottom note: "증강 방법: 좌우 미러링(slot3→4) + 관절 가우시안 노이즈(σ=0.01rad)"

---

## Slide 21 — 2~3분 발표용 핵심 요약 (1)

Create a clean slide with dark navy background (#1A237E), white text.

Large centered text layout.

Top section (white):
"이번 프로젝트에서 해결한 3가지 핵심 문제"

3 large cards (horizontal):

Card 1 (blue):
- Number: "01"
- Title: "자연스러운 재생"
- Content: "텔레포트 → WARMUP 보간\n방해 팔 → frozen_set\n쓰러짐 → stay 호밍"

Card 2 (teal):
- Number: "02"
- Title: "안정적인 파지"
- Content: "HDF5-Guided Snapping\nyo-yo 현상 완전 제거\nkinematic 물리 수정"

Card 3 (green):
- Number: "03"
- Title: "실시간 연동"
- Content: "3대 로봇 병렬 시연\nROS 2 브릿지\nSpawn/Despawn"

---

## Slide 22 — 2~3분 발표용 핵심 요약 (2)

Create a clean slide with white background.

Header strip:
- Title: "2~3분 발표 핵심 스크립트"
- Subtitle: "발표자 참고용"

Layout: Script format, numbered points.

1. (30초) 프로젝트 소개
   "Isaac Sim 기반 SH5 쌍팔 로봇 3대가 물류 창고에서 동시에 Pick & Place를 수행하는 시스템입니다."

2. (40초) 데이터 파이프라인
   "VR 조작으로 400여 개 에피소드를 수집하고, 방해 팔 제거 전처리와 데이터 증강을 거쳐 Vision-ACT 모델을 학습했습니다."

3. (40초) 핵심 기술 — yo-yo 수정
   "가장 큰 기술적 도전은 yo-yo 현상이었습니다. kinematic 박스에 velocity를 설정하면 PhysX 에러가 1000개 누적되어 시뮬레이션이 강제 종료됩니다. write_root_pose_to_sim 전용 헬퍼를 만들어 완전히 해결했습니다."

4. (30초) 최종 시연
   "3대 로봇이 독립적으로 상태머신을 운영하며, AMR 이동 시 작업대가 실시간으로 사라지고 나타나는 Spawn/Despawn까지 구현했습니다."

---

## Slide 23 — USD 에셋 구성

Create a clean slide with white background.

Header strip:
- Title: "USD 에셋 구성"
- Subtitle: "finalfac.usd | RACK.usd | box_assets"

Layout: 3 cards horizontal.

Card 1:
- Icon: factory
- Title: "finalfac.usd (47MB)"
- Content:
  * 물류 창고 전체 씬
  * 컨베이어 벨트 3개 라인
  * RACK_01~10 작업대
  * AMR 경로 및 마킹

Card 2:
- Icon: shelf
- Title: "RACK.usd"
- Content:
  * 작업대 단일 모델
  * SPAWN 시 신규 생성용
  * WorkstationManager에서 사용

Card 3:
- Icon: box
- Title: "PKG_*.usd (140개)"
- Content:
  * QR 코드 부착 택배 상자
  * 날짜별 패키지 ID 관리
  * 6월 6일~12일 분

---

## Slide 24 — 최종 산출물 목록

Create a clean slide with white background.

Header strip:
- Title: "최종 산출물 목록"
- Subtitle: "github.com/sungwung1201/SNFC/tree/main/CEY"

Layout: 2 columns, categorized file list.

Left column:
- Category "🚀 시연": 
  * sh5_bringup_ros2_3robot.py ★
  * ros2_sh5_bridge.py ★
  * test_trigger.sh
- Category "📦 데이터 수집":
  * coupang_sh5_bringup_v.py ★
- Category "🔧 전처리":
  * freeze_idle_arms.py ★
  * create_subset.py ★
  * augment_data.py ★
  * filter_dataset.py

Right column:
- Category "🧠 학습":
  * train_act_v2.py ★
  * evaluate_test_vision.py
  * hdf5_replay_player.py ★
- Category "📡 USD 에셋":
  * assets/scene/finalfac.usd
  * assets/scene/RACK.usd
  * assets/box_assets/PKG_*.usd
- Category "📄 문서":
  * README.md (타임라인/코드설명)
  * DEBUGGING.md (개발일지)

---

## Slide 25 — 마무리 및 향후 계획

Create a clean slide with dark navy background (#1A237E), white text.

Layout: 2 sections.

Top section (60% height):
- Large centered text: "🎯 달성 성과"
- 3 achievement boxes side by side:
  Box 1: "400+ 에피소드 수집 완료"
  Box 2: "3대 로봇 동시 시연 성공"
  Box 3: "yo-yo 현상 완전 해결"

Bottom section (40% height, slightly lighter navy):
- Header: "향후 계획"
- Bullets (white):
  * Vision-ACT 모델 추론 시연으로 전환
  * RACK prim 경로 매핑 완성 및 Despawn 검증
  * GPU PhysX 패치 후 GPU 모드 전환
  * 실제 로봇 Sim2Real 적용 검토
