---
name: jianying-control-skill
description: Use this skill when the user wants to control a running JianAI (JianYing) desktop video editor from WorkBuddy. Covers service health check, status reading, and sending editing commands (list clips, add title, add subtitle, move clip, delete clip) through the local HTTP control API on 127.0.0.1:47821.
---

# JianYing Control Skill

Control the JianAI (JianYing) desktop video editor from WorkBuddy through its local HTTP control API.

## Preconditions

- JianAI desktop app must be running.
- The app must be inside the video editor for the target project.
- The local control service listens on `127.0.0.1:47821`.
- Auth token is stored in `renderer_settings.json` under `externalControl.token`.

Windows default token file path:

`C:\Users\<username>\AppData\Local\JianAI\renderer_settings.json`

## Workflow

1. Read `renderer_settings.json` and extract three fields:
   - `externalControl.enabled`
   - `externalControl.port`
   - `externalControl.token`
2. Call `GET /health` to confirm the service is alive.
3. Call `GET /status` with the auth token to get the current editor state.
4. Send editing commands via `POST /agent/chat` with a JSON body:

```json
{
  "input": "list clips"
}
```

5. Return the JSON result and summarize four things:
   - whether the call succeeded
   - provider used (`llm` or `rule`)
   - action count
   - assistant reply text

## Auth

Two header formats are accepted:

- `Authorization: Bearer <token>`
- `X-JianAI-Token: <token>`

## Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/health` | Check if the control service is running |
| GET | `/status` | Read the current editor status |
| POST | `/agent/chat` | Send an editing command to the in-app Agent |

## PowerShell Examples

Read token:

```powershell
$token = (Get-Content 'C:\Users\Administrator\AppData\Local\JianAI\renderer_settings.json' | ConvertFrom-Json).externalControl.token
```

Health check:

```powershell
Invoke-RestMethod -Method Get -Uri 'http://127.0.0.1:47821/health'
```

Read status:

```powershell
Invoke-RestMethod -Method Get -Uri 'http://127.0.0.1:47821/status' -Headers @{ Authorization = "Bearer $token" }
```

Send editing command:

```powershell
$body = @{ input = 'move the first clip forward by 1 second' } | ConvertTo-Json -Compress
Invoke-RestMethod -Method Post -Uri 'http://127.0.0.1:47821/agent/chat' -Headers @{ Authorization = "Bearer $token" } -ContentType 'application/json' -Body $body
```

## curl Examples

Health check:

```bash
curl http://127.0.0.1:47821/health
```

Read status:

```bash
curl -H "Authorization: Bearer $token" http://127.0.0.1:47821/status
```

Send editing command:

```bash
curl -X POST http://127.0.0.1:47821/agent/chat \
  -H "Authorization: Bearer $token" \
  -H "Content-Type: application/json" \
  -d '{"input":"list clips"}'
```

## Supported Commands

| Command | Description |
|---------|-------------|
| `list clips` | List all clips in the current timeline |
| `list assets` | List all media assets in the project |
| `add title <text> at <time>` | Add a title text overlay at the specified time |
| `add subtitle <text> at <time>` | Add a subtitle-like text overlay at the specified time |
| `move the first clip forward by <n> second(s)` | Move a clip on the timeline |
| `delete the last clip` | Delete the last clip from the timeline |

## Constraints

- This skill only controls the currently open editor window.
- It does not open projects automatically.
- It does not run headless.
- If the LLM endpoint is unavailable, JianAI may fall back to the local rule engine.
- Subtitle commands are currently implemented as title/text overlay elements. The assistant reply may say "已添加标题" instead of "已添加字幕". Treat subtitle insertion as on-screen text until the app exposes a distinct subtitle track action.

## Do Not

- Do not send commands when the health check fails.
- Do not hardcode the token. Always read it from `renderer_settings.json`.
- Do not assume a specific project is open. Always check `/status` first.
