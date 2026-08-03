## smart-bobber

> - KREFT 브랜드 충전식 스마트 전자찌 컨트롤러 앱

# Smart Bobber - 전자찌 컨트롤 앱

## 제품 개요
- KREFT 브랜드 충전식 스마트 전자찌 컨트롤러 앱
- Flutter (iOS / Android)
- 전자찌(camfishing_float)와 블루투스(BLE)로 연동

## 주요 기능
- 찌 최대 20개 표시 및 개별 제어
- 입질 감지 시 해당 찌 빨간색 발광 + 알림
- 캐미 색상 조정 (레드/그린/블루/옐로우/핑크)
- 밝기 조절 슬라이더
- 입질 감도 조절 슬라이더
- 알림음 선택 (sound_1 ~ sound_5)
- 전체 ON/OFF (ALL ON / ALL OFF)
- 블루투스 페어링 스캐너

## 현재 상태
- UI 구현 완료
- 블루투스 연동: 현재 TCP 소켓 + 시뮬레이션 → 실제 BLE로 교체 예정
- 페어링 스캐너: 현재 더미 데이터 → flutter_blue_plus 패키지로 교체 예정

## 연동 하드웨어 (예정)
- MCU: Nordic nRF52832 (BLE 전용)
- 위치센서: MPU-6050
- LED: WS2812B RGB
- 배터리: 3.7V LiPo 200mAh + USB-C 충전
- 방수: IP67

## 작업 방식
- flutter run 으로 실기기(안드로이드/iOS) 테스트
- BLE 연동 시 flutter_blue_plus 패키지 사용

---
> Source: [intp2581-creator/smart_bobber](https://github.com/intp2581-creator/smart_bobber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
