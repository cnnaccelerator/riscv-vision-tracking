# FPGA RISC-V Vision Tracking & Servo Control System

## Overview

DE2 FPGA 기반 RV32I Single-Cycle RISC-V 프로세서와 HuskyLens 비전센서를 연동하여,
객체의 위치 정보를 실시간으로 수신하고 2축 Pan/Tilt Servo를 제어하는 임베디드 시스템입니다.

비전센서에서 UART로 전달되는 X/Y/Width/Height/ID 데이터를 FPGA에서 파싱하고,
RISC-V 프로세서가 MMIO를 통해 데이터를 읽어 객체 위치에 따라 Servo 각도를 결정합니다.

본 프로젝트에서는 센서 입력부터 Processor 연산, PWM 기반 구동부 제어까지
HW/SW Co-Design 방식으로 통합했습니다.

## My Role

- Verilog RTL 설계
- RV32I Single-Cycle RISC-V Processor 구현
- MMIO Address Map 설계
- HuskyLens UART Interface 구현
- Servo PWM Generator 구현
- RISC-V Assembly 기반 제어 알고리즘 작성
- 시스템 통합 및 기능 검증

## System Architecture

HuskyLens
→ UART RX / Packet Parser
→ Sensor Register
→ RV32I Processor
→ MMIO
→ Servo PWM
→ Pan / Tilt Servo

## Key Implementation

### 1. RV32I Single-Cycle Processor

RV32I 기반 Single-Cycle Processor를 Verilog로 구현했습니다.

주요 구성:
- Controller
- Register File
- ALU
- Immediate Extend
- Program Counter
- Instruction / Data Memory

### 2. Vision Sensor UART Interface

HuskyLens와 9600 bps UART 통신을 구성했습니다.

- FPGA Clock: 50 MHz
- UART Baud Rate: 9600 bps
- CLKS_PER_BIT: 5208
- Sensor Data: X, Y, Width, Height, ID

주기적으로 요청 Packet을 전송하고 응답 Packet을 파싱하여
센서 데이터를 Register에 저장하도록 구현했습니다.

### 3. Memory-Mapped I/O

RISC-V Processor가 Sensor 및 Servo를 Memory Address 방식으로 제어하도록 MMIO를 구성했습니다.

예시:
- SENSOR_X
- SENSOR_Y
- SENSOR_WIDTH
- SERVO_PAN
- SERVO_TILT

이를 통해 Assembly Software가 일반 Load/Store 명령어로 주변장치를 제어할 수 있도록 했습니다.

### 4. Servo PWM Control

50 Hz PWM을 생성하여 Pan/Tilt Servo를 제어했습니다.

센서의 X/Y 위치를 구간별로 판단하고,
Assembly Program에서 Servo 각도를 결정하도록 HW/SW 역할을 분리했습니다.

### 5. RISC-V Assembly Control

RV32I에는 `bgt` 명령어가 없기 때문에
`slt` + `beq` 조합으로 조건 비교 로직을 구현했습니다.

Sensor X/Y 값을 구간별로 판단하여
Servo Pan/Tilt Angle을 변경하는 제어 루프를 작성했습니다.

## Result

- Vision Sensor 기반 실시간 객체 위치 추적
- UART Sensor Data Parsing
- RV32I Processor 기반 제어 판단
- 2축 Pan/Tilt Servo 자동 제어
- 객체 소실 시 Servo Neutral Position 복귀
- Sensor → Processor → Actuator End-to-End 동작 검증

## Tools

- Verilog HDL
- RISC-V RV32I Assembly
- FPGA
- UART
- MMIO
- PWM
- Python

## What I Learned

이 프로젝트를 통해 단순 RTL 모듈 설계를 넘어,
센서 인터페이스, Processor, Memory-Mapped I/O, Software 제어,
실제 구동부까지 하나의 시스템으로 통합하는 과정을 경험했습니다.
