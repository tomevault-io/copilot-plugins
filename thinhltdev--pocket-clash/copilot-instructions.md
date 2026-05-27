## pocket-clash

> > Đây là file context master. Claude Code đọc file này đầu tiên mỗi session để hiểu toàn bộ dự án.

# Pocket Clash — Project Context for Claude Code

> Đây là file context master. Claude Code đọc file này đầu tiên mỗi session để hiểu toàn bộ dự án.
> Cập nhật file này khi có quyết định lớn hoặc thay đổi architecture.

---

## 1. Project Overview

**Tên game:** Pocket Clash (tạm thời — có thể đổi)
**Thể loại:** Mobile Auto-Battler Card Game (như Hearthstone Battlegrounds, Storybook Brawl)
**Platform:** iOS + Android, **OFFLINE-FIRST**
**Engine:** Unity 6 LTS (2D), C#
**Target launch:** Q4 2026 (6 tháng từ tháng 5/2026)
**Team:** Solo developer + freelance support khi cần

---

## 2. Core Gameplay (Quick Summary)

- 1 vs 7 đối thủ async (snapshot từ player pool cùng rank, không phải realtime PvP)
- Mỗi player có hero HP 30
- 7-8 round, mỗi round: Shop Phase (30s mua/đặt thẻ) → Combat Phase (auto-battle 15-30s)
- Board 3x2 (6 slot: 3 frontline, 3 backline)
- Shop có 5 slot, refresh free đầu round, reroll tốn 1 gold
- Match ~8-10 phút
- Mục tiêu: là người cuối cùng còn sống (HP > 0)

**3-Star System:** 3 thẻ giống nhau → ghép thành thẻ 2-sao (1.5x power). 3 thẻ 2-sao → 3-sao (2.5x power + ability bonus).

**Synergy:** 6 traits chính (Warrior, Mage, Beast, Undead, Holy, Rogue). Sở hữu 2/4/6 unit cùng trait → bonus toàn team.

---

## 3. Content Scope (Launch Day)

- **50 unit cards** (T1:13, T2:10, T3:9, T4:7, T5:6, T6:5)
- **8 playable heroes** với hero power riêng
- **6 traits** với synergy bonus 3 bậc
- **12 ability keywords** (Battlecry, Deathrattle, Taunt, Divine Shield, Poison, Heal, Buff, Debuff, Summon, Reborn, Windfury, Reflect)

**Single source of truth:** File `Pocket_Clash_CardDB_v1.xlsx` (sẽ export sang JSON để Unity load).

---

## 4. Technical Architecture

### Engine & Tools
- **Unity 6 LTS** (chọn version cụ thể: 6000.0.x)
- **Visual Studio 2022** hoặc **Rider** cho C# (recommended Rider nếu có budget)
- **Git + Git LFS** (cho asset binary)
- **GitHub Private Repo**

### Project Folder Structure
```
/Assets
  /Scripts
    /Core              # Game logic core
      CombatSystem.cs
      ShopManager.cs
      EconomyManager.cs
      GameStateManager.cs
    /Data              # ScriptableObject definitions
      CardData.cs
      HeroData.cs
      TraitData.cs
      AbilityData.cs
    /UI                # UI controllers
      /Screens
      /Popups
      /Components
    /AI                # Bot AI for async opponents
      BotAI.cs
      BotDifficulty.cs
    /Meta              # Meta-game systems
      BattlePassManager.cs
      RankedManager.cs
      ProgressionManager.cs
    /Services          # Third-party integrations
      AnalyticsService.cs
      IAPService.cs
      AdsService.cs
      RemoteConfigService.cs
  /Resources
    /Cards             # JSON card database
    /Heroes
    /Audio
  /Art
    /Sprites
    /Animations
    /UI
  /Prefabs
  /Scenes
    MainMenu.unity
    Match.unity
    Tutorial.unity
```

### Third-Party SDKs
- **Firebase**: Analytics, Remote Config, Crashlytics (free tier)
- **AppLovin MAX**: Ad mediation (rewarded, interstitial, banner)
- **Unity IAP**: In-app purchase
- **GameAnalytics**: Backup analytics
- **DOTween**: Animation library (free)
- **Newtonsoft.Json**: JSON parsing

### Async PvP Implementation
Không có server realtime. Cơ chế:
1. Mỗi match, save "snapshot" board cuối mỗi round vào Firebase Firestore
2. Khi cần đối thủ, fetch snapshot từ pool cùng rank
3. Simulate combat **local** trên máy player
4. Early launch: dùng AI bot snapshot (80% bot, 20% real) → grow dần

---

## 5. Design Pillars (Đừng quên)

1. **Strategic Depth, Casual Friction** — quyết định ý nghĩa, UI tối giản
2. **Short Session, Long Progression** — 8-10 phút/match, meta progression hàng tháng
3. **Watch & React, Not Click & Click** — auto-combat, 80% suy nghĩ, 20% xem
4. **Fair F2P, Ethical Monetization** — KHÔNG bán power, chỉ cosmetic + convenience + content

---

## 6. Monetization Stack (Ads-Heavy Hybrid)

### Rewarded Video Ads (60-70% revenue)
- x2 reward sau match
- Free chest mỗi 4h (6 lần/ngày)
- Reroll shop free (mỗi round)
- +1 battle pass tier (1 lần/ngày)
- x2 daily login
- Hero shard pack (1 lần/ngày)

### Interstitial Ads
- Sau mỗi 3 matches
- Skip cho user mua Remove Ads ($3.99)
- KHÔNG hiển thị 24h đầu tiên (boost D1 retention)

### IAP Catalog
- Starter Pack $0.99 (one-time)
- Remove Ads $3.99
- Battle Pass $4.99 / Bundle $9.99
- Gem Pack S/M/L: $1.99 / $4.99 / $9.99
- Mega Pack $19.99
- Skin Bundle $4.99

---

## 7. Coding Conventions

### C# Style
- **PascalCase** cho class, method, public properties
- **camelCase** cho local variables, private fields (prefix `_` cho private fields)
- **UPPER_SNAKE_CASE** cho const
- **XML doc comments** cho public APIs

### Architecture Principles
- **ScriptableObject-driven data** (cards, heroes, abilities — không hardcode trong C# class)
- **Event-driven combat** (UnityEvent hoặc custom event bus, tránh tight coupling)
- **Deterministic combat** (same input → same output, dùng seed cho RNG)
- **State machine** cho match flow (Lobby → Shop → Combat → Result → NextRound)
- **MVC** cho UI (View không tự ý gọi logic, chỉ raise event)

### File Naming
- 1 file = 1 class chính
- Tên file = tên class (CardData.cs chứa class CardData)
- Folder organize theo feature, không theo type

### Testing
- **Unit test** cho combat logic (NUnit, đảm bảo deterministic)
- **Edit Mode tests** cho data validation
- Không cần Play Mode test cho MVP

---

## 8. Production Roadmap

| Tháng | Mục tiêu | Deliverable |
|---|---|---|
| 1 | Design + paper prototype | GDD final, 50 thẻ playtest giấy |
| 2 | Core gameplay Unity | Playable prototype vs AI |
| 3 | Meta game + AI bot | Feature-complete, no art |
| 4 | Art, UI, audio polish | Looks-final game |
| 5 | Monetization + soft launch | Production build, soft launch VN/PH |
| 6 | Iterate + hard launch | Global launch |

**Hiện tại:** Tháng 1, Week 1 (Design phase)

---

## 9. Naming Conventions (Game Content)

- **Card ID**: `C001`, `C002`, ... `C050`
- **Hero ID**: `H01` ... `H08`
- **Trait**: PascalCase, single word (Warrior, Mage, Beast, Undead, Holy, Rogue)
- **Ability**: PascalCase, single word (Battlecry, Deathrattle, Taunt, ...)
- **Card name (display)**: Tiếng Việt với phong cách dân gian/fantasy lai

---

## 10. KPI Targets (Soft Launch)

| Metric | Min | Good | Best |
|---|---|---|---|
| D1 retention | 30% | 40% | 50% |
| D7 retention | 12% | 18% | 25% |
| D30 retention | 4% | 8% | 12% |
| ARPDAU (US) | $0.08 | $0.15 | $0.30 |
| ARPDAU (SEA) | $0.03 | $0.06 | $0.12 |
| IAP conversion | 1.5% | 3% | 5% |

---

## 11. Files & Resources

- **GDD chi tiết**: `Pocket_Clash_GDD_v1.docx` (16 chương)
- **Card database**: `Pocket_Clash_CardDB_v1.xlsx` (50 cards, 8 heroes, 6 traits, 12 abilities)
- **Repo (sẽ tạo)**: GitHub private
- **Project board**: Notion / Trello

---

## 12. Instructions cho Claude Code (Quan Trọng)

### Khi tôi yêu cầu viết code mới:
1. **Đọc CLAUDE.md trước** (file này)
2. **Hỏi clarify nếu unclear** thay vì assume
3. **Follow folder structure** ở section 4
4. **Follow coding convention** ở section 7
5. **Document mỗi class** với XML comment
6. **Suggest test** cho combat logic mới

### Khi tôi yêu cầu refactor:
- Giải thích lý do refactor trước khi làm
- Backup file cũ trước (Git commit trước khi sửa)
- Test sau refactor không bị regression

### Khi tôi yêu cầu thêm feature:
- Check feature có nằm trong scope MVP không (xem GDD)
- Nếu out-of-scope, **gợi ý đưa vào post-launch** thay vì làm ngay
- Estimate effort (giờ/ngày) trước khi bắt đầu

### Khi tôi yêu cầu fix bug:
- Reproduce bug trước
- Suggest root cause analysis
- Fix với minimal change, không refactor luôn
- Document fix trong commit message

### Không làm:
- ❌ Tạo file không cần thiết (luôn ưu tiên edit file có sẵn)
- ❌ Tạo README/docs nếu user không yêu cầu
- ❌ Refactor "preemptive" (chỉ refactor khi user yêu cầu)
- ❌ Thêm dependency mới không hỏi
- ❌ Suggest "best practice" mà out-of-scope

---

## 13. Vocabulary (Tiếng Việt)

Khi tôi nói tiếng Việt:
- "thẻ" = card
- "đơn vị" = unit (thẻ đặt trên board)
- "hồ" hoặc "shop" = card shop
- "lượt" = round
- "vòng đánh" hoặc "combat" = combat phase
- "kỹ năng" = ability
- "đặc tính" hoặc "trait" = trait
- "đồng thuận" hoặc "synergy" = trait synergy
- "anh hùng" hoặc "hero" = playable hero

---

## Updates Log
- **2026-05-14**: File khởi tạo (v1.0). GDD + Card Database hoàn thành.

---
> Source: [thinhltdev/pocket-clash](https://github.com/thinhltdev/pocket-clash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
