# 🌅 Daily Briefing Assistant

A skill for [Joule Work Desktop](https://www.sap.com/) that delivers a comprehensive daily briefing — weather, priorities, email drafts, SAP stock prices, and worldwide holidays — in a single command.

Supports **bilingual output**: reply in English with `Hi SAP`, or in Chinese (简体中文) with `你好 SAP`.

---

## ✨ Features

| Module | Description |
|---|---|
| 🌤 Weather | Current conditions and forecast for your saved city |
| 🎯 Top 5 Priorities | AI-ranked priorities synthesized from unread emails and today's calendar |
| 📧 Email Reply Drafts | Ready-to-send reply drafts for unread emails where you are a direct recipient (TO only) |
| 📈 SAP Stock Price | Real-time SAP SE stock price on NYSE (USD) and Frankfurt XETRA (EUR) |
| 🎉 Holiday Check | Today's public holidays and observances worldwide |

---

## 🚀 Trigger Phrases

### Full Briefing (all modules)

| Trigger | Language |
|---|---|
| `Hi SAP` | English |
| `你好 SAP` | Chinese (简体中文) |

### Individual Modules

| Trigger | Module |
|---|---|
| `check weather` / `today's weather` | Weather only |
| `today's priorities` / `top 5 today` / `what should I focus on` | Top 5 Priorities only |
| `draft replies` / `unread email replies` | Email Reply Drafts only |
| `SAP stock` / `SAP share price` | Stock Price only |
| `today's holiday` / `is today a holiday` | Holiday Check only |

---

## ⚙️ Setup

### 1. Install the skill

Import `SKILL.md` into Joule Work Desktop via the **Extensions** panel, or clone this repository and load the skill from the local directory.

### 2. Configure your city

On first run of the Weather module, you will be prompted to enter your city. The skill saves it to `daily-briefing-config.json` in your working directory for future use.

To change your city at any time, say:
- **English:** `change my city to [City]`
- **Chinese:** `更改城市为[城市]`

### 3. Required tools

The skill uses the following built-in Joule tools — no additional connectors required:

- `web_search` — weather, stock prices, holidays
- `list_emails`, `get_email`, `search_emails` — inbox reading and reply drafting
- `list_calendar_events` — today's schedule
- `read_file`, `write_file` — city preference storage

---

## 📋 Notes

- **Email reply drafts** are always written in **English**, regardless of the briefing language.
- Drafts are displayed in chat only — Joule cannot save directly to the Outlook Drafts folder. Copy the draft into your email client to send.
- Emails where you are CC'd only are automatically skipped for reply drafts.
- Stock price data is sourced from public web search and may be delayed by up to 15 minutes.

---

## 📁 File Structure

```
daily-briefing-assistant/
├── SKILL.md                     # Skill instructions (agentskills.io format)
├── README.md                    # This file
└── LICENSE                      # MIT License
```

---

## 🔧 Customization

You can customize this skill via the **Extensions** panel in Joule Work Desktop:

- Add new modules (e.g. news summary, currency rates)
- Change output format or language defaults
- Adjust priority-ranking logic for emails and calendar items
- Add or modify trigger phrases

---

## 👤 Author

**Wenbo Bi** 
wenbo.bi@sap.com
Business Processes Senior Consultant, SAP Business Network for Supply Chain
SAP (China) Co., Ltd., Dalian

---

## 📄 License

Apache 2.0. See [LICENSE](LICENSE) for details.
