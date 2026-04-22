## stm32-drv8311p-bldc-motor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

STM32H743 기반 BLDC(Brushless DC) 모터 제어 보드 펌웨어 프로젝트입니다. Texas Instruments의 DRV8311 모터 드라이버를 사용하여 BLDC 모터를 제어하며, ADS8638을 통한 다채널 ADC 샘플링 기능을 포함합니다.

**타겟 MCU**: STM32H743VIHX (ARM Cortex-M7, 480MHz)
**빌드 도구**: STM32CubeIDE with ARM GCC toolchain
**하드웨어**: 듀얼 DRV8311 모터 드라이버가 탑재된 커스텀 BLDC 제어 보드

## 빌드 명령어

### 프로젝트 빌드

STM32CubeIDE의 make 기반 빌드 시스템을 사용합니다. 환경 변수 설정이 필요합니다:

```bash
# 환경 변수 설정 (일반적인 설치 경로)
export STM32_CUBE_IDE_PATH="/c/ST/STM32CubeIDE_1.19.0"
export STM32_CUBE_CLT_PATH="/c/ST/STM32CubeCLT_1.19.0"

# Debug 빌드
cd Debug
PATH="${STM32_CUBE_CLT_PATH}/GNU-tools-for-STM32/bin:$PATH" \
  "${STM32_CUBE_IDE_PATH}/STM32CubeIDE/plugins/com.st.stm32cube.ide.mcu.externaltools.make.win32_2.2.0.202409170845/tools/bin/make.exe" \
  -j all

# Debug 빌드 클린
PATH="${STM32_CUBE_CLT_PATH}/GNU-tools-for-STM32/bin:$PATH" \
  "${STM32_CUBE_IDE_PATH}/STM32CubeIDE/plugins/com.st.stm32cube.ide.mcu.externaltools.make.win32_2.2.0.202409170845/tools/bin/make.exe" \
  clean
```

### VS Code 통합

빌드 태스크는 `.vscode/tasks.json`에 설정되어 있습니다:
- **Build (Debug)**: 기본 빌드 태스크 (Ctrl+Shift+B)
- **Build (Release)**: Release 빌드
- **Clean (Debug/Release)**: 빌드 산출물 정리
- **Flash (ST-Link)**: ST-Link를 통한 펌웨어 플래싱
- **Build and Flash**: 빌드 후 플래싱 자동 실행

빌드 결과물: `Debug/bldc_h743_v101.elf`

## 아키텍처

### 하드웨어 주변장치

**통신 인터페이스**:
- UART1, UART2: DMA 및 컬러 printf를 지원하는 디버그/통신 인터페이스
- FDCAN1: CAN-FD 버스 인터페이스
- SPI1: DRV8311 모터 드라이버 제어 및 ADS8638 ADC 통신
- SPI2, SPI3: 추가 DRV8311 모터 드라이버 제어 (듀얼 드라이버용 다중 CS 핀)
- SPI4: 예비 SPI 인터페이스

**모터 제어**:
- TIM1, TIM2, TIM3, TIM4, TIM5, TIM8: 모터 제어용 6개 PWM 타이머 채널
- TIM15: DRV8311 동기화를 위한 PWM_SYNC 신호 생성
- TIM6, TIM7: 범용 타이머

**센싱**:
- ADC1, ADC2, ADC3: 전류/전압 감지용 DMA 지원 온칩 3채널 ADC
- ADS8638: SPI1을 통한 외부 8채널 12비트 SAR ADC (CS: PD10)

**GPIO**:
- 엔코더 입력: 5개 엔코더 Z-인덱스 신호 (ENC1_Z ~ ENC5_Z)
- 듀얼 DRV8311 드라이버 및 ADC용 다중 SPI CS 핀
- 상태 표시용 LED 출력 (LD1, LD2)

### 소프트웨어 아키텍처

**메인 애플리케이션 흐름**:
1. `main.c`: HAL 초기화, 주변장치 설정, 메인 루프에서 `run_proc()` 호출
2. `user_def.c`: `init_proc()` 및 `run_proc()` 포함 - 메인 애플리케이션 로직
3. `user_def.h`의 테스트 함수들이 다양한 하드웨어 검증 루틴 제공

**주요 모듈**:

**DRV8311 모터 드라이버** (`drv8311.c/h`):
- Texas Instruments 3상 BLDC 게이트 드라이버 (통합 벅 레귤레이터)
- SPI 기반 레지스터 설정 및 상태 모니터링
- 다중 PWM 모드 지원: 6-PWM, 3-PWM, tSPI PWM 생성 모드
- 주요 함수: `DRV8311_Init()`, `DRV8311_Start()`, `DRV8311_SetSpeed()`, `DRV8311_ReadFaultStatus()`
- `DRV8311_HandleTypeDef` 구조체를 통한 하드웨어 추상화
- 완전한 레지스터 문서는 `.doc/DRV8311_Register_Summary.md` 참조

**ADS8638 ADC** (`ads8638.c/h`):
- 프로그래밍 가능한 입력 범위를 가진 8채널 12비트 SAR ADC
- 자동 스캔 및 수동 채널 선택 모드
- 범위 설정: 0-7.5V, 0-6.25V, 0-3.75V 등
- 통신 검증용 진단 함수
- 주요 함수: `ADS8638_Init()`, `ADS8638_ReadAllChannels()`, `ADS8638_SetChannelRange()`

**통신 모듈** (`user_com.c/h`):
- 컬러 UART/USB printf 함수: `printf_UARTC(uart, color, fmt, ...)`
- 라인 버퍼링을 사용하는 DMA 기반 수신
- 안정적인 통신을 위한 링 버퍼 구현
- 컬러 코드: PR_RED, PR_GRN, PR_YEL, PR_BLE, PR_CYN, PR_WHT
- 라인 기반 명령 파싱: `UART2_GetLine()`, `USB_GetLine()`

**ADC 함수** (`adc_func.c/h`):
- STM32 내장 ADC용 헬퍼 함수

**테스트 함수** (`user_def.c`):
하드웨어 검증용으로 사용 가능한 테스트 함수들:
- `test_uart2()`: UART 통신 테스트
- `test_adc()`: 내장 ADC 샘플링 테스트
- `test_ads8638()`: 외부 ADS8638 ADC 테스트
- `test_drv8311()`: 단일 DRV8311 모터 드라이버 테스트
- `test_dual_drv8311()`: 듀얼 모터 드라이버 동시 제어
- `test_motor_run_spi2()`: SPI2를 통한 모터 구동 테스트
- `test_motor_tspi_pwmgen_spi1()`: tSPI PWM 생성 모드 테스트

### 파일 구조

```
Core/
├── Inc/                    # 헤더 파일
│   ├── main.h              # GPIO 핀 정의가 포함된 메인 헤더
│   ├── user_def.h          # 테스트 함수 프로토타입
│   ├── drv8311.h           # DRV8311 드라이버 인터페이스
│   ├── ads8638.h           # ADS8638 ADC 인터페이스
│   ├── user_com.h          # 통신 유틸리티
│   ├── adc_func.h          # ADC 헬퍼 함수
│   └── ring_buffer.h       # 통신용 링 버퍼
├── Src/                    # 소스 파일
│   ├── main.c              # 메인 애플리케이션 진입점
│   ├── user_def.c          # 애플리케이션 로직 및 테스트
│   ├── drv8311.c           # DRV8311 드라이버 구현
│   ├── ads8638.c           # ADS8638 드라이버 구현
│   ├── user_com.c          # 통신 함수
│   ├── stm32h7xx_it.c      # 인터럽트 핸들러
│   └── stm32h7xx_hal_msp.c # HAL MSP 초기화
├── Startup/                # 시작 어셈블리 코드
Drivers/
├── STM32H7xx_HAL_Driver/   # STM32 HAL 라이브러리
├── CMSIS/                  # CMSIS 헤더
Debug/                      # 빌드 출력 디렉토리
├── makefile                # 자동 생성된 makefile
├── bldc_h743_v101.elf      # 빌드 결과물
.vscode/
├── tasks.json              # 빌드 태스크 설정
├── c_cpp_properties.json   # IntelliSense 설정
STM32H743VIHX_FLASH.ld      # FLASH용 링커 스크립트
bldc_h743_v101.ioc          # STM32CubeMX 프로젝트 파일
```

## 중요 사항

### STM32CubeMX 통합

이 프로젝트는 STM32CubeMX(`.ioc` 파일)로 관리됩니다. 코드 재생성 시:
- 사용자 코드는 `/* USER CODE BEGIN */`과 `/* USER CODE END */` 블록 사이에 보존됩니다
- 이 블록 외부의 자동 생성 코드는 수정하지 마세요
- 주변장치 초기화는 자동 생성 섹션에 있습니다

### DRV8311 모터 드라이버

- 보드의 듀얼 DRV8311 칩은 SPI1/SPI2로 제어됩니다
- CS 핀: SPI1용 PD10 (device 1), PD14 (device 2)
- CS 핀: SPI2용 PE15 (device 1), PE11 (device 2)
- 동기화를 위한 TIM15_CH1을 통한 PWM_SYNC 신호
- 레지스터 쓰기 시 SYS_CTRL 레지스터 언락 필요 (쓰기 키 0x5)
- 동작 후 항상 폴트 상태 확인 필요
- tSPI PWM 생성 모드 지원 (외부 PWM 불필요)

### ADS8638 외부 ADC

- PD10의 CS를 통해 SPI1로 연결
- 내부 2.5V 레퍼런스
- 채널별 범위 설정 가능
- 자동 스캔 모드는 활성화된 채널을 순차적으로 읽음
- 단일 채널 읽기를 위한 수동 모드

### 빌드 시스템

- Makefile은 STM32CubeIDE에 의해 자동 생성됨
- 의존성: `sources.mk`, `objects.mk`, 서브디렉토리 makefile
- 링커 스크립트: `STM32H743VIHX_FLASH.ld`
- 컴파일러: `arm-none-eabi-gcc` (GNU Tools for STM32 13.3.rel1)
- Flash/RAM 레이아웃은 링커 스크립트에 정의됨

### 디버깅

- 디버그 설정: `bldc_h743_v101 Debug.launch`
- ST-Link 디버거 지원
- SWD 인터페이스
- 플래싱용 STM32CubeProgrammer CLI 사용: `STM32_Programmer_CLI.exe -c port=SWD -w Debug/bldc_h743_v101.elf`

## 개발 워크플로우

1. `/* USER CODE BEGIN/END */` 섹션 또는 커스텀 모듈에서 코드 수정
2. VS Code 태스크 또는 명령줄 make를 사용하여 빌드
3. 에러 확인 - 일반적인 문제:
   - 주변장치 초기화 누락
   - DMA 버퍼 정렬 (H7의 캐시 일관성을 위해 `__attribute__((aligned(32)))` 사용)
   - 클럭 설정 불일치
   - 모터 드라이버와의 SPI 타이밍 문제
4. ST-Link를 통해 하드웨어에 플래싱
5. UART2를 통해 디버그 출력 모니터링 (컬러 printf 사용 가능)

## 일반적인 코딩 패턴

### 새로운 모터 제어 테스트 추가

1. `user_def.h`에 함수 선언
2. `user_def.c`에 구현
3. SPI 및 타이머로 DRV8311 핸들 초기화
4. PWM 모드 및 파라미터 설정
5. 제어 루프에서 폴트 상태 모니터링
6. 디버그 출력을 위해 `printf_UARTC()` 사용

### 외부 ADC 채널 읽기

1. ADS8638 초기화: `ADS8638_Init(&hadc, &hspi1)`  // SPI1 사용
2. 채널 범위 설정: `ADS8638_SetChannelRange(&hadc, channel, range)`
3. 채널 읽기: `ADS8638_ReadAllChannels(&hadc)` 또는 `ADS8638_ReadChannel()`
4. `hadc.voltage[channel]`을 통해 데이터 접근

### SPI 통신 패턴

```c
// CS Low
HAL_GPIO_WritePin(CS_PORT, CS_PIN, GPIO_PIN_RESET);
// SPI 전송
HAL_SPI_TransmitReceive(&hspi, tx_buf, rx_buf, len, timeout);
// CS High
HAL_GPIO_WritePin(CS_PORT, CS_PIN, GPIO_PIN_SET);
```

## 참고 자료

- DRV8311 데이터시트: Texas Instruments SLVSFN2B
- STM32H743 레퍼런스 매뉴얼: STMicroelectronics RM0433
- ADS8638 데이터시트: Texas Instruments
- 레지스터 문서: `.doc/DRV8311_Register_Summary.md`
- 빌드는 tasks.json 을 참고해서 해.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/trionking) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
