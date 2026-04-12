## okrbest

> 커스텀 상태 (Custom Status) 기능 관련 규칙


# 커스텀 상태 (Custom Status) 도메인

## 개요

사용자가 자신의 상태를 커스텀 이모지와 텍스트로 표시하는 기능입니다.
"회의 중", "점심 시간", "재택 근무" 등 현재 상태를 팀원들에게 알릴 수 있습니다.

## 관련 파일

| 영역 | 파일 |
|------|------|
| API | `server/channels/api4/status.go` |
| 비즈니스 로직 | `server/channels/app/status.go` |
| 모델 | `server/public/model/status.go` |
| 커스텀 상태 | `server/public/model/custom_status.go` |
| 웹앱 컴포넌트 | `webapp/channels/src/components/custom_status/` |
| 상태 드롭다운 | `webapp/channels/src/components/status_dropdown/` |

## CustomStatus 모델

```go
// server/public/model/custom_status.go
type CustomStatus struct {
    Emoji     string    `json:"emoji"`
    Text      string    `json:"text"`
    Duration  string    `json:"duration"`
    ExpiresAt time.Time `json:"expires_at"`  // time.Time 타입!
}

// Duration 유효 값
var validCustomStatusDuration = map[string]bool{
    "thirty_minutes": true,
    "one_hour":       true,
    "four_hours":     true,
    "today":          true,
    "this_week":      true,
    "date_and_time":  true,
}

// 상수
const (
    UserPropsKeyCustomStatus = "customStatus"
    CustomStatusTextMaxRunes = 100
    MaxRecentCustomStatuses  = 5
    DefaultCustomStatusEmoji = "speech_balloon"
)
```

## Status (온라인 상태) 모델

```go
type Status struct {
    UserId         string `json:"user_id"`
    Status         string `json:"status"`
    Manual         bool   `json:"manual"`
    LastActivityAt int64  `json:"last_activity_at"`
    DNDEndTime     int64  `json:"dnd_end_time"`
    PrevStatus     string `json:"prev_status,omitempty"`
}

// 온라인 상태
const (
    StatusOffline         = "offline"
    StatusAway            = "away"
    StatusDnd             = "dnd"     // Do Not Disturb
    StatusOnline          = "online"
    StatusOutOfOffice     = "ooo"     // Out of Office
)
```

## 주요 API

| 엔드포인트 | 메서드 | 설명 |
|-----------|--------|------|
| `/api/v4/users/{user_id}/status` | GET | 사용자 상태 조회 |
| `/api/v4/users/{user_id}/status` | PUT | 상태 변경 |
| `/api/v4/users/status/ids` | POST | 여러 사용자 상태 조회 |
| `/api/v4/users/{user_id}/status/custom` | PUT | 커스텀 상태 설정 |
| `/api/v4/users/{user_id}/status/custom` | DELETE | 커스텀 상태 해제 |
| `/api/v4/users/{user_id}/status/custom/recent` | GET | 최근 커스텀 상태 목록 |

## 커스텀 상태 설정

```go
func (a *App) SetCustomStatus(rctx request.CTX, userId string, cs *model.CustomStatus) *model.AppError {
    // 1. 유효성 검사
    if cs.Emoji == "" && cs.Text == "" {
        return model.NewAppError(...)
    }
    
    // 2. 만료 시간 계산
    if cs.Duration != "" {
        cs.ExpiresAt = calculateExpiry(cs.Duration)
    }
    
    // 3. 사용자 Props에 저장
    user, err := a.GetUser(userId)
    if err != nil {
        return err
    }
    
    user.Props["customStatus"] = cs.ToJson()
    
    // 4. DB 업데이트
    _, err = a.UpdateUser(rctx, user, false)
    if err != nil {
        return err
    }
    
    // 5. 최근 사용 목록에 추가
    a.addToRecentCustomStatuses(userId, cs)
    
    // 6. WebSocket 브로드캐스트
    a.BroadcastStatus(&model.Status{UserId: userId, ...})
    
    return nil
}
```

## 상태 만료 처리

```go
// 커스텀 상태 만료 Job
func (a *App) ClearExpiredCustomStatuses() {
    // ExpiresAt이 지난 커스텀 상태 자동 해제
    users, _ := a.GetUsersWithExpiredCustomStatus()
    
    for _, user := range users {
        a.RemoveCustomStatus(user.Id)
    }
}
```

## DND (방해 금지) 모드

```go
// DND 설정
func (a *App) SetDND(userId string, endTime int64) *model.AppError {
    status := &model.Status{
        UserId:     userId,
        Status:     model.StatusDnd,
        Manual:     true,
        DNDEndTime: endTime,
        PrevStatus: currentStatus.Status,
    }
    
    return a.SaveStatus(status)
}

// DND 자동 해제
func (a *App) ClearExpiredDND() {
    statuses, _ := a.GetExpiredDNDStatuses(model.GetMillis())
    
    for _, status := range statuses {
        // 이전 상태로 복원
        status.Status = status.PrevStatus
        status.DNDEndTime = 0
        a.SaveStatus(status)
    }
}
```

## WebSocket 이벤트

| 이벤트 | 설명 |
|--------|------|
| `status_change` | 온라인 상태 변경됨 |
| `custom_status_updated` | 커스텀 상태 업데이트됨 |

## 기본 제공 상태

| 이모지 | 텍스트 | 설명 |
|--------|--------|------|
| 📅 | In a meeting | 회의 중 |
| 🤒 | Out sick | 병가 |
| 🏠 | Working from home | 재택 근무 |
| 🏖️ | On vacation | 휴가 중 |
| 🚗 | Commuting | 출퇴근 중 |

## 슬래시 명령어

```
/away     - 자리 비움 상태로 변경
/online   - 온라인 상태로 변경
/offline  - 오프라인 상태로 변경
/dnd      - 방해 금지 모드 (DND)
```

## 상태 표시 위치

| 위치 | 표시 내용 |
|------|----------|
| 프로필 사진 옆 | 온라인 상태 점 (초록/노랑/빨강/회색) |
| 사용자 팝오버 | 커스텀 상태 (이모지 + 텍스트) |
| 사이드바 DM | 온라인 상태 점 |
| 멤버 목록 | 온라인 상태 점 + 커스텀 상태 |

## 설정

```json
{
    "ServiceSettings": {
        "EnableCustomStatus": true,
        "EnableCustomStatusRecentStatuses": true
    }
}
```

---

## Q&A 목록

| 질문 | 파일 |
|------|------|
| (새 질문이 생기면 여기에 추가) | |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/okrbest)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/okrbest)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
