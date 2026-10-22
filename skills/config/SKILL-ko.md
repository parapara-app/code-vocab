---
name: config
description: 이 컴퓨터의 vocab 설정(~/.config/parapara-code/config.json)을 보여 주거나 바꾼다 — private (on|off)와 url (서버 주소 또는 default). 토큰은 절대 보여 주지 않는다.
argument-hint: "[private on|off] [url <주소>|default]"
allowed-tools: Bash(jq:*), Bash(chmod:*), Bash(mv:*)
---

당신은 vocab 플러그인의 기기별 설정을 보여 주거나 바꾼다. 설정은 `~/.config/parapara-code/config.json`에 있다: `{ "token": "…", "private": false }`, 로컬 서버로 돌려 볼 때만 `url`이 더 있다.

순서
1. `jq '{private, url, token: (if .token then "saved" else "missing" end)}' ~/.config/parapara-code/config.json`을 실행한다. 파일이 없으면 아직 설정이 없고 `/vocab:review`를 처음 돌릴 때 로그인한다고 한 줄로 말하고 멈춘다.
2. $ARGUMENTS로 jq 필터를 고른다:
   - `private on` → `.private = true`; `private off` → `.private = false`
   - `url <주소>` → `--arg url '<주소>' '.url = $url'`; `url default` → `del(.url)`
   - 인자 없음 → 아무것도 바꾸지 않고 3번으로 간다
   - 그 밖 → `usage: /vocab:config [private on|off] [url <address>|default]`를 찍고 멈춘다
   다른 칸은 그대로 둔 채 다시 쓴다: `jq <필터> config.json > config.json.tmp`, `chmod 600 config.json.tmp`, `mv config.json.tmp config.json` (모두 `~/.config/parapara-code/` 안).
3. 1번의 jq를 다시 실행하고 상태를 세 줄로 찍는다:
   - `private: <true|false>`
   - `url: <url>`, `url`이 없으면 `url: default (https://api.parapara.app/code)`
   - `token: saved` 또는 `token: missing`

규칙
- config.json을 Read하지 않고 토큰도 찍지 않는다. 파일을 보는 건 1번의 jq뿐이다.
- config.json 말고는 어떤 파일도 건드리지 않는다.

ARGUMENTS: $ARGUMENTS
