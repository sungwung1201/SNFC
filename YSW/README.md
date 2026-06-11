# 📋 성웅 담당 작업 타임라인 및 기여 정리 - 협동3

> **기간**: 2026년 5월 29일 ~ 2026년 6월 12일
> **발표일**: 2026년 6월 12일
> **역할**: 팀장 / 프로젝트 기획 / 전체 시나리오 설계 / 팀원 조율 / GitHub README 문서 작업 / PPT 제작 / 발표 준비 및 발표 / ROS2·IsaacSim 연동 디버깅 / AMR 주행 로직 개선 / Global Arbiter 기반 충돌 회피 구조 분석 / ROS2 Bridge 안정화 / 네트워크 통신 문제 해결
> **프로젝트 개요**: 협동3 프로젝트는 IsaacSim 환경에서 AMR 5대가 작업대를 픽업·운반·배치하는 물류 자동화 시뮬레이션이다. 외부 PC 또는 Control Tower에서 ROS2 Action 명령을 내리면, bridge가 이를 JSON command로 변환하고, IsaacSim controller가 해당 명령을 읽어 AMR 주행, 작업대 운반, 상태 반환을 수행하는 구조로 구현하였다.

---

<h2>성웅 담당 작업 타임라인 (프로젝트 기획 및 팀장 역할)</h2>

<table>
  <thead>
    <tr>
      <th nowrap>작업명</th>
      <th nowrap>날짜</th>
      <th nowrap>담당자</th>
      <th nowrap>파트</th>
      <th nowrap>단계</th>
      <th nowrap>완료 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr><td nowrap>협동3 프로젝트 전체 주제 기획</td><td nowrap>5월 29일</td><td nowrap>성웅</td><td nowrap>팀장/프로젝트 기획</td><td nowrap>기획</td><td nowrap>완료</td></tr>
    <tr><td nowrap>IsaacSim 기반 AMR Fleet 물류 자동화 시뮬레이션 방향 설정</td><td nowrap>5월 29일</td><td nowrap>성웅</td><td nowrap>팀장/시스템 기획</td><td nowrap>기획</td><td nowrap>완료</td></tr>
    <tr><td nowrap>실제 창고 자동화를 가정한 AMR 작업 시나리오 구상</td><td nowrap>5월 30일</td><td nowrap>성웅</td><td nowrap>팀장/시나리오 설계</td><td nowrap>운영 시나리오</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR 5대와 작업대 다수를 사용하는 물류 흐름 설계</td><td nowrap>5월 30일</td><td nowrap>성웅</td><td nowrap>팀장/물류 흐름 설계</td><td nowrap>프로세스 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>작업대 한 면 4칸, 두 면 8칸 기준 작업 완료 후 stage 또는 메인 창고로 이동하는 운영 방식 정리</td><td nowrap>5월 30일</td><td nowrap>성웅</td><td nowrap>팀장/작업 시나리오</td><td nowrap>비즈니스 시퀀스 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>팀원별 구현 방향과 역할 분담 조율</td><td nowrap>6월 1일</td><td nowrap>성웅</td><td nowrap>팀장/통합 관리</td><td nowrap>역할 분담</td><td nowrap>완료</td></tr>
    <tr><td nowrap>상대 PC와 내 PC 간 역할 분리 구조 설정</td><td nowrap>6월 1일</td><td nowrap>성웅</td><td nowrap>팀장/통신 구조 설계</td><td nowrap>연동 구조 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>내 PC는 IsaacSim 실행, 상대 PC는 ROS2 명령 송신을 담당하는 구조로 시스템 방향 정리</td><td nowrap>6월 1일</td><td nowrap>성웅</td><td nowrap>팀장/시스템 통합</td><td nowrap>운영 구조 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>전체 시스템 시나리오, 명령 흐름, 상태 반환 흐름 정리</td><td nowrap>6월 2일</td><td nowrap>성웅</td><td nowrap>팀장/시스템 설계</td><td nowrap>아키텍처 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>팀원별 개발 진행 상황 확인 및 구현 방향 조율</td><td nowrap>6월 3일</td><td nowrap>성웅</td><td nowrap>팀장/일정 관리</td><td nowrap>진행 관리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR 작업 흐름과 Control Tower 명령 구조를 연결하는 전체 방향 조율</td><td nowrap>6월 4일</td><td nowrap>성웅</td><td nowrap>팀장/통합 설계</td><td nowrap>통합 방향 설정</td><td nowrap>완료</td></tr>
    <tr><td nowrap>프로젝트 발표 흐름과 구현 시연 기준 정리</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>팀장/발표 조율</td><td nowrap>최종 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>최종 발표 전 전체 기능, 문서, PPT, 시연 흐름 점검</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>팀장/최종 점검</td><td nowrap>발표 준비</td><td nowrap>완료</td></tr>
    <tr><td nowrap>협동3 최종 발표 진행</td><td nowrap>6월 12일</td><td nowrap>성웅</td><td nowrap>팀장/최종 발표</td><td nowrap>발표</td><td nowrap>완료</td></tr>
  </tbody>
</table>

---

<h2>성웅 담당 작업 타임라인 (GitHub 문서화 및 PPT 제작)</h2>

<table>
  <thead>
    <tr>
      <th nowrap>작업명</th>
      <th nowrap>날짜</th>
      <th nowrap>담당자</th>
      <th nowrap>파트</th>
      <th nowrap>단계</th>
      <th nowrap>완료 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr><td nowrap>협동3 GitHub README 문서 구조 설계</td><td nowrap>6월 4일</td><td nowrap>성웅</td><td nowrap>GitHub/문서화</td><td nowrap>문서 구조 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>프로젝트 개요, 목표, 시스템 구성, 실행 방법 README 초안 정리</td><td nowrap>6월 4일</td><td nowrap>성웅</td><td nowrap>GitHub/README</td><td nowrap>문서 작성</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR Fleet 물류 자동화 시나리오 README 반영</td><td nowrap>6월 5일</td><td nowrap>성웅</td><td nowrap>GitHub/README</td><td nowrap>시나리오 문서화</td><td nowrap>완료</td></tr>
    <tr><td nowrap>ROS2 bridge와 IsaacSim controller 연동 구조 문서화</td><td nowrap>6월 6일</td><td nowrap>성웅</td><td nowrap>GitHub/기술 문서</td><td nowrap>연동 구조 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>bridge_queue의 commands/status/results/cancel/done 구조 README 반영</td><td nowrap>6월 6일</td><td nowrap>성웅</td><td nowrap>GitHub/기술 문서</td><td nowrap>파일 구조 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR 작업 phase 구조를 README에 정리</td><td nowrap>6월 7일</td><td nowrap>성웅</td><td nowrap>GitHub/README</td><td nowrap>동작 흐름 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>TO_RACK / LIFTING / LOCAL_ENTRY / PLACING / LOCAL_EXIT / RETURN_HOME 단계 문서화</td><td nowrap>6월 7일</td><td nowrap>성웅</td><td nowrap>GitHub/README</td><td nowrap>작업 단계 문서화</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR 초기 위치, 작업대 위치, SG 위치, grid 좌표 변환 방식 정리</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>GitHub/좌표계 문서화</td><td nowrap>좌표 구조 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>시스템 아키텍처와 전체 시나리오를 발표용 PPT 구조로 정리</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>PPT/발표자료</td><td nowrap>발표 흐름 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>팀 발표용 PPT 제작 및 슬라이드 흐름 구성</td><td nowrap>6월 9일</td><td nowrap>성웅</td><td nowrap>PPT/발표자료</td><td nowrap>자료 제작</td><td nowrap>완료</td></tr>
    <tr><td nowrap>프로젝트 목적, 시장 필요성, 시스템 구조, 구현 방식, 개선 내용을 발표 자료에 반영</td><td nowrap>6월 9일</td><td nowrap>성웅</td><td nowrap>PPT/발표자료</td><td nowrap>내용 구성</td><td nowrap>완료</td></tr>
    <tr><td nowrap>팀원별 발표 내용과 전체 발표 흐름 조율</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>팀장/발표 조율</td><td nowrap>발표 준비</td><td nowrap>완료</td></tr>
    <tr><td nowrap>최종 발표용 README 타임라인 및 기여 내용 정리</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>GitHub/문서화</td><td nowrap>기여도 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>발표일 기준 최종 PPT와 GitHub 문서 내용 정리</td><td nowrap>6월 12일</td><td nowrap>성웅</td><td nowrap>PPT/GitHub</td><td nowrap>최종 제출</td><td nowrap>완료</td></tr>
  </tbody>
</table>

---

<h2>성웅 담당 작업 타임라인 (ROS2 Bridge 및 외부 명령 연동)</h2>

<table>
  <thead>
    <tr>
      <th nowrap>작업명</th>
      <th nowrap>날짜</th>
      <th nowrap>담당자</th>
      <th nowrap>파트</th>
      <th nowrap>단계</th>
      <th nowrap>완료 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr><td nowrap>ROS2 Humble 기반 외부 명령 연동 구조 확인</td><td nowrap>6월 5일</td><td nowrap>성웅</td><td nowrap>ROS2/Bridge</td><td nowrap>통신 구조 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>ROS2 Action 기반 AMR 작업 명령 구조 정리</td><td nowrap>6월 5일</td><td nowrap>성웅</td><td nowrap>ROS2/Action</td><td nowrap>인터페이스 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>/manage_workstation Action Server 구조 확인</td><td nowrap>6월 6일</td><td nowrap>성웅</td><td nowrap>ROS2/Action</td><td nowrap>전역 명령 구조</td><td nowrap>완료</td></tr>
    <tr><td nowrap>/amr_01/manage_workstation ~ /amr_05/manage_workstation per-AMR Action 구조 확인</td><td nowrap>6월 6일</td><td nowrap>성웅</td><td nowrap>ROS2/Action</td><td nowrap>AMR 지정 명령 구조</td><td nowrap>완료</td></tr>
    <tr><td nowrap>상대 PC가 특정 AMR에게 직접 명령을 내릴 수 있는 구조 구현</td><td nowrap>6월 6일</td><td nowrap>성웅</td><td nowrap>ROS2/Bridge</td><td nowrap>외부 명령 구조</td><td nowrap>완료</td></tr>
    <tr><td nowrap>per-AMR Action 수신 시 preferred_amr_name과 require_preferred_amr를 JSON에 추가하는 구조 정리</td><td nowrap>6월 6일</td><td nowrap>성웅</td><td nowrap>ROS2/Bridge</td><td nowrap>AMR 강제 배정</td><td nowrap>완료</td></tr>
    <tr><td nowrap>ROS2 Action goal을 JSON command로 변환하는 bridge 구조 분석</td><td nowrap>6월 7일</td><td nowrap>성웅</td><td nowrap>ROS2/Bridge</td><td nowrap>명령 변환 구조</td><td nowrap>완료</td></tr>
    <tr><td nowrap>command_id 기반 commands/status/results 파일 흐름 확인</td><td nowrap>6월 7일</td><td nowrap>성웅</td><td nowrap>Bridge Queue</td><td nowrap>파일 연동 구조</td><td nowrap>완료</td></tr>
    <tr><td nowrap>bridge가 목적지를 직접 계산하지 않고 IsaacSim controller에 명령을 전달하는 역할 분리 구조 정리</td><td nowrap>6월 7일</td><td nowrap>성웅</td><td nowrap>Bridge/Controller 연동</td><td nowrap>역할 분리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>ROS2 feedback/result가 status/result JSON을 통해 반환되는 구조 확인</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>ROS2/Action 결과 처리</td><td nowrap>반환 구조 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>bridge 실행 스크립트 run_bridge_gpu.sh 동작 확인</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>ROS2/실행환경</td><td nowrap>실행 테스트</td><td nowrap>완료</td></tr>
    <tr><td nowrap>ros2 node list 및 ros2 action list를 통한 bridge 정상 여부 검증</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>ROS2/검증</td><td nowrap>통신 검증</td><td nowrap>완료</td></tr>
    <tr><td nowrap>상대 PC에서 좌표 또는 target_location 기반 명령을 내리는 구조 정리</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>ROS2/명령 구조</td><td nowrap>명령 흐름 정리</td><td nowrap>완료</td></tr>
  </tbody>
</table>

---


<h2>성웅 담당 작업 타임라인 (Global Arbiter 및 Bridge 안정화)</h2>

<table>
  <thead>
    <tr>
      <th nowrap>작업명</th>
      <th nowrap>날짜</th>
      <th nowrap>담당자</th>
      <th nowrap>파트</th>
      <th nowrap>단계</th>
      <th nowrap>완료 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr><td nowrap>Global Arbiter가 모든 AMR의 다음 이동 후보를 tick 단위로 승인하는 구조 분석</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>Global Arbiter</td><td nowrap>중앙 승인 구조 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>same-cell 충돌, edge-swap 충돌, head-on 충돌을 Global Arbiter에서 차단하는 기준 정리</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>충돌 회피</td><td nowrap>안전 규칙 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>작업대 운반 중 rack footprint와 AMR footprint를 함께 고려하는 승인 조건 분석</td><td nowrap>6월 9일</td><td nowrap>성웅</td><td nowrap>Global Arbiter/Footprint</td><td nowrap>운반 안전성 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>same-direction convoy following과 tail-release 정책을 적용한 주행 흐름 확인</td><td nowrap>6월 9일</td><td nowrap>성웅</td><td nowrap>교통 흐름 제어</td><td nowrap>다중 AMR 흐름 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>첫 이동 cell이 Global Arbiter에서 reject될 때 wait/no_path가 증가하는 원인 분석</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>Global Arbiter/디버깅</td><td nowrap>병목 원인 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>오래 대기한 AMR에 priority aging을 적용해야 하는 상황 정리</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>우선순위 정책</td><td nowrap>대기 완화 구조 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>Global Arbiter reject 결과를 cost-aware reroute 판단과 연결하는 구조 설계</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Cost Planner/Arbiter</td><td nowrap>WAIT/REROUTE 연결</td><td nowrap>완료</td></tr>
    <tr><td nowrap>Bridge active registry로 active_workstations, active_targets, active_amrs 관리 구조 정리</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>ROS2 Bridge</td><td nowrap>중복 명령 차단</td><td nowrap>완료</td></tr>
    <tr><td nowrap>result, cancel, timeout 발생 시 Bridge active registry를 해제하는 cleanup 흐름 정리</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Bridge Lifecycle</td><td nowrap>상태 해제 구조</td><td nowrap>완료</td></tr>
    <tr><td nowrap>ROS2 Action goal에서 command JSON으로 변환되는 필드와 command_id 추적 기준 문서화</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Bridge Queue</td><td nowrap>명령 추적 구조</td><td nowrap>완료</td></tr>
    <tr><td nowrap>Global Arbiter와 Bridge 구조를 GitHub README/DEBUGGING 문서에 보강</td><td nowrap>6월 12일</td><td nowrap>성웅</td><td nowrap>GitHub 문서화</td><td nowrap>최종 문서 반영</td><td nowrap>완료</td></tr>
  </tbody>
</table>

---

<h2>성웅 담당 작업 타임라인 (IsaacSim AMR Controller 및 주행 로직)</h2>

<table>
  <thead>
    <tr>
      <th nowrap>작업명</th>
      <th nowrap>날짜</th>
      <th nowrap>담당자</th>
      <th nowrap>파트</th>
      <th nowrap>단계</th>
      <th nowrap>완료 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr><td nowrap>IsaacSim 실행 환경 및 AMR stage 구성 확인</td><td nowrap>5월 29일</td><td nowrap>성웅</td><td nowrap>IsaacSim/환경 구성</td><td nowrap>환경 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>IsaacSim Warehouse Creator 및 stage 구성 기준 확인</td><td nowrap>5월 29일</td><td nowrap>성웅</td><td nowrap>IsaacSim/Stage</td><td nowrap>환경 구성</td><td nowrap>완료</td></tr>
    <tr><td nowrap>기존 stage에 배치된 AMR_01~AMR_05 prim 제어 구조 확인</td><td nowrap>6월 5일</td><td nowrap>성웅</td><td nowrap>IsaacSim/Stage</td><td nowrap>AMR 제어 구조</td><td nowrap>완료</td></tr>
    <tr><td nowrap>작업대 RACK_01~RACK_10 또는 WS_01~WS_10 제어 구조 확인</td><td nowrap>6월 5일</td><td nowrap>성웅</td><td nowrap>IsaacSim/Stage</td><td nowrap>작업대 제어 구조</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR 초기 위치와 작업대 초기 위치 grid cell 기준 정리</td><td nowrap>6월 6일</td><td nowrap>성웅</td><td nowrap>IsaacSim/Grid</td><td nowrap>초기 상태 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>target_location이 target_xy와 target_cell로 변환되는 구조 분석</td><td nowrap>6월 6일</td><td nowrap>성웅</td><td nowrap>IsaacSim/좌표 변환</td><td nowrap>좌표계 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>1.5m grid spacing 기준 world 좌표와 grid cell 변환 구조 정리</td><td nowrap>6월 6일</td><td nowrap>성웅</td><td nowrap>IsaacSim/Grid</td><td nowrap>좌표계 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>sg2_in_01_B, sg2_in_02_B, sg2_in_03_B, sg2_out_00_A 좌표 매핑 확인</td><td nowrap>6월 9일</td><td nowrap>성웅</td><td nowrap>IsaacSim/Grid</td><td nowrap>목적지 좌표 검증</td><td nowrap>완료</td></tr>
    <tr><td nowrap>TO_RACK / LIFTING / LOCAL_ENTRY / PLACING / LOCAL_EXIT / RETURN_HOME phase 구조 분석</td><td nowrap>6월 9일</td><td nowrap>성웅</td><td nowrap>IsaacSim/상태머신</td><td nowrap>phase 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>작업대 운반 중 AMR의 carry 상태 및 rack.carried_by 상태 변화 확인</td><td nowrap>6월 9일</td><td nowrap>성웅</td><td nowrap>IsaacSim/상태관리</td><td nowrap>상태 변화 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>SG 진입 구간에서 deterministic local macro route 적용 구조 분석</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>IsaacSim/Local Route</td><td nowrap>병목 구간 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>작업대 운반 AMR의 4방향 이동 제한과 빈 AMR의 8방향 이동 구조 확인</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>IsaacSim/경로계획</td><td nowrap>이동 정책 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>IsaacSim Script Editor에서 controller 실행 및 전체 AMR 제어 테스트</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>IsaacSim/Controller</td><td nowrap>실행 테스트</td><td nowrap>완료</td></tr>
  </tbody>
</table>

---

<h2>성웅 담당 작업 타임라인 (다중 AMR 경로계획 및 충돌 회피)</h2>

<table>
  <thead>
    <tr>
      <th nowrap>작업명</th>
      <th nowrap>날짜</th>
      <th nowrap>담당자</th>
      <th nowrap>파트</th>
      <th nowrap>단계</th>
      <th nowrap>완료 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr><td nowrap>8-way Time A* 기반 AMR 경로계획 구조 분석</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>경로계획</td><td nowrap>알고리즘 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>Reservation Table 기반 time-indexed cell 예약 구조 확인</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>경로계획/충돌회피</td><td nowrap>예약 구조 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>edge swap 충돌 방지 구조 정리</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>충돌회피</td><td nowrap>안전 로직 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>same-direction convoy following 및 tail-release 정책 확인</td><td nowrap>6월 9일</td><td nowrap>성웅</td><td nowrap>충돌회피/교통흐름</td><td nowrap>주행 정책 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>작업대 운반 중 rack footprint 및 soft reservation 구조 확인</td><td nowrap>6월 9일</td><td nowrap>성웅</td><td nowrap>충돌회피/작업대 운반</td><td nowrap>안전거리 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR이 막혔을 때 wait/no_path가 증가하는 원인 분석</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>경로계획/디버깅</td><td nowrap>병목 원인 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>경로 없음(no_path)과 arbiter 대기(wait) 상태를 구분하여 분석</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>경로계획/로그 분석</td><td nowrap>상태 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>LOCAL_ENTRY 구간에서 정적 작업대가 future route를 막는 문제 분석</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>경로계획/Local Macro</td><td nowrap>문제 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>future route에 있는 static blocker를 사전 감지하도록 local macro route 개선</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>경로계획/Local Macro</td><td nowrap>패치 반영</td><td nowrap>완료</td></tr>
    <tr><td nowrap>전체 주행부에 cost-aware reroute 판단을 넣는 구조 설계</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>경로계획/최적화</td><td nowrap>개선 구조 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>TO_RACK / LOCAL_ENTRY / TO_TARGET / LOCAL_EXIT / RETURN_HOME 공통 주행부에 cost 판단 적용</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>경로계획/전체 적용</td><td nowrap>공통 레이어 적용</td><td nowrap>완료</td></tr>
    <tr><td nowrap>COST_DECISION 로그 기반 WAIT/REROUTE 판단 결과 확인 구조 적용</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>경로계획/로그 개선</td><td nowrap>검증 로그 적용</td><td nowrap>완료</td></tr>
  </tbody>
</table>

---

<h2>성웅 담당 작업 타임라인 (QR 위치 인식 및 상태 동기화)</h2>

<table>
  <thead>
    <tr>
      <th nowrap>작업명</th>
      <th nowrap>날짜</th>
      <th nowrap>담당자</th>
      <th nowrap>파트</th>
      <th nowrap>단계</th>
      <th nowrap>완료 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr><td nowrap>AMR 하단 카메라 기반 바닥 QR 인식 구조 확인</td><td nowrap>6월 7일</td><td nowrap>성웅</td><td nowrap>QR/위치인식</td><td nowrap>구조 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>OpenCV QRCodeDetector 기반 QR 디코딩 흐름 확인</td><td nowrap>6월 7일</td><td nowrap>성웅</td><td nowrap>QR/위치인식</td><td nowrap>인식 로직 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>QR ID를 grid cell로 변환하는 구조 정리</td><td nowrap>6월 7일</td><td nowrap>성웅</td><td nowrap>QR/Grid Mapping</td><td nowrap>좌표 변환 정리</td><td nowrap>완료</td></tr>
    <tr><td nowrap>QR MISS 발생 시 fallback 사용 여부 및 hold-last-cell 정책 확인</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>QR/디버깅</td><td nowrap>예외 처리 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>QR 오인식으로 인한 cell jump 문제를 막기 위한 safety gate 구조 확인</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>QR/Safety Gate</td><td nowrap>안전 로직 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR 이동 중 QR cell 갱신을 막는 조건 확인</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>QR/Safety Gate</td><td nowrap>안전 조건 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>LIFTING / PLACING / ROTATING 중 QR 갱신 차단 조건 확인</td><td nowrap>6월 8일</td><td nowrap>성웅</td><td nowrap>QR/Safety Gate</td><td nowrap>상태 기반 차단 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>QR 기반 위치와 transform 기반 위치가 다를 때 발생하는 문제 분석</td><td nowrap>6월 9일</td><td nowrap>성웅</td><td nowrap>QR/디버깅</td><td nowrap>위치 동기화 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>QR MISS 로그에 AMR state, moving, target, carry 정보를 포함하도록 로그 개선 방향 반영</td><td nowrap>6월 9일</td><td nowrap>성웅</td><td nowrap>QR/로그 개선</td><td nowrap>디버깅 로그 개선</td><td nowrap>완료</td></tr>
  </tbody>
</table>

---

<h2>성웅 담당 작업 타임라인 (네트워크 및 CycloneDDS 통신 문제 해결)</h2>

<table>
  <thead>
    <tr>
      <th nowrap>작업명</th>
      <th nowrap>날짜</th>
      <th nowrap>담당자</th>
      <th nowrap>파트</th>
      <th nowrap>단계</th>
      <th nowrap>완료 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr><td nowrap>상대 PC에서 AMR Action Server가 보이지 않는 문제 확인</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>ROS2 네트워크</td><td nowrap>문제 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>내 PC에서는 /fleet_manager_bridge_node와 manage_workstation action이 정상 표시되는 것 확인</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>ROS2 네트워크</td><td nowrap>로컬 검증</td><td nowrap>완료</td></tr>
    <tr><td nowrap>상대 PC에서 action discovery가 되지 않는 원인을 DDS interface 설정 문제로 분석</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>CycloneDDS</td><td nowrap>원인 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>~/.ros/cyclonedds_thunderbolt.xml 설정 확인</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>CycloneDDS</td><td nowrap>설정 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>NetworkInterface가 thunderbolt0으로 고정되어 있던 문제 발견</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>CycloneDDS</td><td nowrap>문제 원인 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>현재 실제 통신망이 Wi-Fi 192.168.10.x 대역임을 확인</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>네트워크</td><td nowrap>IP 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>Wi-Fi 인터페이스 wlp128s20f3 기준으로 CycloneDDS XML 수정</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>CycloneDDS</td><td nowrap>설정 수정</td><td nowrap>완료</td></tr>
    <tr><td nowrap>Peer address를 Thunderbolt 대역에서 Wi-Fi 대역으로 변경</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>CycloneDDS</td><td nowrap>Peer 설정 수정</td><td nowrap>완료</td></tr>
    <tr><td nowrap>ROS_DOMAIN_ID=119, rmw_cyclonedds_cpp, CYCLONEDDS_URI 환경변수 확인</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>ROS2 환경설정</td><td nowrap>환경변수 검증</td><td nowrap>완료</td></tr>
    <tr><td nowrap>bridge 재시작 후 ros2 action list로 discovery 재검증</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>ROS2 네트워크</td><td nowrap>검증</td><td nowrap>완료</td></tr>
  </tbody>
</table>

---

<h2>성웅 담당 작업 타임라인 (중복 명령 및 목적지 충돌 문제 해결)</h2>

<table>
  <thead>
    <tr>
      <th nowrap>작업명</th>
      <th nowrap>날짜</th>
      <th nowrap>담당자</th>
      <th nowrap>파트</th>
      <th nowrap>단계</th>
      <th nowrap>완료 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr><td nowrap>WS05와 WS06이 같은 target_location으로 명령을 받은 문제 확인</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>명령 충돌/디버깅</td><td nowrap>로그 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>sg2_in_03_B가 target_cell=(4,-5)로 중복 배정된 문제 확인</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>명령 충돌/디버깅</td><td nowrap>원인 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR_04가 WS05를 먼저 배치한 후 AMR_03이 WS06을 들고 같은 cell에 진입하려는 문제 확인</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>다중 AMR 디버깅</td><td nowrap>상태 분석</td><td nowrap>완료</td></tr>
    <tr><td nowrap>해당 문제가 경로계획 문제가 아니라 명령 충돌 문제임을 판단</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>문제 원인 분석</td><td nowrap>원인 분류</td><td nowrap>완료</td></tr>
    <tr><td nowrap>같은 workstation_id 중복 명령 차단 구조 설계</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Bridge Guard</td><td nowrap>개선 구조 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>같은 target_location 또는 target_cell 중복 명령 차단 구조 설계</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Bridge Guard</td><td nowrap>개선 구조 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>같은 preferred AMR에 중복 명령이 들어오는 경우 차단 구조 설계</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Bridge Guard</td><td nowrap>개선 구조 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>fleet_manager_bridge_node_gpu_v43_guarded_actions.py admission guard 패치 적용</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>ROS2 Bridge</td><td nowrap>패치 적용</td><td nowrap>완료</td></tr>
    <tr><td nowrap>DUPLICATE_WORKSTATION / DUPLICATE_TARGET / DUPLICATE_AMR reject 구조 반영</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Bridge Guard</td><td nowrap>예외 처리 반영</td><td nowrap>완료</td></tr>
    <tr><td nowrap>중복 명령 reject 시 action result에 실패 결과를 반환하는 구조 확인</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>ROS2 Action</td><td nowrap>결과 처리 확인</td><td nowrap>완료</td></tr>
  </tbody>
</table>

---

<h2>성웅 담당 작업 타임라인 (Cost-aware Reroute 주행 최적화)</h2>

<table>
  <thead>
    <tr>
      <th nowrap>작업명</th>
      <th nowrap>날짜</th>
      <th nowrap>담당자</th>
      <th nowrap>파트</th>
      <th nowrap>단계</th>
      <th nowrap>완료 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr><td nowrap>기존 AMR 주행에서 기다림과 우회 판단이 부족한 문제 인식</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>경로계획/최적화</td><td nowrap>문제 인식</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR이 막혔을 때 wait_cost와 detour_cost를 비교하는 구조 설계</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>경로계획/Cost Planner</td><td nowrap>개선 구조 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>LOCAL_ENTRY만이 아니라 전체 주행부에 cost 판단을 넣는 구조로 방향 확정</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>경로계획/전체 적용</td><td nowrap>설계 확정</td><td nowrap>완료</td></tr>
    <tr><td nowrap>각 phase에 개별 적용하지 않고 공통 주행 판단부에 cost planner를 넣는 방식 결정</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>경로계획/구조 설계</td><td nowrap>공통 레이어 설계</td><td nowrap>완료</td></tr>
    <tr><td nowrap>TO_RACK / LOCAL_ENTRY / TO_TARGET / LOCAL_EXIT / RETURN_HOME 전체 적용 구조 반영</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>경로계획/전체 적용</td><td nowrap>전체 주행 적용</td><td nowrap>완료</td></tr>
    <tr><td nowrap>global arbiter가 첫 이동을 reject한 경우 detour A*를 다시 계산하는 구조 적용</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Cost Planner</td><td nowrap>우회 판단 적용</td><td nowrap>완료</td></tr>
    <tr><td nowrap>rejected first cell을 temporary blocked cell로 두고 우회 경로를 재탐색하는 방식 적용</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Cost Planner</td><td nowrap>패치 적용</td><td nowrap>완료</td></tr>
    <tr><td nowrap>작업대 운반 중인 AMR은 우회에 더 보수적인 비용을 적용하도록 설정</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Cost Planner</td><td nowrap>carry penalty 적용</td><td nowrap>완료</td></tr>
    <tr><td nowrap>COST_DECISION 로그로 WAIT/REROUTE 판단 결과를 확인할 수 있도록 반영</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>디버깅 로그</td><td nowrap>검증 로그 적용</td><td nowrap>완료</td></tr>
    <tr><td nowrap>amr_live_existing_stage_true8_qr_camera_controller_gpu_v42_cost_aware_global.py 패치 적용</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>IsaacSim Controller</td><td nowrap>주행 최적화 패치</td><td nowrap>완료</td></tr>
  </tbody>
</table>

---

<h2>성웅 담당 작업 타임라인 (실행 테스트 및 검증)</h2>

<table>
  <thead>
    <tr>
      <th nowrap>작업명</th>
      <th nowrap>날짜</th>
      <th nowrap>담당자</th>
      <th nowrap>파트</th>
      <th nowrap>단계</th>
      <th nowrap>완료 여부</th>
    </tr>
  </thead>
  <tbody>
    <tr><td nowrap>IsaacSim Script Editor에서 AMR controller 실행 테스트</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>IsaacSim 실행</td><td nowrap>실행 검증</td><td nowrap>완료</td></tr>
    <tr><td nowrap>bridge 실행 후 ROS2 action server 목록 확인</td><td nowrap>6월 10일</td><td nowrap>성웅</td><td nowrap>ROS2 실행</td><td nowrap>실행 검증</td><td nowrap>완료</td></tr>
    <tr><td nowrap>business open sequence 실행을 통한 다중 AMR 작업 테스트</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>시나리오 테스트</td><td nowrap>통합 테스트</td><td nowrap>완료</td></tr>
    <tr><td nowrap>current_run_full_log.txt 기반 tick별 AMR 상태 분석</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>로그 분석</td><td nowrap>상태 검증</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR별 state, cell, target, carry, wait, no_path 상태 확인</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>로그 분석</td><td nowrap>주행 상태 검증</td><td nowrap>완료</td></tr>
    <tr><td nowrap>작업대가 목표 SG 또는 stage 위치에 정상 배치되는지 확인</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>시나리오 검증</td><td nowrap>작업 결과 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>중복 명령이 들어왔을 때 bridge에서 reject되는지 확인</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Bridge Guard 검증</td><td nowrap>예외 처리 검증</td><td nowrap>완료</td></tr>
    <tr><td nowrap>AMR이 막혔을 때 COST_DECISION 로그가 출력되는지 확인</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>Cost Planner 검증</td><td nowrap>로그 검증</td><td nowrap>완료</td></tr>
    <tr><td nowrap>ROS2 Action feedback/result가 상대 PC에 정상 반환되는지 확인</td><td nowrap>6월 11일</td><td nowrap>성웅</td><td nowrap>통신 검증</td><td nowrap>결과 반환 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>발표 당일 기준 시연 흐름 및 발표 자료 최종 확인</td><td nowrap>6월 12일</td><td nowrap>성웅</td><td nowrap>최종 발표 준비</td><td nowrap>최종 확인</td><td nowrap>완료</td></tr>
    <tr><td nowrap>협동3 최종 발표 및 질의응답 대응</td><td nowrap>6월 12일</td><td nowrap>성웅</td><td nowrap>최종 발표</td><td nowrap>발표/Q&A</td><td nowrap>완료</td></tr>
  </tbody>
</table>

---

## 📊 타임라인

```mermaid
gantt
    title 성웅 담당 작업 타임라인 - 협동3
    dateFormat  YYYY-MM-DD

    section 프로젝트 기획/팀장
    프로젝트 주제 기획 및 방향 설정              :done, p1, 2026-05-29, 1d
    AMR Fleet 물류 시나리오 설계                 :done, p2, 2026-05-30, 2d
    팀원 역할 조율 및 구현 방향 정리              :done, p3, 2026-06-01, 3d
    전체 시스템 시나리오 및 운영 구조 조율          :done, p4, 2026-06-02, 4d
    최종 발표 흐름 조율                           :done, p5, 2026-06-10, 2d
    최종 발표                                    :done, p6, 2026-06-12, 1d

    section GitHub/PPT 문서화
    README 구조 설계 및 프로젝트 개요 정리          :done, d1, 2026-06-04, 1d
    시스템 구조 및 bridge queue 문서화             :done, d2, 2026-06-06, 2d
    AMR phase 및 좌표계 문서화                     :done, d3, 2026-06-07, 2d
    발표용 PPT 제작 및 슬라이드 구성                :done, d4, 2026-06-09, 2d
    최종 README/PPT 정리                           :done, d5, 2026-06-11, 1d
    발표일 최종 자료 반영                          :done, d6, 2026-06-12, 1d

    section ROS2 Bridge
    manage_workstation Action 구조 확인            :done, r1, 2026-06-05, 1d
    per-AMR Action alias 구조 구현                  :done, r2, 2026-06-06, 1d
    bridge_queue command/status/result 연동 분석    :done, r3, 2026-06-07, 1d
    bridge 실행 및 action list 검증                 :done, r4, 2026-06-08, 1d
    bridge v43 admission guard 패치                 :done, r5, 2026-06-11, 1d

    section Global Arbiter/Bridge 안정화
    Global Arbiter tick 승인 구조 분석              :done, g1, 2026-06-08, 1d
    same-cell/edge-swap 충돌 차단 규칙 정리          :done, g2, 2026-06-08, 1d
    rack footprint 기반 운반 안전성 확인             :done, g3, 2026-06-09, 1d
    Arbiter reject와 cost-aware reroute 연결         :done, g4, 2026-06-11, 1d
    Bridge active registry cleanup 정리              :done, g5, 2026-06-11, 1d

    section IsaacSim Controller
    IsaacSim stage 및 AMR prim 구조 확인             :done, c1, 2026-05-29, 1d
    AMR/작업대 제어 구조 확인                       :done, c2, 2026-06-05, 1d
    target_cell 좌표 변환 및 phase 구조 분석         :done, c3, 2026-06-06, 2d
    Local Macro Entry/Exit 구조 분석                 :done, c4, 2026-06-10, 1d
    cost-aware global reroute 패치                  :done, c5, 2026-06-11, 1d

    section QR/경로계획/충돌회피
    QR 위치 인식 및 safety gate 확인                :done, q1, 2026-06-07, 2d
    Time A* 및 reservation table 분석               :done, q2, 2026-06-08, 1d
    wait/no_path 및 Local Macro 문제 분석            :done, q3, 2026-06-10, 1d
    중복 목적지 문제 분석 및 개선                   :done, q4, 2026-06-11, 1d
    COST_DECISION 기반 reroute 검증                 :done, q5, 2026-06-11, 1d

    section 네트워크/통신
    ROS2 discovery 문제 분석                        :done, n1, 2026-06-10, 1d
    CycloneDDS thunderbolt0 고정 문제 확인           :done, n2, 2026-06-10, 1d
    Wi-Fi interface 및 peer 설정 수정                :done, n3, 2026-06-10, 1d

    section 테스트/검증
    IsaacSim + Bridge 통합 실행 테스트               :done, t1, 2026-06-10, 1d
    다중 AMR 작업 시나리오 검증                     :done, t2, 2026-06-11, 1d
    로그 기반 wait/no_path/cost decision 분석        :done, t3, 2026-06-11, 1d
    발표 당일 최종 시연 및 발표                     :done, t4, 2026-06-12, 1d
```

---

## 🔑 핵심 전환점

| #  | 전환점                          | 관련 내용                                                                         | 날짜     |
| -- | ---------------------------- | ----------------------------------------------------------------------------- | ------ |
| 1  | 프로젝트 방향 설정                   | IsaacSim 기반 AMR Fleet 물류 자동화 시뮬레이션으로 기획                                       | 5월 29일 |
| 2  | 실제 창고 시나리오 구체화               | AMR 5대와 작업대 다수를 사용하는 픽업·운반·배치 흐름 설계                                           | 5월 30일 |
| 3  | 팀장 역할 확정                     | 전체 시나리오 설계, 팀원 조율, 구현 방향 결정, GitHub 문서화, PPT 제작 담당                            | 6월 1일  |
| 4  | 외부 명령 연동 구조 결정               | 상대 PC 또는 Control Tower에서 ROS2 Action으로 AMR 작업 명령을 내리는 구조로 설계                  | 6월 5일  |
| 5  | per-AMR Action 구조 적용         | `/amr_01/manage_workstation` ~ `/amr_05/manage_workstation`으로 특정 AMR 지정 명령 가능 | 6월 6일  |
| 6  | bridge_queue 구조 확정           | commands/status/results JSON 파일을 통해 ROS2 bridge와 IsaacSim controller를 분리      | 6월 7일  |
| 6-1 | Global Arbiter 구조 명확화        | 모든 AMR의 다음 이동 후보를 중앙에서 승인하고, same-cell/edge-swap/footprint 충돌을 차단하는 구조로 정리 | 6월 8일  |
| 6-2 | Bridge Lifecycle 관리 보강        | command_id 기준으로 active command를 추적하고, result/cancel/timeout 시 registry를 해제하는 흐름 정리 | 6월 11일 |
| 7  | QR 기반 위치 인식 구조 반영            | AMR 하단 카메라가 바닥 QR을 읽어 grid cell을 갱신하는 구조 확인                                   | 6월 7일  |
| 8  | Local Macro Route 적용         | SG 진입부에서 일반 A*만으로는 불안정하여 deterministic local route 구조 사용                      | 6월 10일 |
| 9  | CycloneDDS 통신 문제 해결          | Thunderbolt 인터페이스 고정 문제를 Wi-Fi 인터페이스 기준으로 수정                                  | 6월 10일 |
| 10 | 중복 목적지 문제 발견                 | WS05와 WS06이 같은 `sg2_in_03_B`로 들어가며 같은 target_cell에 배치되는 문제 확인                 | 6월 11일 |
| 11 | bridge admission guard 적용    | 같은 workstation, 같은 target, 같은 preferred AMR 중복 명령을 bridge에서 사전 차단             | 6월 11일 |
| 12 | cost-aware global reroute 적용 | 전체 주행 공통부에 WAIT/REROUTE cost 판단 구조 추가                                         | 6월 11일 |
| 13 | 발표 자료 및 README 최종 정리         | 프로젝트 기획, 구현 구조, 문제 해결 과정, 기여 내용을 발표용으로 정리                                     | 6월 12일 |
| 14 | 최종 발표 진행                     | 협동3 최종 발표 및 질의응답 대응                                                           | 6월 12일 |

---

## 🧩 담당 역할 요약

성웅은 협동3 프로젝트에서 팀장 역할을 맡아 전체 프로젝트 기획, AMR Fleet 운영 시나리오 설계, 팀원별 구현 방향 조율, GitHub README 문서 작업, PPT 제작, 발표 준비 및 최종 발표를 담당하였다.

이번 프로젝트는 단순히 IsaacSim에서 AMR을 움직이는 테스트가 아니라, 외부 PC에서 ROS2 Action 명령을 내리고, 해당 명령이 bridge를 거쳐 IsaacSim controller로 전달되며, AMR이 작업대를 픽업·운반·배치한 뒤 결과를 다시 반환하는 전체 시스템을 구성하는 방식으로 진행되었다.

성웅은 프로젝트 초기에 전체 시스템의 방향을 AMR 기반 물류 자동화 시뮬레이션으로 설정하고, 실제 창고에서 사용할 수 있는 작업 흐름을 기준으로 AMR 5대와 작업대 다수를 사용하는 시나리오를 구성하였다. 이후 팀원들과 구현 방향을 조율하면서 어떤 PC에서 IsaacSim을 실행하고, 어떤 PC에서 ROS2 명령을 내릴지, bridge와 controller의 역할을 어떻게 나눌지 정리하였다.

기술적으로는 ROS2 bridge 구조, per-AMR Action Server 구조, bridge_queue 기반 JSON 명령 흐름, IsaacSim controller의 작업 phase, QR 기반 위치 인식, Time A*와 reservation table 기반 경로계획, Local Macro Route 기반 SG 진입 구조를 분석하고 구현에 반영하였다.

또한 다중 AMR 작업 중 발생한 중복 목적지 문제를 분석하여, 같은 workstation_id 또는 같은 target_location으로 중복 명령이 들어올 경우 bridge 단계에서 사전에 차단하는 admission guard 구조를 적용하였다. 이로 인해 서로 다른 작업대가 같은 목적지 cell에 배치되려는 문제를 방지할 수 있게 되었다.

추가로 AMR이 막혔을 때 단순히 기다리는 기존 방식의 한계를 확인하고, 기다리는 비용과 우회 비용을 비교하는 cost-aware global reroute 구조를 controller에 적용하였다. 이 패치는 특정 phase에만 적용한 것이 아니라, TO_RACK, LOCAL_ENTRY, TO_TARGET, LOCAL_EXIT, RETURN_HOME 등 전체 주행 흐름이 공통으로 거치는 주행 판단부에 적용하였다.

문서화 측면에서는 GitHub README에 들어갈 프로젝트 개요, 시스템 구조, 실행 방법, 주요 기능, 문제 해결 과정, 패치 내용, 검증 방법을 정리하였고, 발표용 PPT에서는 프로젝트 목적, 전체 시나리오, 시스템 아키텍처, ROS2-IsaacSim 연동 구조, 주행 로직 개선 내용을 시각적으로 구성하였다.

---


## 🧠 Global Arbiter와 Bridge 핵심 구현 요약

### 1. Bridge는 “명령 변환 계층”으로 분리하였다

협동3 시스템에서 Bridge는 AMR을 직접 움직이는 코드가 아니라, 외부 ROS2 명령과 IsaacSim controller 사이를 연결하는 중간 계층이다. 상대 PC 또는 Control Tower가 ROS2 Action Goal을 보내면 Bridge는 이를 `command_id`가 포함된 JSON 명령으로 변환하고, `bridge_queue/commands` 폴더에 저장한다. 이후 IsaacSim controller가 해당 JSON을 읽어 실제 AMR 주행을 수행한다.

```text
ROS2 Action Goal
→ Bridge 수신
→ command_id 생성
→ command JSON 저장
→ Controller가 command JSON 읽음
→ status/result JSON 작성
→ Bridge가 Action feedback/result 반환
```

이 구조를 사용한 이유는 ROS2 통신 계층과 IsaacSim 제어 계층을 직접 강하게 묶지 않기 위해서이다. Bridge는 명령 접수, command_id 추적, feedback/result 반환, cancel 처리, 중복 명령 차단을 담당하고, controller는 실제 AMR 배정, 경로계획, 작업대 운반, 배치, 복귀를 담당한다.

### 2. Bridge Admission Guard로 불가능한 명령을 사전에 차단하였다

다중 AMR 환경에서는 같은 작업대나 같은 목적지로 동시에 명령이 들어오면 controller가 아무리 경로계획을 잘해도 물리적으로 해결할 수 없다. 따라서 Bridge 단계에서 다음 조건을 먼저 검사하도록 정리하였다.

```text
1. 같은 workstation_id가 이미 작업 중이면 reject
2. 같은 target_location 또는 target_cell이 이미 예약되어 있으면 reject
3. 같은 preferred AMR이 이미 작업 중이면 reject
4. 명령이 완료, 실패, 취소, timeout되면 active registry에서 해제
```

이를 통해 잘못된 명령이 IsaacSim controller까지 내려가기 전에 `DUPLICATE_WORKSTATION`, `DUPLICATE_TARGET`, `DUPLICATE_AMR`로 차단되도록 하였다.

### 3. Global Arbiter는 “다중 AMR 이동 승인 계층”으로 정리하였다

Global Arbiter는 각 AMR이 계산한 다음 이동 후보를 그대로 실행시키지 않고, 매 tick마다 전체 AMR의 이동 요청을 모아 충돌 가능성을 검사한 뒤 승인 또는 대기시킨다.

```text
각 AMR이 next_cell 후보 계산
→ Global Arbiter가 전체 이동 후보 수집
→ same-cell 충돌 검사
→ edge-swap 충돌 검사
→ rack footprint 충돌 검사
→ 예약 테이블과 비교
→ 승인된 AMR만 이동
→ 거부된 AMR은 WAIT 또는 REROUTE 판단
```

Global Arbiter가 필요한 이유는 AMR이 1대일 때는 로컬 경로계획만으로 충분하지만, AMR 5대가 동시에 움직이면 각 AMR이 개별적으로 안전한 경로를 계산해도 같은 tick에 같은 cell을 요구하거나 서로 자리를 맞바꾸는 충돌이 발생할 수 있기 때문이다.

### 4. Global Arbiter와 Cost-aware Reroute를 연결하였다

기존에는 Global Arbiter가 첫 이동 cell을 reject하면 AMR이 단순히 기다리는 구조였다. 개선 후에는 reject된 cell을 temporary blocked cell로 보고, 그 cell을 피하는 detour A*를 다시 계산한다. 이후 기다리는 비용과 우회 비용을 비교해 `WAIT` 또는 `REROUTE`를 선택한다.

```text
Arbiter reject 발생
→ rejected first cell을 임시 blocked cell로 지정
→ detour A* 재계산
→ wait_cost와 detour_cost 비교
→ wait가 유리하면 WAIT
→ 우회가 유리하면 REROUTE
```

이 구조 덕분에 협동3 시스템은 단순히 “충돌하지 않게 멈추는 구조”에서 “대기와 우회를 비교해 더 효율적인 이동을 선택하는 구조”로 개선되었다.

### 5. 최종적으로 분리된 책임 구조

| 계층 | 담당 역할 | 핵심 개선 |
| --- | --- | --- |
| ROS2 Action Client | 외부 PC 또는 Control Tower에서 AMR 작업 명령 전송 | 명령 송신 계층 분리 |
| Fleet Manager Bridge | Action goal을 JSON command로 변환하고 feedback/result 반환 | admission guard, active registry, command_id 추적 |
| Bridge Queue | commands/status/results/cancel/done 파일 기반 비동기 연동 | ROS2와 IsaacSim 결합도 감소 |
| IsaacSim Controller | AMR 배정, 작업대 픽업·운반·배치·복귀 수행 | phase 기반 작업 제어 |
| Global Arbiter | 모든 AMR의 tick 단위 이동 승인 | same-cell, edge-swap, footprint 충돌 차단 |
| Cost-aware Reroute | Arbiter reject 시 WAIT/REROUTE 판단 | 병목 상황에서 효율 개선 |

---

## ✅ 최종 정리

성웅의 협동3 프로젝트 기여는 단일 코드 수정에 한정되지 않는다. 프로젝트의 전체 방향을 기획하고, 실제 창고 물류 자동화를 가정한 AMR Fleet 시나리오를 설계했으며, 팀원별 구현 방향을 조율하고, GitHub 문서화와 PPT 제작, 최종 발표까지 담당한 팀장 역할을 수행하였다.

기술적으로는 ROS2 Action 기반 외부 명령 구조, IsaacSim controller 기반 AMR 주행 구조, bridge_queue 기반 JSON 연동 구조, Bridge admission guard, Global Arbiter 기반 tick 단위 이동 승인 구조, CycloneDDS 네트워크 설정, QR 기반 위치 인식, Time A* 경로계획, Local Macro Route, cost-aware reroute까지 프로젝트의 핵심 연결부를 분석하고 개선하였다.

특히 중복 목적지로 인해 AMR이 멈추는 문제를 단순 경로계획 문제가 아니라 명령 충돌 문제로 분류하고, bridge 단계에서 중복 workstation, 중복 target, 중복 AMR 명령을 차단하도록 개선한 점이 주요 성과이다. 또한 AMR이 막혔을 때 무조건 기다리는 방식에서 벗어나, 기다림과 우회 비용을 비교하는 cost-aware global reroute 구조를 추가하여 다중 AMR 주행의 효율성을 높였다.

결과적으로 협동3 프로젝트에서 성웅은 팀장으로서 기획, 시나리오 설계, 팀 조율, 문서화, PPT 제작, 발표를 담당했고, 기술적으로는 ROS2와 IsaacSim을 연결하는 핵심 구조와 다중 AMR 주행 안정성 개선을 주도하였다.
