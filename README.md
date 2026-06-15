# 건강수호자 (Health Guardian)

> MPU9250 가속도 센서와 FPGA SoC 기반 커스텀 IP를 활용한 블루투스 연동 스마트 만보기

[![Notion](https://img.shields.io/badge/Notion-프로젝트%20상세%20보기-000000?style=for-the-badge&logo=notion&logoColor=white)](https://app.notion.com/p/minseokim-profile/Custom-IP-360b5d65c68c8011a3efc4ca7c6ca202?source=copy_link)

---

## 1. 개요 (Overview)

| 항목 | 내용 |
|------|------|
| 플랫폼 | BASYS3 (Xilinx Artix-7 FPGA) |
| 언어 | Verilog, C |
| 도구 | Vivado HLx, Vitis |
| 통신 | I2C (MPU9250), SPI (MAX7219), UART / Bluetooth (HC-06) |
| 개발 기간 | 2026.05 |
| 팀 구성 | 3인 팀 프로젝트 |

---

## 2. 담당 역할 (My Role)

- UART IP 설계 참여 — AXI4-Lite 슬레이브 기반 TX/RX 겸용 커스텀 UART IP 설계
- Vitis 소프트웨어 구현 — MPU + STEP 모니터링 초기 버전 작성 및 상태 머신 기반 최종 통합 로직 병합
- 블루투스 통신 로직 — 목표 걸음수 수신 및 결과(걸음수·거리·시간·속도) 포맷 변환 후 HC-06 전송 구현
- 데이터시트 작성 · 발표

---

## 3. 주요 기능 (Key Features)

- **걸음 수 측정** — MPU9250 3축 가속도 합산 모션 레벨 기반 알고리즘으로 걸음 수·거리 실시간 산출 및 MAX7219 8×32 도트매트릭스 즉시 반영
- **블루투스 연동** — HC-06으로 목표 걸음수 입력(1~9999) → 달성/중단 시 걸음수·거리·시간(MM:SS)·속도(km/h) 결과 전송
- **상태 머신 제어** — `WAIT_GOAL → RUNNING → DONE` 3-state 구조로 목표 설정·측정·결과 출력 흐름 관리, 보조 배터리 구동으로 휴대 가능

---

## 4. 기술 스택 및 아키텍처 (Tech Stack & Architecture)

![Verilog](https://img.shields.io/badge/Verilog-FF6B35?style=for-the-badge&logo=v&logoColor=white)
![C](https://img.shields.io/badge/C-A8B9CC?style=for-the-badge&logo=c&logoColor=white)
![Basys3](https://img.shields.io/badge/Basys3-E01F27?style=for-the-badge&logo=xilinx&logoColor=white)
![Vivado](https://img.shields.io/badge/Vivado-FF0000?style=for-the-badge&logo=xilinx&logoColor=white)
![Vitis](https://img.shields.io/badge/Vitis-FF6600?style=for-the-badge&logo=xilinx&logoColor=white)

### 시스템 구성

| 구성 요소 | 역할 |
|------|------|
| MicroBlaze RISC-V | AXI Interconnect 기반 소프트 프로세서 |
| I2C_MPU IP | MPU9250 가속도 X·Y·Z 주기 읽기, `data_valid` 신호 출력 |
| STEP IP | 3축 합산 모션 레벨 기반 걸음 수·거리 산출, 임계값 파라미터 레지스터 제공 |
| MAX7219 IP | 8×32 도트매트릭스 SPI 제어 (din · cs · csk) |
| UART IP (myip_rxtx) | TX/RX 겸용, `BAUD_DIV` 레지스터로 보드레이트 가변 설정 |

---

## 5. 핵심 구현 및 트러블슈팅 (Key Implementation & Troubleshooting)

### 핵심 구현

**① 상태 머신 기반 전체 제어 흐름**
`STATE_WAIT_GOAL → STATE_RUNNING → STATE_DONE` 3-state 구조로 목표 수신, 측정, 결과 출력 흐름을 분리. `base_step_count` 오프셋 기준으로 세션별 상대값을 계산해 이전 세션 누적값 영향을 제거했다.

**② IP 레지스터 직접 접근 (Vitis)**
`Xil_In32()` / `Xil_Out32()`로 STEP IP의 걸음수·거리 레지스터를 200ms 주기로 폴링하고, 임계값(`HIGH_TH`, `LOW_TH` 등)을 런타임에 직접 설정해 감도를 조정했다.

**③ 블루투스 다자리 수신 (`rxtx_recv_goal`)**
최대 4자리를 버퍼링하고 `'\r'/'\n'` 수신 또는 idle 500,000회 타임아웃 시 종료하는 논블로킹 수신 함수를 적용해 안정적인 목표 입력을 구현했다.

### 트러블슈팅

| 발생 문제 | 발생 원인 | 해결 방안 | 결과 |
|-----------|-----------|-----------|------|
| 블루투스 수신 시 한 자리 숫자만 인식 | 기존 수신 함수가 첫 번째 바이트 수신 즉시 반환하는 구조 | 팀원이 4자리 버퍼링 + idle 타임아웃 적용한 `rxtx_recv_goal()` 재설계, 최종 통합 로직에 병합 | 1~9999 범위 목표 걸음수 정상 수신 |

---

## 6. 디렉토리 구조 (Directory Structure)

```
step_counter_project/
│
├── SoC/
│   ├── ip_repo/                              # 커스텀 IP 소스 (Verilog)
│   │   ├── dotmatrix_ip_1_0/                 # MAX7219 도트매트릭스 IP
│   │   ├── i2c_mpu_ip_1_0/                   # I2C MPU9250 IP
│   │   ├── myip_rxtx_1_0/                    # 커스텀 UART TX/RX IP
│   │   └── step_counter_ip_1_0/              # 걸음 수 카운터 IP
│   │
│   └── project_all/                          # Vivado 프로젝트
│       ├── project_all.srcs/                 # Block Design 소스
│       ├── project_all.runs/                 # 합성·구현 결과
│       ├── project_all.hw/                   # 하드웨어 핸드오프
│       ├── project_all.ip_user_files/        # IP 사용자 파일
│       ├── design_1_wrapper.xsa              # Vitis 연동용 하드웨어 명세
│       └── project_all.xpr                   # Vivado 프로젝트 파일
│
├── Vitis/
│   ├── platform_Project_MPU_Counter/         # Vitis 플랫폼 (BSP · xparameters.h)
│   │   ├── export/
│   │   ├── hw/
│   │   └── microblaze_riscv_0/
│   │
│   └── Project_MPU_Counter/                  # Vitis 애플리케이션 프로젝트
│       ├── src/helloworld.c                  # 메인 소프트웨어 (상태 머신 · BT 통신)
│       └── build/
│
└── SoC Project IP Register Description.docx  # IP 레지스터 데이터시트
```
