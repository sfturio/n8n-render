# n8n-render

This repository exists to run a dedicated n8n instance on Render for the AI agent project.

## What This Is For

Use this repo when you want a stable, separate n8n deployment that receives webhook calls from the AI Agent API and executes your workflows in production.

In short:
- `ai-agent-n8n` handles the chat API and UI.
- `n8n-render` hosts the automation/workflow engine (n8n).
- The API sends requests to an n8n webhook URL, and n8n returns the processed response.

## How It Connects To Your AI Agent

Architecture flow:
1. User sends a message in the AI agent chat.
2. The Node.js API (`ai-agent-n8n`) receives `POST /agent`.
3. The service layer calls an n8n webhook endpoint hosted from this repo.
4. n8n runs the workflow and returns JSON.
5. The API responds back to the user.

Related app repo:
- https://github.com/sfturio/ai-agent-n8n

Project link:
- https://ai-agent-n8n-zefs.onrender.com/

## Why Keep This Separate

Keeping n8n in its own repository makes it easier to:
- deploy/redeploy workflows independently from API code,
- isolate n8n environment variables and secrets,
- avoid mixing Node app runtime concerns with workflow runtime concerns.

## Files In This Repo

- `Dockerfile`: uses the official `n8nio/n8n` image and exposes port `5678`.
- `.env.example`: sample environment variables used to configure n8n.

## Environment Variables

Main variables used here:
- `N8N_HOST`
- `N8N_PORT`
- `GENERIC_TIMEZONE`
- `N8N_BASIC_AUTH_ACTIVE`
- `N8N_BASIC_AUTH_USER`
- `N8N_BASIC_AUTH_PASSWORD`
- `N8N_SECURE_COOKIE`
- `N8N_ENCRYPTION_KEY`
- `WEBHOOK_URL`

For production, set strong values for auth password and encryption key.

## Deploy On Render (High Level)

1. Create a new Web Service from this repository in Render.
2. Use Docker deployment (Render will build from `Dockerfile`).
3. Set environment variables from `.env.example`.
4. Save and deploy.
5. Copy your n8n webhook URL and configure it in `ai-agent-n8n`.

## Cold Start Mitigation (Recommended)

To reduce first-message failures on Render free tier:

1. Create a dedicated warmup workflow in n8n:
   - Trigger: `Webhook` (method `GET`, path `agent-warmup`)
   - Action: `Respond to Webhook` with `200` and a simple JSON body.
2. Keep your main agent workflow in a separate webhook path.
3. In `ai-agent-n8n` set:
   - `N8N_WARMUP_URL=https://your-n8n.onrender.com/webhook/agent-warmup`
   - `N8N_WARMUP_METHOD=GET`
   - `N8N_REQUEST_TIMEOUT_MS=45000` (or higher if needed)
4. Optionally use an external uptime ping on the warmup endpoint every 5 minutes.

This keeps n8n warm more often and lets the API gate requests until workflow readiness.

## Render/Proxy Best Practices

- Set `N8N_PROTOCOL=https`
- Set `N8N_PROXY_HOPS=1`
- Set `N8N_EDITOR_BASE_URL` to your Render n8n URL
- Set `WEBHOOK_URL` to the same public URL
- Keep `N8N_ENCRYPTION_KEY` stable across redeploys
- Do not commit real `.env` credentials into Git

## Notes

This repository is infrastructure support for the AI agent project, not the chat API itself.
