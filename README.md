# JianYing Control Skill

A WorkBuddy skill for controlling the JianAI (JianYing) desktop video editor through its local HTTP control API. Connects to the running JianAI app on `127.0.0.1:47821` and sends editing commands through the in-app Agent.

## Features

- Connect to running JianAI desktop app via local HTTP API
- Health check and status reading
- Timeline editing: list clips, move clip, delete clip
- Text overlay: add title, add subtitle
- Asset management: list assets, import asset
- Full response parsing with success/failure verification

## Requirements

- JianAI desktop app running with a project open in the video editor
- External control enabled in `renderer_settings.json`
- Auth token accessible in `renderer_settings.json`
- Windows (token file path uses Windows convention)

## Installation

### WorkBuddy UI

1. Open "Expert - Skill - Connector" in WorkBuddy desktop
2. Choose "Add Skill" > "Upload Skill"
3. Select the ZIP file
4. Confirm the security scan

### Manual

Copy to `~/.workbuddy-ai/skills/jianying-control-skill/`

## Usage

```powershell
$settings = Get-Content 'C:\Users\Administrator\AppData\Local\JianAI\renderer_settings.json' | ConvertFrom-Json
$token = $settings.externalControl.token
Invoke-RestMethod -Method Get -Uri 'http://127.0.0.1:47821/health'
Invoke-RestMethod -Method Get -Uri 'http://127.0.0.1:47821/status' -Headers @{ Authorization = "Bearer $token" }
$body = @{ input = 'add title Hello World at 1 second' } | ConvertTo-Json -Compress
Invoke-RestMethod -Method Post -Uri 'http://127.0.0.1:47821/agent/chat' -Headers @{ Authorization = "Bearer $token" } -ContentType 'application/json' -Body $body
```

## Structure

```
jianying-control-skill/
  SKILL.md              Skill definition, command reference, workflow
  LICENSE               MIT License (this derivative work)
  NOTICE                Original Apache 2.0 license (preserved per Section 4)
  README.md             This file
  agents/
    openai.yaml         Agent interface configuration
```

## Limitations

The JianAI HTTP control API currently exposes three endpoints: `/health`, `/status`, and `/agent/chat`. The `/agent/chat` endpoint accepts natural language commands and supports timeline editing (list, move, delete clips), text overlays (add title, add subtitle), and asset management (list, import). Features not yet exposed by the API include video export, transitions, audio adjustment, effects, and color correction. When JianAI adds new endpoints, update the SKILL.md Command Reference section.

## License

MIT License. See [LICENSE](LICENSE) for details.

This skill is a derivative work based on the [openclaw-jianai-control](https://github.com/luoluoluo22/JianAI/tree/main/skills/openclaw-jianai-control) skill from the JianAI project.

Original work: Copyright 2024 Lightricks, licensed under Apache 2.0. The original license is preserved in [NOTICE](NOTICE).

Copyright (c) 2026 zjun92669-dotcom
