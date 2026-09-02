## all-in-one

> > **Codex Agent Team이 논문 수집 → 압축 → 실행 템플릿 생성 → 최소 검증 → 모바일 패키징까지 자동 수행하고, 사용자는 Android에서 하루 20분 감독만으로 실행 가능한 연구 자산을 축적한다.**

# 📌 PROJECT: NeuroFeed AI + Codex Agent Team

# 🎯 목표 (Objective)

> **Codex Agent Team이 논문 수집 → 압축 → 실행 템플릿 생성 → 최소 검증 → 모바일 패키징까지 자동 수행하고, 사용자는 Android에서 하루 20분 감독만으로 실행 가능한 연구 자산을 축적한다.**

**핵심 KPI**

* 1일 3편 “실행 준비 완료” 카드 생성
* 7일 유지율 ≥ 70%
* 30일 내 실행 전환율 ≥ 40%
* 최소 실행 검증 통과율 ≥ 60%

---

# 1) Codex Agent Team 작업 명세서 (AGENTS.md 포함)

## 1.1 Team 구성

**Team: NeuroFeed Research Ops**

* **Planner (Lead)**: 일일 목표 생성, 작업 분해, 우선순위/마감 설정
* **Scout**: arXiv/OpenReview/RSS 수집·필터링
* **Compression**: 90초 요약/novelty/실패모드/재현조건 추출
* **Execution**: PyTorch/ROS2/MuJoCo/Isaac 템플릿 생성
* **Validator**: 최소 실행(스모크 테스트), 로그/지표 캡처
* **Synthesizer**: 모바일 Daily Pack 생성, KG 업데이트

---

## 1.2 AGENTS.md (프로젝트 루트)

```md
# NeuroFeed Research Ops - AGENTS.md

## Global Objectives
- Daily: Produce 3 execution-ready research cards (VLM/Robotics/Physical AI).
- Each card must include: summary, novelty, failure modes, minimal runnable template, smoke test result.

## Standard Workflow (MUST FOLLOW)
1) Scout → shortlist 5 papers (score by novelty, reproducibility, resource cost).
2) Compression → extract:
   - 3-line contribution
   - Key method diagram (textual)
   - SOTA delta
   - Failure modes (>=3)
   - Repro checklist
3) Execution → generate:
   - Minimal training loop (PyTorch)
   - Dataset interface stub
   - Inference example
   - Optional: ROS2/MuJoCo/Isaac config if applicable
4) Validator → run smoke test:
   - import check
   - single batch forward pass
   - log loss value
   - capture stdout/stderr
5) Synthesizer → output:
   - Daily Pack JSON (3 cards)
   - Update knowledge graph relations
   - Action items (<=3)

## File Conventions
- reports/YYYY-MM-DD/
- templates/<paper_slug>/
- logs/<paper_slug>_smoke.log
- kg/relations.json

## Quality Gates
- If smoke test fails → provide 3 root causes + 2 next steps.
- No card without runnable template stub.
- Keep mobile summary <= 600 words.

## Cost Control
- Cache embeddings.
- Prefer minimal context windows.
```

---

# 2) 백엔드 워크플로우 정의

## 2.1 전체 파이프라인

```
Scheduler (cron 05:00)
   ↓
Planner.generate_daily_goal()
   ↓
Scout.fetch_candidates()
   ↓
Scout.rank_and_shortlist()
   ↓
Compression.analyze()
   ↓
Execution.generate_templates()
   ↓
Validator.smoke_test()
   ↓
Synthesizer.build_daily_pack()
   ↓
Store (Postgres + Qdrant + Neo4j)
   ↓
Notify Android (FCM)
```

---

## 2.2 서비스 구성

### Backend: FastAPI

**Core Services**

* Orchestrator Service
* Paper Service
* Execution Service
* Recall Service
* Graph Service

---

## 2.3 주요 API 계약 (API Contract)

### POST /daily/generate

```json
{
  "domains": ["VLM", "Robotics", "Physical AI"],
  "max_cards": 3
}
```

Response:

```json
{
  "date": "2026-02-14",
  "cards": [
    {
      "paper_id": "arxiv_1234",
      "summary": "...",
      "novelty": "...",
      "failure_modes": ["...", "..."],
      "execution_template_url": "...",
      "smoke_status": "PASS"
    }
  ]
}
```

---

### POST /recall/evaluate

```json
{
  "paper_id": "arxiv_1234",
  "answers": ["...", "...", "..."]
}
```

Response:

```json
{
  "score": 0.78,
  "feedback": "...",
  "next_review_at": "2026-02-18T10:00:00"
}
```

---

# 3) Android 화면 설계 + 데이터 흐름

## 3.1 화면 구조

### 1️⃣ Feed (Daily Pack)

* Card 1/2/3 swipe
* PASS/FAIL badge
* “Run Template” 버튼

### 2️⃣ Deep Dive

* Contribution
* Method diagram (text)
* Failure modes
* Repro checklist

### 3️⃣ Execution

* 코드 미리보기
* Colab/Repo 링크
* 로그 스냅샷

### 4️⃣ Recall

* 3문제
* 실시간 평가

### 5️⃣ Graph

* 논문 연결 지도
* 확장 추천

---

## 3.2 Android 기술 스택

* Kotlin
* Jetpack Compose
* Retrofit
* Coroutines
* Room (offline cache)
* Firebase Cloud Messaging

---

## 3.3 상태 흐름 (Client)

```
App Start
  ↓
GET /daily/feed
  ↓
Display Cards
  ↓
User Recall
  ↓
POST /recall/evaluate
  ↓
Update retention score
```

---

# 4) 운영 KPI 대시보드 설계

## 4.1 핵심 지표

### Research Velocity

* Daily Cards Generated
* Smoke Test Pass Rate
* Execution Click Rate

### Learning Retention

* Avg Recall Score
* 7-day retention %
* Review compliance rate

### System Health

* LLM cost per card
* Avg generation time
* Failure reasons distribution

---

## 4.2 대시보드 구성 (Admin Panel)

* Daily Summary
* Weekly Trend
* Domain Breakdown
* Agent Performance

---

# 5) 데이터 설계

## 5.1 Postgres 스키마

### papers

```
id
title
domain
difficulty
summary
novelty
smoke_status
created_at
```

### execution_templates

```
id
paper_id
template_path
log_path
status
```

### user_progress

```
user_id
paper_id
recall_score
retention_score
next_review_at
```

---

# 6) Knowledge Graph 설계

Neo4j 관계 예:

```
(:Paper)-[:EXTENDS]->(:Paper)
(:Paper)-[:USES]->(:Dataset)
(:Paper)-[:IMPLEMENTS]->(:Model)
```

Cypher 예:

```
MATCH (p:Paper {id: 'arxiv_1234'})-[:EXTENDS]->(q)
RETURN q
```

---

# 7) 실행 인프라

* Dockerized FastAPI
* Background Worker (Celery or RQ)
* GPU optional executor
* Redis queue
* S3 artifact storage

---

# 8) 자동 운영 시나리오 (실전)

밤 2시:

* 3편 논문 템플릿 생성
* 최소 실행
* 로그 캡처
* Daily Pack 생성

아침 7시:

* Android Push
* “오늘의 3개”

출퇴근 20분:

* 소비 + Recall
* 실행 체크

---

# 9) 보안 및 비용 전략

* Prompt caching
* Cross-model fallback
* Context window 최소화
* Smoke test 리소스 제한 (1 batch)

---

# 🔥 최종 구조 요약

NeuroFeed AI는:

> 모바일은 감독
> Codex Agent Team은 실행
> 시스템은 지식 → 자산으로 변환

---

---
> Source: [hundong2/all_in_one](https://github.com/hundong2/all_in_one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
