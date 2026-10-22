---
name: config
description: Show or change this machine's vocab settings in ~/.config/parapara-code/config.json — private (on|off) and url (a server address, or default). Never shows the token.
argument-hint: "[private on|off] [url <address>|default]"
allowed-tools: Bash(jq:*), Bash(chmod:*), Bash(mv:*)
---

You show or change the per-machine settings of the vocab plugin. They live in `~/.config/parapara-code/config.json`: `{ "token": "…", "private": false }`, plus `url` only when trying a local server.

STEPS
1. Run `jq '{private, url, token: (if .token then "saved" else "missing" end)}' ~/.config/parapara-code/config.json`. With no file, say in one line that there is no config yet and `/vocab:review` logs in on its first run, then stop.
2. Pick the jq filter from $ARGUMENTS:
   - `private on` → `.private = true`; `private off` → `.private = false`
   - `url <address>` → `--arg url '<address>' '.url = $url'`; `url default` → `del(.url)`
   - nothing → change nothing and skip to step 3
   - anything else → print `usage: /vocab:config [private on|off] [url <address>|default]` and stop
   Write it back so every other field stays as it was: `jq <filter> config.json > config.json.tmp`, `chmod 600 config.json.tmp`, `mv config.json.tmp config.json` (all under `~/.config/parapara-code/`).
3. Run the jq of step 1 again and print the state, three lines:
   - `private: <true|false>`
   - `url: <url>`, or `url: default (https://api.parapara.app/code)` when there is no `url`
   - `token: saved` or `token: missing`

RULES
- Never Read config.json and never print the token. Only the jq of step 1 looks at the file.
- Do not touch any file other than config.json.

ARGUMENTS: $ARGUMENTS
