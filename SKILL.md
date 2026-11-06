---
name: jianying-control-skill
description: Use this skill when the user wants to control a running JianAI (JianYing) desktop video editor from WorkBuddy. Covers full local HTTP API control including service health check, editor status reading, and all supported editing commands through the in-app Agent on 127.0.0.1:47821.
---

# JianYing Control Skill

Control the JianAI (JianYing) desktop video editor from WorkBuddy through its local HTTP control API. This skill covers all endpoints and all commands currently supported by the JianAI control service.

## Preconditions

- JianAI desktop app must be running.
- The app must be inside the video editor for the target project.
- The local control service listens on `127.0.0.1:47821`.
- Auth token is stored in `renderer_settings.json` under `externalControl.token`.
- The `externalControl.enabled` field in `renderer_settings.json` must be `true`.

Windows default token file path:

`C:\Users\<username>\AppData\Local\JianAI\renderer_settings.json`

## Connection Setup

1. Read `renderer_settings.json`.
2. Verify `externalControl.enabled` is `true`.
3. Extract `externalControl.port` (default `47821`).
4. Extract `externalControl.token`.
5. Call `GET /health` to confirm the service is alive.
6. Call `GET /status` with the token to get the current editor state.
7. If any step fails, stop and report the failure to the user.

## Auth

Two header formats are accepted:

- `Authorization: Bearer <token>`
- `X-JianAI-Token: <token>`

Always read the token from `renderer_settings.json`. Never hardcode it.

## API Endpoints

| Method | Path | Purpose | Auth Required |
|--------|------|---------|---------------|
| GET | `/health` | Check if the control service is running | No |
| GET | `/status` | Read the current editor status (project, timeline, clips, assets) | Yes |
| POST | `/agent/chat` | Send an editing command to the in-app Agent | Yes |

## Command Reference

All commands are sent via `POST /agent/chat` with this JSON body format:

```json
{
  "input": "<natural language command>"
}
```

The JianAI in-app Agent parses the natural language input and executes the corresponding action. The response includes:

- `success`: boolean
- `provider`: `llm` or `rule` (which engine handled the command)
- `actions`: array of executed actions
- `assistant`: reply text from the agent

### Timeline Commands

| Command Pattern | Description |
|----------------|-------------|
| `list clips` | List all clips in the current timeline |
| `list assets` | List all media assets in the project |
| `move the first clip forward by 1 second` | Move a clip forward on the timeline |
| `move clip <n> back by <m> second(s)` | Move a clip backward on the timeline |
| `delete the last clip` | Delete the last clip from the timeline |
| `delete clip <n>` | Delete a specific clip by index |

### Text Overlay Commands

| Command Pattern | Description |
|----------------|-------------|
| `add title <text> at <time>` | Add a title text overlay at the specified time |
| `add subtitle <text> at <time>` | Add a subtitle-like text overlay at the specified time |
| `add title <text> at <time> for <duration> seconds` | Add a title with a specified duration |

### Asset Commands

| Command Pattern | Description |
|----------------|-------------|
| `list assets` | List all media assets imported into the project |
| `add asset <file_path>` | Import a media file into the project assets |

### Status Reading

| Command Pattern | Description |
|----------------|-------------|
| (via GET `/status`) | Read current project name, timeline duration, clip count, asset count |

## Workflow

1. **Connect**: Read token, call `/health`, call `/status`.
2. **Interpret**: Parse the status response to understand the current editor state.
3. **Command**: Send the user's requested editing command via `POST /agent/chat`.
4. **Verify**: Check the response `success` field. If the command failed, report the error.
5. **Confirm**: Call `/status` again to verify the command took effect.
6. **Report**: Summarize the result to the user with four things:
   - whether the call succeeded
   - which provider handled it (`llm` or `rule`)
   - how many actions were executed
   - the assistant reply text

## PowerShell Examples

Read token:

```powershell
$settings = Get-Content 'C:\Users\Administrator\AppData\Local\JianAI\renderer_settings.json' | ConvertFrom-Json
$token = $settings.externalControl.token
$port = $settings.externalControl.port
if (-not $port) { $port = 47821 }
```

Health check:

```powershell
Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:$port/health"
```

Read status:

```powershell
Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:$port/status" -Headers @{ Authorization = "Bearer $token" }
```

List clips:

```powershell
$body = @{ input = 'list clips' } | ConvertTo-Json -Compress
Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:$port/agent/chat" -Headers @{ Authorization = "Bearer $token" } -ContentType 'application/json' -Body $body
```

Add title:

```powershell
$body = @{ input = 'add title Hello World at 1 second' } | ConvertTo-Json -Compress
Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:$port/agent/chat" -Headers @{ Authorization = "Bearer $token" } -ContentType 'application/json' -Body $body
```

Move clip:

```powershell
$body = @{ input = 'move the first clip forward by 1 second' } | ConvertTo-Json -Compress
Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:$port/agent/chat" -Headers @{ Authorization = "Bearer $token" } -ContentType 'application/json' -Body $body
```

Delete clip:

```powershell
$body = @{ input = 'delete the last clip' } | ConvertTo-Json -Compress
Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:$port/agent/chat" -Headers @{ Authorization = "Bearer $token" } -ContentType 'application/json' -Body $body
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

Send command:

```bash
curl -X POST http://127.0.0.1:47821/agent/chat \
  -H "Authorization: Bearer $token" \
  -H "Content-Type: application/json" \
  -d '{"input":"add title Hello World at 1 second"}'
```

## Response Handling

The `POST /agent/chat` response is a JSON object. Always check these fields:

- `success`: If `false`, the command failed. Report the error to the user.
- `provider`: `llm` means the AI model processed it. `rule` means the local rule engine handled it.
- `actions`: Array of actions executed. Each action has a type and parameters.
- `assistant`: Natural language reply from the agent. Relay this to the user.

If `success` is `false`, check the `assistant` text for the reason. Common failure reasons:
- The clip index does not exist.
- The time format is invalid.
- The command is not recognized by the in-app Agent.
- The LLM endpoint is unreachable and the rule engine cannot handle the command.

## Important Notes

- Subtitle commands are currently implemented as title/text overlay elements by JianAI. The assistant reply may say "已添加标题" when you send an `add subtitle` command. This is expected behavior. Treat subtitle insertion as on-screen text until JianAI exposes a distinct subtitle track API.
- The in-app Agent uses natural language processing. Command phrasing matters. Use the patterns in the Command Reference section above. If a command fails, try rephrasing it.
- The control service only affects the currently open editor window. It cannot switch projects or open the app.
- If the LLM endpoint is unavailable, JianAI falls back to a local rule engine. The rule engine supports a subset of commands.

## Constraints

- This skill controls the currently open editor window only.
- It does not open projects automatically.
- It does not run headless.
- It does not export video, add transitions, adjust audio, add effects, or perform color correction. These features are not exposed by the JianAI HTTP control API.
- When JianAI adds new API endpoints or command capabilities, update the Command Reference section to include them.

## Do Not

- Do not send commands when the health check fails.
- Do not hardcode the token. Always read it from `renderer_settings.json`.
- Do not assume a specific project is open. Always check `/status` first.
- Do not claim the skill can perform operations not listed in the Command Reference.
- Do not ignore the `success` field in the response. Always verify the command result.
