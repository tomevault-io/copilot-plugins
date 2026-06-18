## openclaw-twitter-skill

> >


# Twitter Post Skill

Standardized 5-step workflow for OpenClaw agents to post on X (Twitter) via browser tools.

## Prerequisites

- The user must already be logged into x.com in the Chrome OpenClaw profile.
- The agent must have access to `browser` tools (navigate, act, type, screenshot, snapshot, wait).

## Quick Start

When the user asks to post to Twitter, follow these steps **exactly in order**:

```
Step 1 → Open https://x.com/compose/post
Step 2 → Verify login (is the compose box visible?)
Step 3 → Type content into the compose box
Step 4 → Screenshot → show user for confirmation
Step 5 → Click Post → verify success
```

**CRITICAL RULES (read before proceeding):**
- NEVER reply to an existing tweet — new post mode only.
- NEVER post without showing the user a screenshot first.
- NEVER retry silently on failure.
- NEVER modify user's content without explicit permission.

## Detailed Instructions

### Step 1: Open Compose Page

```javascript
// ALWAYS use the direct compose URL — never click sidebar buttons
await browser.navigate("https://x.com/compose/post");
await browser.wait(3000); // wait for page load
```

**Why direct URL?** Twitter's sidebar UI changes frequently. The `/compose/post` modal route is the most stable entry point. Sidebar "Post" button, `Ctrl+Enter` shortcut, and floating action buttons all break across UI updates.

### Step 2: Verify Login

After navigation, take a snapshot to check page state:

```javascript
const snapshot = await browser.snapshot();
// Examine the snapshot text for indicators below
```

**Check for these indicators:**

| State | What you'll see in snapshot | Action |
|-------|---------------------------|--------|
| ✅ Logged in | `contenteditable` text area, "Post" button, character counter | Proceed to Step 3 |
| ❌ Not logged in | "Sign in" / "Log in" text, email/password fields | **STOP** — report to user |
| ⚠️ Rate limited | "Rate limit" / "Try again later" / error banner | **STOP** — screenshot and report |
| ⚠️ Account suspended | "Your account is suspended" message | **STOP** — screenshot and report |

**If not logged in, respond with:**
> ⚠️ Twitter 未登录。请先在 Chrome OpenClaw profile 中手动登录 x.com，然后重试。

Do NOT attempt to enter credentials or guess passwords. This is a hard stop.

### Step 3: Type Content

```javascript
// Click the compose text area first to focus it
await browser.act("click the tweet compose text area");
await browser.wait(500);

// Type the content character by character (do NOT paste)
await browser.type("Your tweet content here #OpenClaw");
await browser.wait(1000); // wait for content to settle
```

**Content rules (MUST follow):**

| Rule | Spec |
|------|------|
| Mode | New post ONLY — never reply to existing tweets |
| Length | Maximum 200 characters (leave room for rendering) |
| Emoji | 1–3 emojis, placed naturally (not emoji spam) |
| Hashtags | 1–3 relevant hashtags at the end |
| Language | Match user's language preference |
| Links | Only if the user explicitly provides them |
| Media | Text-only by default (see Media Attachments section below) |

**Content template:**
```
[Core message, 1-2 sentences] [1-2 emoji]

#Tag1 #Tag2
```

**Why type() not paste?** Twitter's CSP and anti-bot measures may block clipboard paste events. `type()` simulates real keyboard input and is consistently more reliable.

### Step 4: Screenshot for Confirmation

```javascript
// MANDATORY — never skip this step
const screenshot = await browser.screenshot();
```

Present the screenshot to the user and say:
> 📸 请确认推文内容是否正确。确认后我将点击发布。

**Wait for user confirmation before proceeding.** If the user wants changes:
1. Select all text in the compose box: `await browser.act("select all text in the compose area");`
2. Delete it: `await browser.act("press Backspace");`
3. Type the new content
4. Screenshot again

### Step 5: Click Post & Verify

```javascript
// Wait for the Post button to be enabled (it takes ~500ms-1s after typing)
await browser.wait(1500);

// Click the Post button
await browser.act('click the "Post" button');

// Wait for the post to be submitted
await browser.wait(3000);

// Take a verification screenshot
const result = await browser.screenshot();
```

**Success indicators (check in screenshot/snapshot):**
- "Your post was sent" toast notification at the bottom
- Compose modal closes and returns to timeline
- A "View" link appears in the toast

**Failure indicators:**
- Post button still visible → it wasn't clicked
- "Something went wrong" toast → Twitter server error
- Error popup / red text → content validation error
- Page unchanged → JavaScript error, nothing happened

**On success, respond:**
> ✅ 推文已发布成功！

**On failure, respond with screenshot:**
> ❌ 发布失败。截图如上，请检查。[具体错误描述]

## Media Attachments (Optional)

If the user wants to attach an image:

```javascript
// After Step 3 (typing content), before Step 4 (screenshot):
await browser.act('click the "Add photos or video" button (media icon)');
await browser.wait(1000);
// The file picker will open — this requires user interaction
// Report to user: "请在弹出的文件选择器中选择要上传的图片"
```

**Limitations:**
- File picker is a native OS dialog — the agent cannot select files programmatically.
- Wait for the image to upload and thumbnail to appear before proceeding.
- If no media button is found, proceed with text-only post.

## Multi-Tweet Handling

If the user provides multiple tweets to post:

1. **Post them ONE AT A TIME** — complete the full 5-step cycle for each.
2. **Wait at least 30 seconds between posts** to avoid rate limiting.
3. **Get confirmation for each tweet separately** — do not batch confirmations.
4. **If any tweet fails, STOP** — do not continue with remaining tweets.

## Error Handling Rules

1. **NEVER retry silently.** If posting fails, report the error with a screenshot immediately.
2. **NEVER guess or fill in credentials.** Login is the user's responsibility.
3. **NEVER post without user confirmation** (Step 4 screenshot approval).
4. **NEVER modify the user's content** without telling them.
5. **Report specific errors:** "Button disabled", "Page not loaded", "Rate limited" — not just "failed".
6. **One retry maximum:** For "Something went wrong" server errors only, wait 30 seconds and retry the full workflow once. If it fails again, stop.

## Common Failure Modes & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Compose box not found | Page didn't fully load | `wait(5000)` then `snapshot()` to check again |
| Post button disabled | Content empty or invalid | Verify text was actually typed via `snapshot()` |
| "Something went wrong" | Twitter server error | Wait 30s, retry once, then report |
| Page shows login form | Session expired | Ask user to re-login in Chrome profile |
| Content truncated | Typed too fast | Add `wait(500)` between segments if content is long |
| Modal doesn't open | URL didn't navigate properly | Try `navigate()` again with a fresh page load |
| "Post" button not found | UI layout changed | Use `snapshot()` to find the actual button text/label |

## Lessons Learned (经验教训)

1. **`/compose/post` is king** — Sidebar "Post" button and keyboard shortcuts are unreliable across UI updates. Always navigate directly.
2. **Type, don't paste** — `type()` simulates human input; paste may be blocked by Twitter's CSP.
3. **Always screenshot before posting** — This is the single biggest reliability improvement. Users catch errors, and you have evidence of what was attempted.
4. **Wait before clicking Post** — The Post button takes 500ms–1s to become active after content is entered. Clicking too early does nothing.
5. **One post per session** — Don't chain multiple posts without spacing. Complete one full cycle before starting another.
6. **Check toast, not URL** — After posting, the URL may not change immediately. Look for the success toast notification.
7. **snapshot() over screenshot() for logic** — Use `snapshot()` (DOM text) for programmatic checks, `screenshot()` (image) for user-facing confirmation.
8. **act() is your friend** — When specific selectors break, describe the action in natural language: `act('click the blue Post button')` is more robust than CSS selectors.

---
> Source: [adminlove520/openclaw-twitter-skill](https://github.com/adminlove520/openclaw-twitter-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
