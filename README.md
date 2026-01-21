# Intelligent Document Processing (IDP) with Slack Alerts (n8n)

Automated pipeline that watches a Google Drive folder for new files, downloads them, extracts insights via AI image analysis, runs custom JavaScript logic, then routes outcomes to either:
- Create a record in Airtable (e.g., for tracking/triage), or
- Send a Slack alert message (e.g., when a condition fails / needs attention).

## Demo / Screenshots
## n8n Workflow
![n8n Workflow](https://github.com/zukui1984/n8n-Intelligent-Document-Processing-Slack-Airtable/raw/master/images/n8n_workflow.jpg)

## Slack Chatbot results
![Slack](https://github.com/zukui1984/n8n-Intelligent-Document-Processing-Slack-Airtable/blob/master/images/slack.jpg)

## Architecture (n8n nodes)
1. **Google Drive Trigger** — fires when a new file is created in a target folder
2. **Download file** — pulls the file content
3. **Analyze an image** — AI analysis/extraction
4. **Code in JavaScript** — transforms/enriches output (risk scoring, normalization, etc.)
5. **IF** — routes based on your condition
6. **Airtable: Create a record** (true branch)
7. **Slack: Send a message** (false branch)

## Features
- End-to-end automated processing (no manual steps once active)
- Conditional routing with audit-friendly record creation (Airtable)
- Real-time notifications via Slack
- Easy to extend (additional branches, channels, storage, tagging)

## Requirements
- n8n (self-hosted `http://localhost:5678` / cloud).
- Google Drive account + folder to watch
- Slack App (Bot) with correct scopes
- Airtable base/table configured (if using the record branch)

## Setup

### 1) Export / Import the workflow
- In n8n, export/download the workflow as JSON.
- In a new n8n instance, import the JSON to recreate the workflow.

![Connection](https://github.com/zukui1984/n8n-Intelligent-Document-Processing-Slack-Airtable/blob/master/images/n8n_connection.jpg)

### 2) Google Drive credentials
- Create/connect Google Drive credentials in n8n.
- Configure the Trigger to watch the intended folder.

![GDrive](https://github.com/zukui1984/n8n-Intelligent-Document-Processing-Slack-Airtable/blob/master/images/Google%20Drive%20.jpg)

### 3) Slack App configuration (for alerts)
Create a Slack App with a bot and ensure **Bot Token Scopes** include at least:
- `chat:write`
- `channels:read` (for public channels, if needed)
- `groups:read` (for private channels, if needed)

After adding scopes: **Reinstall the app to the workspace** so the token includes the updated permissions.

### 4) Code for Javascript
```js
// === Example placeholder (replace with your real n8n Code node script) ===
// Inputs typically available via `items`.
// Goal: normalize extracted fields and compute a risk_score.

return items.map(item => {
  const data = item.json;

  // Example normalization
  const vendor_name = (data.vendor_name || "Unknown Vendor").trim();

  // Example parsing
  const liability_cap = Number(data.liability_cap || 0);

  // Example scoring
  const risk_score = Number(data.risk_score || 0);

  return {
    json: {
      ...data,
      vendor_name,
      liability_cap,
      risk_score,
    }
  };
});
```

### 5) Airtable (for record creation)
- Create an Airtable Base + Table
- Add fields used by the workflow (example: vendor_name, risk_score, liability_cap, etc.)
- Connect Airtable credentials in n8n and map fields in “Create a record”
### Airtable configuration
Create fields for (example):
- `vendor_name`
- `liability_cap`
- `risk_score`
- `source_file_name`
- `source_file_url`
- `created_at`

![Airtable](https://github.com/zukui1984/n8n-Intelligent-Document-Processing-Slack-Airtable/blob/master/images/airtable.jpg)

## Running / Operating

### Testing (manual)
Use “Execute workflow” in the editor to run one test execution.

### Production
Toggle the workflow to **Active** so it runs automatically whenever the Google Drive trigger fires.

## Security notes
- Do **not** commit tokens/credentials to GitHub.
- Keep secrets in n8n Credentials (recommended) and/or environment variables.
- If publishing publicly, scrub any screenshots that contain IDs/tokens.

## Roadmap / Possible improvements
- Add a Slack message for the “Airtable record created” (true) branch too
- Add retries / error workflow (Error Trigger) for observability
- Store the analyzed results in a database for search/reporting
- Add structured logging + metrics (execution count, failures, avg processing time)



