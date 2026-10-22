---
name: config
description: 이 컴퓨터의 vocab 설정(~/.config/parapara-code/config.json)을 보여 주거나 바꾼다 — private (on|off)와 url (서버 주소 또는 default). 토큰은 절대 보여 주지 않는다.
argument-hint: "[private on|off] [url <주소>|default]"
allowed-tools: Read, Write
---

당신은 vocab 플러그인의 기기별 설정을 보여 주거나 바꾼다. 설정은 `~/.config/parapara-code/config.json`에 있다: `{ "token": "…", "private": false }`, 로컬 서버로 돌려 볼 때만 `url`이 더 있다.

순서
1. `~/.config/parapara-code/config.json`을 읽는다. 파일이 없으면 아직 설정이 없고 `/vocab:review`를 처음 돌릴 때 로그인한다고 한 줄로 말하고 멈춘다.
2. $ARGUMENTS를 적용하고, 다른 칸은 그대로 둔 채 파일을 다시 쓴다:
   - `private on` → `"private": true`; `private off` → `"private": false`
   - `url <주소>` → `"url": "<주소>"`; `url default` → `url`을 지운다
   - 인자 없음 → 아무것도 바꾸지 않고 쓰지도 않는다
   - 그 밖 → `usage: /vocab:config [private on|off] [url <address>|default]`를 찍고 멈춘다
3. 지금 상태를 세 줄로 찍는다:
   - `private: <true|false>`
   - `url: <url>`, `url`이 없으면 `url: default (https://api.parapara.app/code)`
   - `token: saved`

규칙
- 토큰 값은 요청받아도 어디에도 찍거나, 출력하거나, 옮겨 적지 않는다.
- config.json 말고는 어떤 파일도 건드리지 않는다.

ARGUMENTS: $ARGUMENTS
