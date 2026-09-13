# A Transient-Enhanced Self-Clocked Hybrid LDO With Digital Coarse Search and Analog Body-Bias Fine-Tuning

**과목/분야**: 종합설계(캡스톤디자인), 3인 팀
**기간**: 2026.03 ~ 2026.06
**사용 도구**: Cadence Virtuoso Schematic Editor, Yosys, Verilog HDL

## 개요
0.5 V 초저전압에서 0.4 V를 공급하는 **하이브리드 LDO**를 65 nm CMOS 공정 기반으로 설계했습니다.
부하 변동 구간에서는 디지털 루프가 빠르게 복구(Coarse)하고, 정상 상태에서는 아날로그 루프가 PMOS body-bias를 조절해 리플을 제거(Fine)합니다.
외부 클럭 없이 이벤트 발생 시에만 내부 오실레이터가 동작하는 Event-Driven Self-Clocked 구조입니다.

## 문제 정의
- 0.5 V에서는 아날로그 LDO가 voltage headroom을 확보하기 어렵습니다.
- 디지털 LDO는 이산 제어 때문에 limit cycle oscillation과 정상 상태 리플이 생깁니다.
- 목표:
  - 빠른 load transient 복구
  - Zero-ripple 정상 상태
  - 불필요한 클럭 스위칭 제거를 통한 동적 전력 최소화

## 설계 및 구현
**전체 구조**: Window Comparator(Fast / UP / DN) → Adaptive Linear Search Controller(CTRL[15:0]) → 16-PMOS Pass Transistor Array
여기에 Transient Enhancement Unit(TEU), Bulk-driven Error Amplifier(body-bias), Self-Clock Generator를 결합했습니다.

| 블록 | 핵심 설계 |
|---|---|
| **Window Comparator** | V_L = 370 mV, V_H = 430 mV(창 폭 60 mV). 비동기(self-biased diff amp + 증폭단) 구조. `VCO Enable = UP OR (NOT DN)` |
| **Adaptive Linear Search** | Normal: 1비트 shift(UP `ctrl>>1 \| 8000`, DN `ctrl<<1`). Fast(V_OUT < 350 mV): 4비트 점프(`ctrl>>4 \| F000`)로 Binary형 탐색 |
| **Pass Transistor Array** | Thermometer 16개 PMOS, 가중치 8·8·8·8 / 4·4·4·4 / 2·2·2·2 / 1·1·1·1(총 60 unit), unit 216.6 µA → 디지털만 약 13 mA, body-bias 포함 최대 19 mA |
| **TEU** | 계층형 comparator 3개(355 / 350 / 345 mV)가 2 / 4 / 8 unit PMOS 게이트를 클럭 없이 직접 구동(최대 3.03 mA) |
| **Analog Error Amp** | Bulk-driven OTA 입력단(non-tailed) + partial positive feedback(m = 0.7) + self-cascode + CS gain stage + Miller compensation. HVT PMOS array의 body 전압 조절 |
| **Self-Clock Generator** | `EN = NAND(DN, NOT UP)`, NAND 1개 + 인버터 3단 current-starved 링 오실레이터, 약 537 MHz |

**동작 모드**: Analog(창 내부) / Fast(심각한 undershoot) / UP(완만한 undershoot) / DN(overshoot).
Digital coarse와 analog fine이 번갈아 동작하는 **Ping-Pong** 방식입니다.

## 결과
| 항목 | 값 |
|---|---|
| 공정 / V_IN / V_OUT | 65 nm CMOS / 0.5 V / 0.4 V |
| 최대 부하 전류 / C_L | 19 mA / 50 pF |
| 정적 소비 전류 (I_Q) | 299 µA |
| T_coarse / T_settling | 12 ns / 14 ns 이내 |
| Undershoot / Overshoot (13.2 mA/10 ns 스텝) | 123 mV / 약 97 mV |
| 최악 droop (5 mA → 19 mA 스텝) | 약 176 mV |
| PSRR | −32 dB @ 100 kHz |
| Analog loop gain / Phase margin | 40.658 dB / 60.55° |

## 배운 점 / 의의
- Digital coarse + analog body-bias fine의 역할 분담으로 limit cycle oscillation을 원천 차단하면서, 빠른 과도 응답과 zero-ripple을 동시에 달성했습니다.
- 한계:
  - Body 제어 시 누설 전류
  - 0.5 V 초과 공급 시 source-body 순방향 바이어스 방지용 clamp 회로 필요
  - Miller cap으로 인한 slew rate 제한
  - 최대 19 mA의 부하 전류 한계
- 향후 과제: HVT 스위치 어레이 면적 최적화, 실제 칩 제작을 통한 교차 검증

> 본인 담당 역할: [추가 정보 필요]
