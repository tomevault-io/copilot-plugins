## ai-weekly-report

> 슈퍼컴퍼니 서비스개발1팀의 주간 업무 보고서를 자동으로 작성하고 팀장에게 발송하는 프로젝트.

# 주간보고서 자동화 — 슈퍼컴퍼니 서비스개발1팀

## 프로젝트 개요
슈퍼컴퍼니 서비스개발1팀의 주간 업무 보고서를 자동으로 작성하고 팀장에게 발송하는 프로젝트.

---

## 팀 정보
- 회사명: 슈퍼컴퍼니
- 팀명: 서비스개발1팀
- 팀장 (보고 수신자): Thomas Moon <thomas@itengineers.net>
- 보고자: Daniel Kim <daniel@itengineers.net>
- 보고 주기: 매주 금요일 오전
- 어투: 격식체

---

## 소스 데이터 위치
- STT 텍스트 파일 (회의 녹음 변환본): C:\Users\Administrator\Documents\회의녹음\
- 회의록 (Google Drive): https://drive.google.com/drive/folders/1G8Ok9vDdaFJc6rU6Hp8HkwtrF_bjiy70
- 업무 결과물 (Google Drive): https://drive.google.com/drive/folders/19NwSJBilPaRL2G0O_h-8oRaBcbkzidQS
- 업무 관련 메일: Gmail (최근 1주일 수발신 메일)

---

## 보고서 양식 & 저장
- 양식 파일: C:\Users\Administrator\Documents\ppt_templates\주간보고서.pptx
- IMPORTANT: 양식 파일 수정·덮어쓰기 절대 금지. 반드시 복사본에 작업할 것.
- 저장 위치: C:\Users\Administrator\Documents\보고서결과\
- 파일명 규칙: 주간보고서_YYYYMMDD_v1.pptx (수정 시 v2, v3으로 버전 올림)
- 이메일 제목 형식: [주간보고] Daniel _ YYYY년 MM월 N주차 업무 요약

---

## 사용하는 Skill
- 주간보고서 작성 전체 워크플로우: @skills/주간보고서/SKILL.md
- 회의록 작성 및 업로드: @skills/회의록/SKILL.md
- PPT 생성·편집: @skills/pptx/SKILL.md

---

## "주간보고서 작성해줘" 실행 워크플로우

IMPORTANT: 아래 7단계 중 1~5단계는 사용자에게 묻지 않고 순서대로 자동 실행한다.
사용자에게 멈추고 확인하는 시점은 6단계(이메일 발송 컨펌) 단 한 번뿐이다.

### Step 1 — 업무 메일 수집 및 정리 [자동 실행]

Gmail MCP로 최근 1주일 업무 관련 메일을 수집한다.

- 수집 기간: 직전 월요일 00:00 ~ 오늘 현재 시각
- 수집 대상: daniel@itengineers.net 수발신 메일 전체
- 제외: noreply, 자동발송, 광고, 구독, 스팸 메일
- 정리 항목: 발신자, 날짜, 제목, 핵심 내용 1~2줄 요약

수집 완료 후 중단 없이 Step 2로 바로 진행한다.

---

### Step 2 — 회의록 누락 확인 및 자동 작성 [자동 실행]

아래 두 소스를 비교해서 회의록이 작성되지 않은 STT 파일을 찾는다.

비교 대상:
- STT 파일 목록: C:\Users\Administrator\Documents\회의녹음\ 폴더의 *_STT원본.txt
- 작성 완료 회의록: Google Drive 회의록 폴더

비교 기준: STT 파일명의 날짜(YYYYMMDD)와 Google Drive 회의록 파일명의 날짜를 대조한다.
예) 20250406_회의_STT원본.txt 에 대응하는 20250406_*.md 가 Google Drive에 없으면 누락으로 판단.

누락 파일이 있는 경우:
@skills/회의록/SKILL.md 를 사용해 해당 STT 파일로 회의록을 작성하고 Google Drive에 업로드한다.
누락 파일이 여러 개면 날짜 순서대로 순차 처리한다.
모든 처리 완료 후 중단 없이 Step 3으로 진행한다.

누락 파일이 없는 경우:
그대로 Step 3으로 진행한다.

---

### Step 3 — 회의 내용 정리 [자동 실행]

Google Drive 회의록 폴더에서 최근 1주일(직전 월요일 ~ 오늘) 날짜의 회의록 파일을 읽는다.

각 회의록에서 아래 항목을 추출해 통합 정리한다:
- 주요 결정사항
- 액션아이템 (담당자 + 기한)
- 다음 주 예정 사항

정리 완료 후 중단 없이 Step 4로 바로 진행한다.

---

### Step 4 — 업무 결과물 확인 및 정리 [자동 실행]

Google Drive 업무 결과물 폴더 (https://drive.google.com/drive/folders/19NwSJBilPaRL2G0O_h-8oRaBcbkzidQS) 에서
최근 1주일(직전 월요일 ~ 오늘) 생성·수정된 파일 목록을 읽는다.

각 파일에서 아래 항목을 추출해 정리한다:
- 완료된 업무명과 산출물
- 진행 중인 업무와 현재 상태
- 수치·지표가 있으면 포함

정리 완료 후 중단 없이 Step 5로 바로 진행한다.

---

### Step 5 — 주간보고서 PPT 작성 [자동 실행]

Step 1~4에서 수집·정리한 내용을 바탕으로 회사 양식 PPT를 작성한다.

- @skills/pptx/SKILL.md 를 활용해 PPT를 생성한다
- 슬라이드 매핑 규칙: @skills/주간보고서/references/slide-mapping.md
- 양식 파일 복사: C:\Users\Administrator\Documents\ppt_templates\주간보고서.pptx
- 저장: C:\Users\Administrator\Documents\보고서결과\주간보고서_YYYYMMDD_v1.pptx
- IMPORTANT: 원본 양식 파일 덮어쓰기 절대 금지
- 슬라이드 추가·삭제·순서·디자인 변경 금지. 텍스트만 채운다
- 어투: 격식체, 슬라이드당 3줄 이내

PPT 작성 완료 후 Step 6으로 진행한다.

---

### Step 6 — 이메일 발송 컨펌 [사용자 확인 — 유일한 멈춤 지점]

PPT 완성 후 반드시 아래 형식으로 정확히 물어본다.
이 단계는 자동 발송 사고를 방지하기 위해 반드시 거쳐야 한다.
IMPORTANT: 컨펌 없이 절대 이메일을 발송하지 않는다.

```
주간보고서가 완성됐습니다.

[이번 주 핵심 요약]
- 주요 업무: (Step 4 결과 중 완료 항목 2~3개)
- 이슈: (Step 1·3 결과 중 이슈 항목, 없으면 "없음")
- 다음 주 계획: (Step 3 결과 중 액션아이템 2~3개)

[발송 정보]
파일: 주간보고서_YYYYMMDD_v1.pptx
수신자: Thomas Moon <thomas@itengineers.net>
제목: [주간보고] Daniel _ YYYY년 MM월 N주차 업무 요약

팀장에게 메일을 발송할까요? (예 / 아니요)
```

→ "예", "응", "보내줘", "발송해줘" → Step 7 진행
→ "아니요" 또는 수정 요청 → 요청 내용 반영 후 파일명 버전 올려 재저장(v2, v3...), Step 6 재실행

---

### Step 7 — 이메일 발송 [사용자 확인 후 실행]

이메일 발송 양식: @skills/주간보고서/references/email-template.md

발송 완료 후 아래 형식으로 완료를 보고하고, 파일을 보관 폴더로 이동한다:

```
발송 완료됐습니다.

수신자: Thomas Moon <thomas@itengineers.net>
발송 시각: YYYY-MM-DD HH:MM
파일 보관: C:\Users\Administrator\Documents\보고서결과\발송완료\주간보고서_YYYYMMDD_v1.pptx
```

---

## 예외 처리

| 상황 | 처리 방법 |
|------|---------|
| Gmail 수집 0건 | "이번 주 관련 메일 없음"으로 기록 후 Step 2로 계속 진행 |
| STT 파일 0개 | "STT 파일 없음"으로 기록 후 Step 3으로 계속 진행 |
| Google Drive 접근 실패 | 오류 내용을 기록하고 해당 소스 제외 후 계속 진행. Step 6에서 사용자에게 고지 |
| PPT 작성 실패 | 오류 내용을 사용자에게 보고하고 중단 |
| Step 6 컨펌에서 "아니요" | 수정 요청 내용 반영 → 버전 올려 재저장 → Step 6 재실행 |
| 이메일 발송 실패 | 오류 내용 보고 후 재시도 여부 확인 |

---

## 절대 하지 말 것
- IMPORTANT: Step 6 컨펌 전 이메일 발송 금지
- IMPORTANT: 개인정보(연락처, 주민번호 등) 보고서 포함 금지
- 원본 양식 파일(ppt_templates/) 수정·덮어쓰기 금지
- 보고서결과/ 폴더 외 다른 위치에 파일 저장 금지
- Step 1~5 진행 중 사용자에게 진행 여부 묻지 말 것

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Asakhan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Asakhan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
