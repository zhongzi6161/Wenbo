---
name: daily-briefing-assistant
description: >-
  A comprehensive daily briefing assistant. Run all modules at once or trigger each independently. Covers: today's date and weather (with saved city preference), top 5 priority items from email and calendar, unread email reply drafts (TO only, CC skipped), SAP stock prices (NYSE and Frankfurt), and a worldwide holiday check. Trigger phrases: "Hi SAP" (full briefing in English), "你好 SAP" (full briefing in Chinese), "check weather", "today's priorities", "draft email replies", "SAP stock", "today's holiday".
allowed-tools: web_search list_emails get_email search_emails list_calendar_events read_file write_file
metadata:
  version: 1.0.0
  tags: productivity daily weather email calendar stocks briefing
---

# Daily Briefing Assistant

## Language Rule

Determine the response language from the trigger phrase:

| Trigger | Response Language |
|---|---|
| "Hi SAP" | English — all output in English |
| "你好 SAP" | Chinese — all output in simplified Chinese (简体中文) |
| All other module triggers | English (default) |

Apply the selected language to all module headers, labels, tips, and status messages throughout the briefing.

**Exception — Module 3 (Email Reply Drafts):** Reply draft content is **always written in English**, regardless of the briefing language or the original email's language.

---

## Trigger Conditions

| Trigger | Module |
|---|---|
| "Hi SAP" | All modules (full briefing, English) |
| "你好 SAP" | All modules (full briefing, Chinese) |
| "check weather", "today's weather", "weather" | Module 1 only |
| "today's priorities", "top 5 today", "what should I focus on", "my schedule" | Module 2 only |
| "draft replies", "unread email replies", "email drafts" | Module 3 only |
| "SAP stock", "SAP share price", "stock price" | Module 4 only |
| "today's holiday", "is today a holiday", "holiday check" | Module 5 only |

---

## City Configuration

Before running Module 1, check if `daily-briefing-config.json` exists using `read_file`.

**If found:** Parse and use the `city` field as the default city.

**If not found (first run):**
1. Ask for the user's city in the current response language:
   - English: "Welcome to your Daily Briefing Assistant! To provide weather updates, I need to know your current city. What city are you in?"
   - Chinese: "欢迎使用每日简报助手！为了提供天气更新，我需要知道您所在的城市。请问您在哪个城市？"
2. Wait for the user's reply.
3. Save using `write_file` at path `daily-briefing-config.json`:
   ```json
   { "city": "USER_CITY" }
   ```
4. Confirm in the current response language.

**To change the city:** If the user says "change my city to [CITY]" or "更改城市为[城市]", overwrite `daily-briefing-config.json` with the new city and confirm.

---

## Module 1 — Weather Briefing

**Step 1:** Note today's date from system context (format: `Wednesday, August 19, 2026`).

**Step 2:** Read default city from `daily-briefing-config.json`.

**Step 3:** Call `web_search` with query: `"[CITY] weather today forecast temperature humidity wind"`

**Output (English):**
```
📅  Date:         [Full date]
📍  Location:     [City]

🌤   Weather:      [Condition]
🌡  Temperature:  [Low]°C – [High]°C
💧  Humidity:     [X]%
💨  Wind:         [Direction] at [Speed] km/h
💡  Tip:          [Practical suggestion based on conditions]
```

**Output (Chinese):**
```
📅  日期：     [完整日期]
📍  地点：     [城市]

🌤   天气：     [天气状况]
🌡  温度：     [最低]°C – [最高]°C
💧  湿度：     [X]%
💨  风速：     [风向]，[速度] 公里/小时
💡  建议：     [基于天气状况的实用建议，如带伞、穿衣等]
```

---

## Module 2 — Today's Top 5 Priorities

Run **in a single message with 2 parallel tool calls**:

- **Call A:** `list_emails` — `folder: "inbox"`, `onlyUnread: true`, `top: 20`
- **Call B:** `list_calendar_events` — `from: [today 00:00 ISO 8601]`, `to: [today 23:59 ISO 8601]`, `limit: 20`

Wait for both results, then analyze:
- **Emails:** Urgency signals — deadlines, action requests, important senders, time-sensitive language.
- **Calendar:** Time-sensitive meetings, preparation needed, scheduling conflicts.

Synthesize and rank the **top 5 highest-priority items**. If fewer than 5 items exist, list only what is available.

**Output (English):**
```
## 🎯 Today's Top 5 Priorities

1. **[Item title]** — [Brief context, source (email/calendar), and action needed]
2. **[Item title]** — [...]
3. **[Item title]** — [...]
4. **[Item title]** — [...]
5. **[Item title]** — [...]
```

**Output (Chinese):**
```
## 🎯 今日前 5 大优先事项

1. **[事项标题]** — [简要说明，来源（邮件/日历）及所需行动]
2. **[事项标题]** — [...]
3. **[事项标题]** — [...]
4. **[事项标题]** — [...]
5. **[事项标题]** — [...]
```

---

## Module 3 — Unread Email Reply Drafts

> **Note:** Joule can read your emails but cannot save directly to the Outlook Drafts folder. Reply drafts will be displayed in chat for you to review and copy into your email client.

**Step 1:** Call `list_emails` with `folder: "inbox"`, `onlyUnread: true`, `top: 10`.

**Step 2:** For each unread email, call `get_email` with the message ID to retrieve full details including the recipient fields (toRecipients, ccRecipients).

**Step 3:** Filter — keep only emails where the user appears in the **TO** field as a direct recipient. Skip any email where the user is only listed in CC. If you cannot determine the recipient fields, include the email and note the uncertainty.

If no qualifying emails found:
- English: "No unread emails found where you are a direct recipient (TO). Emails where you are CC'd only have been skipped."
- Chinese: "未找到您作为直接收件人（TO）的未读邮件。仅抄送给您的邮件已跳过。"

**Step 4:** For each qualifying email (up to 5), generate a professional reply draft. **Always write the draft content in English**, regardless of the briefing language or the original email's language. Professional business tone, concise (3–5 sentences).

**Output per email (always in English):**
```
---
📧 Reply Draft — "[Original Subject]"
From: [Sender Name]  |  Received: [Date]
Summary: [One-line summary of the original email]

Dear [Sender Name],

[Draft reply content — always in English]

Best regards,
[Sign off]
---
```

After all drafts:
- English: "Please review these drafts and copy them into your email client to send."
- Chinese: "请查看以上草稿，并将其复制到您的邮件客户端发送。"

---

## Module 4 — SAP Stock Price

Run **in a single message with 2 parallel web searches**:

- **Search A:** `"SAP SE stock price NYSE today"`
- **Search B:** `"SAP SE Aktie Frankfurt XETRA Kurs aktuell"`

**Output (English):**
```
## 📈 SAP Stock Price

🇺🇸  NYSE (SAP):       $[Price] USD   [+/-X.XX | +/-X.XX%]
🇩🇪  Frankfurt (SAP):  €[Price] EUR   [+/-X.XX | +/-X.XX%]

Source: [Data source] · As of: [Timestamp]
```

**Output (Chinese):**
```
## 📈 SAP 股价

🇺🇸  纽交所（SAP）：   $[价格] 美元   [+/-X.XX | +/-X.XX%]
🇩🇪  法兰克福（SAP）：  €[价格] 欧元   [+/-X.XX | +/-X.XX%]

数据来源：[来源] · 截至：[时间戳]
```

If data is unavailable for a market:
- English: "Price data unavailable for [market] — please check your brokerage platform."
- Chinese: "[市场]价格数据暂不可用，请查看您的证券平台。"

---

## Module 5 — Holiday Check

Call `web_search` with query: `"public holidays today [current full date] worldwide international"`

Analyze:
- If today is a holiday in any country or culture, write a 2–4 sentence introduction: name, country/culture, significance, and notable traditions.
- Cover up to 3 holidays if multiple exist.
- If none found:
  - English: "No major holidays found for today."
  - Chinese: "今日未发现重要节假日。"

**Output (English):**
```
## 🎉 Today's Holidays

**[Holiday Name]** — [Country / Culture]
[2–4 sentence description]

**[Holiday Name 2]** — [Country / Culture]  (if applicable)
```

**Output (Chinese):**
```
## 🎉 今日节假日

**[节假日名称]** — [国家 / 文化]
[2–4 句介绍]

**[节假日名称 2]** — [国家 / 文化]（如适用）
```

---

## Full Briefing Mode

When the user says **"Hi SAP"** or **"你好 SAP"**:

- Trigger **"Hi SAP"** → use **English** for all output
- Trigger **"你好 SAP"** → use **Chinese (简体中文)** for all output (except Module 3 draft content, which is always in English)

**Step 1 — Print header:**
- English: `# 🌅 Daily Briefing — [Today's Full Date]`
- Chinese: `# 🌅 每日简报 — [今日完整日期]`

**Step 2 — Run in a single message with 6 parallel tool calls:**
- `web_search`: `"[CITY] weather today forecast"`
- `web_search`: `"SAP SE stock price NYSE today"`
- `web_search`: `"SAP SE Aktie Frankfurt XETRA Kurs aktuell"`
- `web_search`: `"public holidays today [full date] worldwide"`
- `list_emails`: `folder: "inbox"`, `onlyUnread: true`, `top: 20`
- `list_calendar_events`: today's full day range, `limit: 20`

Wait for all results, then assemble and present in this order:
1. Module 1 — Weather
2. Module 2 — Top 5 Priorities
3. Module 4 — SAP Stock
4. Module 5 — Holidays

**Step 3 — Run Module 3 (Email Drafts) separately** as it requires reading individual emails.

**Step 4 — Footer:**
- English: `✅ Daily Briefing complete.`
- Chinese: `✅ 每日简报完成。`