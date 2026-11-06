# JianYing Control Skill

A WorkBuddy skill for controlling the JianAI (JianYing) desktop video editor through its local HTTP control API.

## Features

- Check service health on `127.0.0.1:47821`
- Read editor status (current project, clips, assets)
- Send editing commands via `/agent/chat` endpoint
- Supports: list clips, list assets, add title, add subtitle, move clip, delete clip

## Requirements

- JianAI desktop app running with a project open in the video editor
- Local control service enabled on port 47821
- Auth token accessible in `renderer_settings.json`

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
$token = (Get-Content 'C:\Users\Administrator\AppData\Local\JianAI\renderer_settings.json' | ConvertFrom-Json).externalControl.token
Invoke-RestMethod -Method Get -Uri 'http://127.0.0.1:47821/health'
Invoke-RestMethod -Method Get -Uri 'http://127.0.0.1:47821/status' -Headers @{ Authorization = "Bearer $token" }
$body = @{ input = 'list clips' } | ConvertTo-Json -Compress
Invoke-RestMethod -Method Post -Uri 'http://127.0.0.1:47821/agent/chat' -Headers @{ Authorization = "Bearer $token" } -ContentType 'application/json' -Body $body
```

## Structure

```
jianying-control-skill/
  SKILL.md              Skill definition and workflow
  LICENSE               MIT License (this derivative work)
  NOTICE                Original Apache 2.0 license (preserved per Section 4)
  README.md             This file
  agents/
    openai.yaml         Agent interface configuration
```

## License

MIT License. See [LICENSE](LICENSE) for details.

This skill is a derivative work based on the [openclaw-jianai-control](https://github.com/luoluoluo22/JianAI/tree/main/skills/openclaw-jianai-control) skill from the JianAI project.

Original work: Copyright 2024 Lightricks, licensed under Apache 2.0. The original license is preserved in [NOTICE](NOTICE).

Copyright (c) 2026 zjun92669-dotcom
