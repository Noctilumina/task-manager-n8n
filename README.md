# task-manager-n8n

N8N calendar sync workflows for the Task Manager app.

## Workflows

| File | Trigger | Purpose |
|------|---------|---------|
| `google-calendar-pull.json` | Every 15 min | Pulls Google Calendar events into Firestore |
| `google-calendar-push.json` | Webhook POST `/webhook/task-to-calendar` | Creates calendar event from task due date |

## Local Setup

### 1. Start N8N with Docker

```bash
docker run -d --name n8n -p 5678:5678 \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=changeme \
  -e FIREBASE_PROJECT_ID=your-project-id \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

N8N will be available at http://localhost:5678

### 2. Add Google Calendar credentials

1. Open http://localhost:5678 → Login (admin/changeme)
2. Go to **Credentials → Add Credential → Google Calendar OAuth2 API**
3. Add your Google Cloud project OAuth credentials (Client ID + Secret)
4. Complete the OAuth flow
5. Name it exactly: `Google Calendar - Production`

### 3. Import workflows

In N8N UI: Settings → Import Workflow → select each JSON from the `workflows/` folder.

OR via API:
```bash
curl -X POST http://localhost:5678/api/v1/workflows/import \
  -H "X-N8N-API-KEY: your-api-key" \
  -H "Content-Type: application/json" \
  -d @workflows/google-calendar-pull.json
```

### 4. Activate workflows

In N8N UI, toggle each workflow to Active.

## GitHub Secrets for CI

In repo Settings → Secrets and variables → Actions, add:
- `N8N_BASE_URL` — your N8N instance URL (e.g., `http://your-server:5678`)
- `N8N_API_KEY` — N8N API key (Settings → API → Generate API Key)

## Adding a New Calendar Provider

1. Copy a workflow JSON
2. Replace Google Calendar nodes with the new provider's nodes
3. Keep the same webhook path and output structure
4. Commit — CI deploys automatically
