# Telegram-Chatbot-using-n8n-Tasks

## Primary Task: A/B Test Chatbot in n8n
A Telegram chatbot that automatically assigns new users to a **Control Group** (generic welcome) or **Test Group** (guided 3-step onboarding), using Statsig for A/B group assignment and Google Sheets for persistent session state.
Control Group — receives a generic welcome message
Test Group — goes through a guided 3-step onboarding flow

1. Build a Telegram Chatbot A/B Test in n8n
Develop a simple Telegram chatbot that automatically assigns each new user to one of two groups:
You must use Statsig as the A/B testing provider for group assignment.

3. Event Logging
Log every group assignment and key user event in a structured manner to enable downstream analysis.

5. Evaluation Plan
Define a rigorous evaluation framework that includes:
A primary metric (leading indicator correlated with long-term retention) 
Guardrail metrics (e.g., bot block rate) to detect unintended negative effects 
Secondary metrics (e.g., Day-7 return rate, onboarding completion rate) 
A pre-committed decision framework outlining how results will be interpreted and acted upon

## Secondary Task: Health Chatbot

Build a simple health chatbot on Telegram where users can:
• Log meals 
• Track their day 
• Edit logged meals 
• Delete logged meals
This can be kept lightweight in scope. Focus on clean command handling and reliable data storage.


---

### Prerequisites
- [n8n](https://n8n.io/) installed locally or hosted (e.g. n8n.cloud)
- A Telegram Bot Token from [@BotFather](https://t.me/BotFather)
- A [Statsig](https://statsig.com/) account with a Server Secret Key
- A Google account with Google Sheets API access

### Steps

**1. Clone / Import the Workflow**
```bash
# If running n8n locally
npx n8n
# Then import telegram_ab_workflow_v3.json via:
# n8n UI → Top Menu → Import from File
```

**2. Set Up Credentials in n8n**

Go to **Settings → Credentials** and create:

| Credential Name | Type | Value |
|---|---|---|
| `Telegram Bot` | Telegram API | Your Bot Token from @BotFather |
| `Statsig API Key` | Header Auth | Header: `STATSIG-API-KEY`, Value: your Server Secret |
| `Google Sheets` | Google Sheets OAuth2 | Sign in with your Google account |

**3. Create the Google Sheet for health bot**

Create a new Google Sheet at [sheets.google.com](https://sheets.google.com) with this header row in the first tab (named `user_states`):

```
| user_id | userName | meal | timestamp |
```

**4. Create the Statsig Feature Gate**

1. Go to [console.statsig.com](https://console.statsig.com)
2. Create a Feature Gate named exactly: `telegram_onboarding_test`
3. Add a rule: **Everyone → 50% Pass**
4. Add primary metrics ( daily_stickiness, d1_retention_rate), guardrail metrics (bot_blocked) and secondary metrics (onboarding_completed) 
5. Enable the gate

**5. Activate the Workflow**

- Open the imported workflow in n8n for each task
- Assign all credentials to the relevant nodes
- Toggle the workflow **Active**

**6. Test the Bot**

Open Telegram, find your bot and type:
```
/start       → Step 1: who are you?
student      → Step 2: your goal?
learn        → Step 3: notifications?
yes          → ✅ Onboarding complete!
/exit        → Reset your session
```

---

## Environment Variables

When running n8n locally via Docker or `.env`, set these:

| Variable | Description | Example |
|---|---|---|
| `TELEGRAM_BOT_TOKEN` | Bot token from @BotFather | `123456:ABC-DEF...` |
| `STATSIG_SERVER_SECRET` | Statsig Server Secret Key | `secret-abc123...` |
| `GOOGLE_SHEET_ID` | ID from your Google Sheet URL | `12p24JWjf0xLSDB...` |
| `N8N_WEBHOOK_URL` | Public base URL of your n8n instance | `https://your-n8n.cloud` |
| `N8N_ENCRYPTION_KEY` | n8n encryption key for credentials | any random string |

> In n8n Cloud, credentials are stored securely in the UI — no `.env` file needed.

---

Every incoming Telegram message triggers a **fresh workflow execution**. Google Sheets acts as the persistent state store, tracking each user's current onboarding step and assigned group across executions.

---


## Assumptions and Trade-offs

**Assumptions**
- Each Telegram `userId` is a stable, unique identifier suitable for A/B assignment (true for all Telegram users).
- Users complete onboarding in a single language — keyword matching (`student`, `learn`, `yes`) is case-insensitive but English-only.
- The experiment runs on a single n8n instance — no horizontal scaling concerns for the Google Sheets state store at this traffic level.
- Statsig's feature gate provides sufficiently random 50/50 splitting based on `userId` hash; no additional randomisation is needed.

**Trade-offs**

| Decision | Trade-off |
|---|---|
| Google Sheets as state store | Simple and free, but not suitable for high traffic (>100 concurrent users). A proper database (Postgres, Redis) would be used in production. |
| Keyword-based onboarding answers | Easy to implement and debug, but fragile — users must type exact words. A button-based reply keyboard would be more user-friendly. |
| Statsig Feature Gate (not full Experiment) | Gates are simpler to set up but provide less built-in statistical analysis than Statsig's Experiment product. Compensated by the rigorous external evaluation plan. |
| Single n8n workflow for all logic | Keeps the project self-contained and easy to import, but makes the workflow complex. A production system would split into sub-workflows. |
| n8n Cloud hosting | Zero infrastructure setup, but workflow static data doesn't persist between executions — hence the switch to Google Sheets for state. |

---

## Approximate Time Breakdown

| Section | Time Spent |
|---|---|
| Telegram AB Test Bot using n8n and Statsig | ~5 hours |
| Telegram Health Bot | ~35 minutes |



