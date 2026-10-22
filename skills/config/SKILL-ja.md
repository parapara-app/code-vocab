---
name: config
description: この端末の vocab 設定（~/.config/parapara-code/config.json）を表示または変更する — private（on|off）と url（サーバーのアドレス、または default）。トークンは決して表示しない。
argument-hint: "[private on|off] [url <アドレス>|default]"
allowed-tools: Bash(jq:*), Bash(chmod:*), Bash(mv:*)
---

あなたは vocab プラグインの端末ごとの設定を表示または変更する。設定は `~/.config/parapara-code/config.json` にある: `{ "token": "…", "private": false }`、ローカルのサーバーで試すときだけ `url` が加わる。

手順
1. `jq '{private, url, token: (if .token then "saved" else "missing" end)}' ~/.config/parapara-code/config.json` を実行する。ファイルがなければ、まだ設定がなく `/vocab:review` の初回実行でログインすると一行で言い、止まる。
2. $ARGUMENTS から jq のフィルタを選ぶ:
   - `private on` → `.private = true`; `private off` → `.private = false`
   - `url <アドレス>` → `--arg url '<アドレス>' '.url = $url'`; `url default` → `del(.url)`
   - 引数なし → 何も変えず手順 3 へ
   - それ以外 → `usage: /vocab:config [private on|off] [url <address>|default]` と出して止まる
   ほかの項目はそのままにして書き戻す: `jq <フィルタ> config.json > config.json.tmp`、`chmod 600 config.json.tmp`、`mv config.json.tmp config.json`（すべて `~/.config/parapara-code/` の中）。
3. 手順 1 の jq をもう一度実行し、状態を三行で出す:
   - `private: <true|false>`
   - `url: <url>`、`url` がなければ `url: default (https://api.parapara.app/code)`
   - `token: saved` または `token: missing`

ルール
- config.json を Read せず、トークンも出さない。ファイルを見るのは手順 1 の jq だけ。
- config.json 以外のファイルには触れない。

ARGUMENTS: $ARGUMENTS
