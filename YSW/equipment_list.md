# equipment_list.md

# 협동3 AMR Fleet 사용 장비 및 실행 환경

## 1. 물리 장비

| 구분 | 장비 | 용도 |
|---|---|---|
| 메인 PC | IsaacSim 실행 PC | AMR / 창고 / 작업대 시뮬레이션 실행 |
| 상대 PC | Control Tower 또는 명령 송신 PC | ROS2 Action goal 송신 |
| 네트워크 | Wi-Fi | ROS2 DDS 통신 |
| GPU | NVIDIA GPU 권장 | IsaacSim 렌더링 및 QR 전처리 가속 |

---

## 2. 시뮬레이션 장비

| 구분 | 수량 | 설명 |
|---|---:|---|
| AMR | 5대 | `AMR_01` ~ `AMR_05` |
| 작업대 / Workstation | 10개 이상 | WS01 ~ WS10 |
| SG 입력 슬롯 | 3개 | `sg2_in_01_B`, `sg2_in_02_B`, `sg2_in_03_B` |
| SG 출력 슬롯 | 2개 이상 | `sg2_out_00_A`, `sg2_out_00_B` |
| QR Marker | 다수 | cell 위치 인식 |
| Downward Camera | AMR별 1개 | QR 인식용 |

---

## 3. AMR 초기 위치

Grid 기준:

| AMR | Cell | World 좌표 |
|---|---:|---:|
| AMR_01 | `(-4, -6)` | `(-6.0, -9.0)` |
| AMR_02 | `(-4, -5)` | `(-6.0, -7.5)` |
| AMR_03 | `(-4, -4)` | `(-6.0, -6.0)` |
| AMR_04 | `(-4, -3)` | `(-6.0, -4.5)` |
| AMR_05 | `(-4, -2)` | `(-6.0, -3.0)` |

변환 기준:

```text
world_x = cell_x * 1.5
world_y = cell_y * 1.5
```

---

## 4. 주요 Workstation 위치

| Workstation | Cell | World 좌표 |
|---|---:|---:|
| WS01 | `(3, 6)` | `(4.5, 9.0)` |
| WS02 | `(5, 1)` | `(7.5, 1.5)` |
| WS03 | `(5, -2)` | `(7.5, -3.0)` |
| WS04 | `(5, -5)` | `(7.5, -7.5)` |
| WS05 | `(-1, -2)` | `(-1.5, -3.0)` |
| WS06 | `(-2, -2)` | `(-3.0, -3.0)` |
| WS07 | `(-1, 0)` | `(-1.5, 0.0)` |
| WS08 | `(-2, 0)` | `(-3.0, 0.0)` |
| WS09 | `(-1, 2)` | `(-1.5, 3.0)` |
| WS10 | `(-2, 2)` | `(-3.0, 3.0)` |

---

## 5. 주요 SG Target 위치

| Target | World 좌표 | Cell |
|---|---:|---:|
| `sg2_in_01_B` | `(6.0, 1.5)` | `(4, 1)` |
| `sg2_in_02_B` | `(6.0, -3.0)` | `(4, -2)` |
| `sg2_in_03_B` | `(6.0, -7.5)` | `(4, -5)` |
| `sg2_out_00_A` | `(-4.5, 9.0)` | `(-3, 6)` |
| `sg2_out_00_B` | `(-4.5, 7.5)` | `(-3, 5)` |

---

## 6. 소프트웨어 환경

| 항목 | 값 |
|---|---|
| OS | Ubuntu 22.04 |
| ROS | ROS2 Humble |
| Python | Python 3.10 |
| DDS | CycloneDDS |
| RMW | `rmw_cyclonedds_cpp` |
| ROS Domain | `119` |
| IsaacSim | AMR / 창고 시뮬레이션 |
| OpenCV | QR 인식 및 전처리 |
| Redis | 상태 발행 선택 기능 |

---

## 7. 네트워크 구성

기준 구성:

```text
메인 PC:
192.168.10.40

상대 PC:
192.168.10.19

인터페이스:
wlp128s20f3

ROS_DOMAIN_ID:
119
```

양쪽 PC 모두 같은 값을 사용해야 한다.

```bash
export ROS_DOMAIN_ID=119
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI=file:///home/rokey/.ros/cyclonedds_wifi.xml
```

---

## 8. 실행 파일 목록

| 파일 | 위치 | 설명 |
|---|---|---|
| `amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py` | `~/isaaclab_ws/isaac_aruco/amr/` | 최종 Controller |
| `fleet_manager_bridge_node_gpu_v43_guarded_actions.py` | `~/isaaclab_ws/isaac_aruco/amr/` | 최종 Bridge |
| `run_bridge_gpu.sh` | `~/isaaclab_ws/isaac_aruco/amr/` | Bridge 실행 스크립트 |
| `cyclonedds_wifi.xml` | `~/.ros/` | Wi-Fi ROS2 통신 설정 |
| `bridge_queue/` | `~/isaaclab_ws/isaac_aruco/amr/` | Bridge/Controller 파일 queue |

---

## 9. 역할별 장비 사용

| 역할 | 사용하는 장비 |
|---|---|
| IsaacSim 시뮬레이션 | 메인 PC |
| AMR 제어 코드 실행 | 메인 PC IsaacSim Script Editor |
| ROS2 Action Bridge | 메인 PC 터미널 |
| 명령 송신 / Control Tower | 상대 PC |
| 상태 모니터링 | 메인 PC 또는 상대 PC |
| 발표 시연 | 메인 PC 화면 녹화 또는 GIF 자료 |
