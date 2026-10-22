---
name: config
description: この端末の vocab 設定（~/.config/parapara-code/config.json）を表示または変更する — private（on|off）と url（サーバーのアドレス、または default）。トークンは決して表示しない。
argument-hint: "[private on|off] [url <アドレス>|default]"
allowed-tools: Read, Write
---

あなたは vocab プラグインの端末ごとの設定を表示または変更する。設定は `~/.config/parapara-code/config.json` にある: `{ "token": "…", "private": false }`、ローカルのサーバーで試すときだけ `url` が加わる。

手順
1. `~/.config/parapara-code/config.json` を読む。ファイルがなければ、まだ設定がなく `/vocab:review` の初回実行でログインすると一行で言い、止まる。
2. $ARGUMENTS を適用し、ほかの項目はそのままにしてファイルを書き戻す:
   - `private on` → `"private": true`; `private off` → `"private": false`
   - `url <アドレス>` → `"url": "<アドレス>"`; `url default` → `url` を消す
   - 引数なし → 何も変えず、書き込みもしない
   - それ以外 → `usage: /vocab:config [private on|off] [url <address>|default]` と出して止まる
3. 今の状態を三行で出す:
   - `private: <true|false>`
   - `url: <url>`、`url` がなければ `url: default (https://api.parapara.app/code)`
   - `token: saved`

ルール
- トークンの値は、頼まれても、どこにも出したり写したりしない。
- config.json 以外のファイルには触れない。

ARGUMENTS: $ARGUMENTS
