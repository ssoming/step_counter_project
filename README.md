# MPU9250 기반 FPGA SoC 스마트 만보기

>
 
[![Notion](https://img.shields.io/badge/Notion-프로젝트%20상세%20보기-000000?style=for-the-badge&logo=notion&logoColor=white)](https://app.notion.com/p/minseokim-profile/Custom-IP-360b5d65c68c8011a3efc4ca7c6ca202?source=copy_link)
 
---

## 📌 개요
 
BASYS3(MicroBlaze RISC-V) 위에 UART · I2C_MPU · MAX7219 · STEP IP를 직접 설계·통합하여,
MPU9250 가속도 센서로 걸음 수를 측정하고 블루투스(HC-06)로 스마트폰과 통신하는 휴대형 스마트 만보기.
 
| 항목 | 내용 |
|---|---|
| 플랫폼 | BASYS3 (Xilinx Artix-7) |
| 개발 환경 | Vivado HLx · Vitis |
| 통신 | HC-06 Bluetooth (UART 9600 bps) |
| 디스플레이 | MAX7219 8×32 도트매트릭스 (SPI) |
| 개발 기간 | 2025.05.14 - 20 |
| 팀 구성 | 3인 |

## ✨ 주요 기능
 
- **걸음 수 측정** : MPU9250 3축 가속도 합산 모션 레벨 기반 알고리즘 (HIGH_TH / LOW_TH / cooldown 파라미터)
- **실시간 디스플레이** : MAX7219 8×32 도트매트릭스에 현재 걸음 수 즉시 반영
- **블루투스 통신** : 스마트폰 앱에서 목표 걸음수 입력(1~9999) → 달성/중단 결과 수신
- **운동 결과 출력** : 종료 시 걸음수 · 거리 · 시간(MM:SS) · 속도(km/h) 블루투스 전송
- **상태 머신 제어** : `WAIT_GOAL → RUNNING → DONE` 3-state 구조로 전체 흐름 관리
- **휴대성** : 보조 배터리 구동 · 비휘발성 메모리 저장

## 📁 파일 구성
 
| 경로 | 내용 |
|---|---|
| `SoC/ip_repo/` | 커스텀 IP 소스 (dotmatrix · i2c_mpu · myip_rxtx · step_counter) |
| `SoC/project_all/` | Vivado 프로젝트 (Block Design · 합성·구현 결과 · `.xsa` · `.xpr`) |
| `Vitis/Project_MPU_Counter/` | Vitis 애플리케이션 (상태 머신 · BT 통신) |
| `Vitis/platform_Project_MPU_Counter/` | Vitis 플랫폼 (BSP · `xparameters.h`) |
| `SoC Project IP Register Description.docx` | IP 레지스터 데이터시트 |
 

## 🔧 주요 구현
 
| 구분 | 내용 |
|---|---|
| `i2c_mpu_ip` | MPU9250 I2C 통신 · 3축 가속도 AXI 레지스터 반영 · `data_valid` 신호 출력 |
| `step_counter_ip` | 3축 합산 모션 레벨 기반 걸음 수·거리 산출 · 임계값 파라미터 런타임 설정 |
| `dotmatrix_ip` | MAX7219 SPI 제어 (din · cs · csk) · `CTRL_UPDATE` Write로 걸음 수 즉시 표시 |
| `myip_rxtx` | TX/RX 겸용 커스텀 UART · `BAUD_DIV` 레지스터로 보드레이트 가변 설정 |
| Vitis `helloworld.c` | 3-state 상태 머신 · IP 레지스터 폴링 · 블루투스 다자리 수신 · 결과 포맷 출력 |
 

## 👤 담당 역할

- UART IP 설계 참여
- Vitis 소프트웨어 구현(모니터링 초기 버전 → 상태 머신 최종 병합)
- 블루투스 통신 로직
- 데이터시트 작성 · 발표 
