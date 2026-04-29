## testtool

> PTZ 카메라의 제어와 테스트 업무를 효율화·자동화하는 통합 도구를 개발하여 업무 생산성을 향상시킨다.

# RULES_SET : TestTool 프로젝트 개발 규칙

## 🎯 목적
PTZ 카메라의 제어와 테스트 업무를 효율화·자동화하는 통합 도구를 개발하여 업무 생산성을 향상시킨다.

## 📚 배경
다양한 카메라 모델(Uncooled, DRS, FineTree, NYX, MiniGimbal, Multi, CTEC)의 프로토콜이 분산되어 있어 통합 관리 요구가 높다. 현재는 모델별로 별도 도구·별도 작업이 필요하여 효율성이 저하되고 있다.

## 🎯 목표
단일 인터페이스로 모든 모델의 PTZ 제어를 실현하고, 카메라 테스트·제어 작업 시간을 **70% 단축**한다. GUI에서의 RTSP 스트리밍·로그 관리를 구현하여 현장 검증을 간소화한다.

## 📦 범위
GUI 기반 테스트 도구(Python, Tkinter, PyQt5, OpenCV, VLC). RTSP 스트리밍 표시와 PTZ 제어. 모델별 명령어 자동화·로그 저장. API 연동·클라우드 연동은 본 단계 대상 외.

## ⛔ 제약 조건
기존 통신 프로토콜과의 호환성을 유지한다. 4채널 동시 스트리밍에서도 실시간성을 보장한다. 멀티스레드 환경에서 UI 블로킹을 발생시키지 않는다. 릴리스 후에도 신규 모델 추가가 용이한 확장성을 보장한다.

## ⚖ 판단 기준(우선순위)
1. **안정성**(통신 오류·크래시 방지)
2. **성능**(응답 속도·프레임 유지)
3. **호환성**(기존 카메라 프로토콜 대응)
4. **사용성**(직관적인 UI/UX)
5. **확장성**(신모델 추가의 용이성)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chalie00) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
