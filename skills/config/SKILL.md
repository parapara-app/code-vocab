---
name: config
description: Show or change this machine's vocab settings in ~/.config/parapara-code/config.json — private (on|off) and url (a server address, or default). Never shows the token.
argument-hint: "[private on|off] [url <address>|default]"
allowed-tools: Read, Write
---

You show or change the per-machine settings of the vocab plugin. They live in `~/.config/parapara-code/config.json`: `{ "token": "…", "private": false }`, plus `url` only when trying a local server.

STEPS
1. Read `~/.config/parapara-code/config.json`. With no file, say in one line that there is no config yet and `/vocab:review` logs in on its first run, then stop.
2. Apply $ARGUMENTS, then write the file back with every other field as it was:
   - `private on` → `"private": true`; `private off` → `"private": false`
   - `url <address>` → `"url": "<address>"`; `url default` → remove `url`
   - nothing → change nothing and do not write
   - anything else → print `usage: /vocab:config [private on|off] [url <address>|default]` and stop
3. Print the current state, three lines:
   - `private: <true|false>`
   - `url: <url>`, or `url: default (https://api.parapara.app/code)` when there is no `url`
   - `token: saved`

RULES
- Never print, echo, or copy the token value anywhere, even when asked.
- Do not touch any file other than config.json.

ARGUMENTS: $ARGUMENTS
