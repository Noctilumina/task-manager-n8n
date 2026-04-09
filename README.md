# task-manager-n8n

N8N calendar sync workflows for the Task Manager app. Handles two-way Google Calendar sync, decoupled from Firebase via webhooks.

## Workflows

| File | Trigger | Purpose |
|------|---------|---------|
| `workflows/google-calendar-pull.json` | Every 15 minutes | Fetches Google Calendar events and writes them to Firestore `calendarEvents` collection |
| `workflows/google-calendar-push.json` | Webhook `POST /webhook/task-to-calendar` | Creates a Google Calendar event from a task due date and writes back the `calendarEventId` |

## First-time setup

### 1. Start N8N with Docker

```powershell
docker run -d --name n8n -p 5678:5678 `
  -e N8N_BASIC_AUTH_ACTIVE=true `
  -e N8N_BASIC_AUTH_USER=admin `
  -e N8N_BASIC_AUTH_PASSWORD=changeme `
  -e FIREBASE_PROJECT_ID=task-manager-79da0 `
  -v n8n_data:/home/node/.n8n `
  n8nio/n8n
```

Open **http://localhost:5678** and log in with `admin` / `changeme`.

### 2. Create Google Calendar OAuth credentials

1. Go to **https://console.cloud.google.com/apis/credentials?project=task-manager-79da0**
2. Enable the **Google Calendar API** (APIs & Services → Enable APIs → search "Google Calendar API")
3. Create an **OAuth 2.0 Client ID** — type: Web application
4. Add `http://localhost:5678` to Authorised JavaScript origins and redirect URIs

### 3. Add credentials in N8N

1. In N8N: **Credentials → Add Credential → Google Calendar OAuth2 API**
2. Paste the Client ID and Client Secret from step 2
3. Complete the OAuth flow
4. Name it exactly: `Google Calendar - Production`

### 4. Import workflows

In N8N: **Settings → Import Workflow** → import each file:
- `workflows/google-calendar-pull.json`
- `workflows/google-calendar-push.json`

Then toggle both workflows to **Active**.

### 5. Add GitHub Secrets for CI auto-deploy

Get your N8N API key: N8N → **Settings → API → Generate API Key**

Go to **https://github.com/Noctilumina/task-manager-n8n/settings/secrets/actions** and add:

| Secret name | Value |
|-------------|-------|
| `N8N_BASE_URL` | `http://localhost:5678` (or your hosted URL) |
| `N8N_API_KEY` | key from N8N settings |

After this, pushing changes to `workflows/` auto-deploys to N8N.

## Adding a new calendar provider (e.g. Outlook)

1. Duplicate a workflow JSON
2. Replace the Google Calendar nodes with the new provider's N8N nodes
3. Keep the same input/output structure (same webhook path, same Firestore write format)
4. Commit to `workflows/` — CI deploys automatically
5. No app code changes needed

## Stopping / restarting N8N

```powershell
docker stop n8n
docker start n8n
```
