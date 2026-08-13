## soop-stock-market

> - 커밋 메시지(요약)는 한글로 작성한다.

## Git 커밋 규칙

- 커밋 메시지(요약)는 한글로 작성한다.
- 작업(파일 수정)을 마치면 사용자에게 묻지 않고 자동으로 git commit을 수행한다. 커밋이 끝나면 별도로 묻지 않고 바로 git push까지 진행한다(2026-08-03, 사용자가 모든 프로젝트에 이 지침을 명시적으로 요청함).

## 구현 후 검증 필수

- 코드를 구현한 뒤 배포·커밋으로 넘어가기 전에 반드시 검증 단계를 거친다. 필드명·파라미터명·상태값을 "이렇게 생겼겠지"라고 추측하지 말고, 실제로 그 데이터를 쓰는(write) 쪽 소스 코드를 다시 읽어 대조한다.
- 이 프로젝트(및 자매 프로젝트 admin-center)에서 실제로 검증 없이 넘어갔다면 조용히 묻혔을 사례들: 구매 현황 집계에서 `uid` 필드가 실제로는 `requesterUid`였던 걸 가정만 하고 넘어가 5/10개 항목이 누락될 뻔했던 일, 감사 로그 자동 기록 대상 액션 3개가 누락됐던 일, 배너/고정노출/중계방 신청이 최근 리팩터로 이미 신청 즉시 자동 승인되도록 바뀌어 있어 "승인 대기" 큐 UI를 만들어도 절대 나타나지 않는다는 걸 뒤늦게 발견한 일. 전부 실제 소스를 재확인하는 검증 단계에서만 잡을 수 있었다.
- 구체적으로 확인할 것: 문법 검사(`node -c` 등), 새 HTML id/함수명이 실제로 정의·호출 양쪽에 다 있는지 교차 확인, RTDB 규칙 변경은 `--dry-run`으로 먼저 확인, 그리고 무엇보다 새로 읽거나 다루는 RTDB 노드의 필드명은 그 노드를 쓰는 실제 코드를 찾아 대조.

## Firebase 데이터 소모량 경고

- 작업 지시를 받았을 때, 그 작업이 Firebase(Realtime Database 읽기/쓰기, Cloud Functions 호출 등) 관련 데이터 소모량이 더 커지는 방향(예: 폴링 주기 단축, 실시간 리스너 추가, 불필요한 전체 스캔/재조회 증가 등)으로 이뤄질 것 같으면, 바로 작업을 시작하지 말고 먼저 사용자에게 경고하고 확인을 받는다.

## Firebase Functions 배포 주의사항

- 이 Firebase 프로젝트(soop-stock-market)는 "스트리머 배팅시장"이라는 별도 웹앱의 Cloud Functions와 같이 쓰인다(codebase가 둘 다 `default`라 겹친다). 예: adminAdjustBalance, adminLookupUser, addPromotedStreamer, approveChestPurchase, approveStreamerRequest, approveVerification, banAccount, blockNickname, cancelBet, cancelPendingJudgment, claimAttendance, claimJackpotDraw, closeBettingScheduled, closeMarketEarly, dismissChestPurchase, dismissMarketReport, dismissNicknameReport, dismissStreamerRequest, distributeJackpotWeekly, equipSkin, exchangeCurrency, getAdminDashboardStats, getAnomalyMonitor, getExchangeLog, getStatsTimeseries, judgeMarket, onLikeWritten, openChest, placeBet, purchaseSkin, rejectVerification, removePromotedStreamer, reportMarket, reportNickname, reviewProposal, revokeVerification, sendAdminChatMessage, setMinParticipantsOverride, setVerifiedSoopId, submitChestPurchase, submitMarketProposal, submitStreamerRequest, submitVerificationRequest, unbanAccount, unblockNickname, updateProfile, voidMarket. 이 함수들은 이 레포의 `functions/` 소스에는 없지만 실제 배포된 상태로 존재하며, 절대 삭제하면 안 된다.
- `firebase deploy --only functions`처럼 함수명을 지정하지 않고 전체 배포하면, 로컬 소스에 없는 위 함수들을 삭제 대상으로 인식해 삭제를 시도한다(대화형 확인 없이는 실제로 삭제되진 않지만, 자동화 스크립트 등에서는 위험하다). 반드시 `firebase deploy --only functions:<함수명>,functions:<함수명>,...` 형태로 변경/추가한 함수만 지정해서 배포한다.
- 위 목록은 이 문서를 마지막으로 갱신한 시점(2026-08-02) 기준이라 시간이 지나면 정확하지 않을 수 있다. 배포 전 `firebase deploy --only functions --project soop-stock-market`을 한번 시도해 "found in your project but do not exist in your local source code" 목록이 뜨면(실제 삭제되기 전에 중단됨), 그게 지금 시점의 정확한 목록이다.

---
> Source: [neezu-crypto/soop-stock-market](https://github.com/neezu-crypto/soop-stock-market) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
