# Dary: Automated Property Outreach System

## What is Dary?

Dary (دارى) is an automation tool built for real estate agents and property managers in Arabic-speaking markets. Instead of manually reaching out to every potential tenant or buyer, Dary lets you send a single phone number to a Telegram bot and the system handles everything else — it checks for duplicates, sends a professional Arabic WhatsApp message, logs the result, and reports back to you instantly.

**Built with:** [n8n](https://n8n.io) · Telegram Bot API · WhatsApp via Evolution API · Google Sheets

---

## How It Works (User Flow)

1. You send a phone number to the Dary Telegram bot.
2. The system checks if that number was already successfully contacted.
   - **If yes (duplicate):** You get a "Number already exists ⚠️" reply — nothing else happens.
   - **If no:** The system sends an Arabic outreach WhatsApp message to that number.
3. The result (Done / Fail) is saved to your Google Sheet and reported back to you on Telegram.

<!-- IMAGE: telegram-bot-demo.png — A screenshot showing the Telegram chat: admin sends a phone number, bot replies with "Done ✅" or "Number already exists ⚠️" -->

---

## Technical Architecture

The workflow is built in n8n and handles retries for failed messages and prevents duplicate records:

1. **Trigger** — Receives a phone number sent to the Telegram Bot.
2. **Duplicate Check** — Queries Google Sheets for an existing row with that phone number and a `"Done"` status.
   - **Duplicate found:** Sends "Number already exists ⚠️" notification to admin via Telegram and stops.
   - **Not a duplicate:** Continues to the outreach phase.
3. **Outreach Phase**
   - **Data Retrieval:** Fetches any existing row for the number to check for a prior `"Fail"` status.
   - **Message Generation:** A JavaScript node builds a personalized Arabic outreach message.
   - **WhatsApp Delivery:** Sends the message via Evolution API. The node uses `continueErrorOutput` so delivery failures are captured rather than crashing the workflow.
4. **Conditional Data Persistence**
   - **Update:** If the number existed with a `"Fail"` status, its row is updated with the new result.
   - **Append:** If the number is brand new, a fresh row is added.
5. **Reporting** — Final status (Done / Fail / Duplicate) is sent back to the admin via Telegram.

<!-- IMAGE: n8n-workflow-canvas.png — Full n8n canvas screenshot showing all nodes and their connections -->
![Dary Workflow](Dary%20Workflow%20Image.png)

---

## Core Components

| Node | Function | Service |
| :--- | :--- | :--- |
| **Telegram Trigger** | Entry point — receives the phone number from admin. | Telegram API |
| **Check if number exists** | Looks up the phone number with `"Done"` status in Google Sheets. | Google Sheets API |
| **If (row_number check)** | Routes flow: duplicate path vs. outreach path. | n8n If Node |
| **Get row(s) in sheet** | Fetches existing row to detect prior `"Fail"` status. | Google Sheets API |
| **Code in JavaScript** | Generates the Arabic WhatsApp outreach message. | n8n Code Node |
| **Send text (Evolution API)** | Delivers the WhatsApp message; failures go to error output. | Evolution API |
| **Merge / Merge1** | Consolidates parallel paths before persistence and reporting. | n8n Merge Node |
| **If1 (Fail check)** | Decides whether to update an existing row or append a new one. | n8n If Node |
| **Update / Add row** | Writes the final result back to Google Sheets. | Google Sheets API |
| **Send a text message** | Reports the final status to the admin on Telegram. | Telegram API |

---

## Outreach Message

The system sends the following Arabic message to each new lead via WhatsApp:

> سلام عليكم 👋
>
> إحنا فريق دارى، منصة بتساعد أصحاب العقارات فى تأجير وتسويق وحداتهم والوصول لعدد أكبر من المستأجرين.
>
> تواصلنا معك لأن عندنا اهتمام نضيف عقارك على المنصة ونساعد فى تسويقه بشكل أفضل.
>
> لو حابب تعرف تفاصيل أكتر عن طريقة العمل أو عندك أى استفسار، ابعتلنا رسالة وهنكون سعداء نوضح كل حاجة.
>
> تشرفنا بالتواصل معك 🌟

To customize this message, edit the `Code in JavaScript` node inside your n8n workflow.

---

## Setup & Deployment

### Prerequisites

- A running **n8n** instance (self-hosted or cloud). Tested on n8n node versions: Telegram Trigger v1.2, Google Sheets v4.7.
- A **Telegram Bot** created via [@BotFather](https://t.me/BotFather).
- A **Google Cloud** project with the Sheets API enabled and OAuth2 credentials.
- A running **Evolution API** instance with a connected WhatsApp session named `Dary`.

### Step 1 — Import the Workflow

1. Open your n8n instance.
2. Go to **Workflows → Import from file**.
3. Select `Dary.json` from this repository.

<!-- IMAGE: n8n-import-workflow.png — Screenshot of the n8n "Import from file" dialog with Dary.json selected -->

### Step 2 — Configure Credentials

In n8n, go to **Settings → Credentials** and add the following:

| Credential | Used by |
| :--- | :--- |
| Telegram API (Bot Token) | Telegram Trigger, Send a text message |
| Google Sheets OAuth2 | Check if number exists, Get row(s), Update row, Add row |
| Evolution API | Send text |

### Step 3 — Set Up Your Google Sheet

Create a new Google Sheet and add the following headers **exactly** in row 1 of the first sheet (`gid=0`):

| A | B | C |
| :--- | :--- | :--- |
| `Phone Number` | `Status of Messaging` | `Status of Acceptance` |

<!-- IMAGE: google-sheet-headers.png — Screenshot of the Google Sheet with the three column headers highlighted -->

Then open each Google Sheets node in the workflow and **replace the document ID** with your own spreadsheet's ID (found in its URL: `https://docs.google.com/spreadsheets/d/<YOUR_SHEET_ID>/`).

### Step 4 — Activate the Workflow

Toggle the workflow to **Active** in n8n. The Telegram webhook will be registered automatically.

---

## Usage

1. Open the Telegram bot you configured.
2. Send a phone number in international format, e.g.:
   ```
   201012345678
   ```
3. The bot will reply with one of:
   - `Done` — Message sent successfully and logged.
   - `Fail` — Delivery failed; the number is logged for a future retry.
   - `Number already exists ⚠️` — This number was already successfully contacted.

---

## Error Handling

| Scenario | Behavior |
| :--- | :--- |
| WhatsApp delivery fails | Captured via `continueErrorOutput`; status logged as `"Fail"` in Google Sheets for retry on next attempt. |
| Number already contacted | Duplicate check prevents re-sending; admin notified immediately. |
| Google Sheets API quota exceeded | n8n will surface the error in the execution log; no data is lost. |
| Invalid Telegram token | The Trigger node will fail to register the webhook; check credentials in n8n. |

---

## License

This project is licensed under the terms of the [LICENSE](LICENSE) file included in this repository.
