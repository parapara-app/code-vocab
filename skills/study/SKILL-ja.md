---
name: study
description: diff なしで勉強する — 目次からまだノートのない関数（例: php array_）を選び、一つずつ公式ドキュメントで埋める。一つずつ自分が勉強したものとして残り、diff で出会った回数とは別に数える。第一引数で出力言語を選ぶ: ko|en|ja。デフォルト: ko。
argument-hint: "[ko|en|ja] <lang> <接頭辞 | 名前 …>"
allowed-tools: Bash(curl:*), Bash(jq:*), Write, WebFetch
---

あなたは、ユーザーがまだ diff で出会っていない関数のノートを、公式ドキュメントだけを見て埋める。サーバーは `https://api.parapara.app/code` で、以下では <server> と呼ぶ。設定に `url` があれば、そのアドレスに置き換わる。

引数
- 最初のトークンが ko|en|ja なら出力言語（デフォルト ko）。次のトークンは目次の綴りどおりのコード言語（`php`、`go`、`postgres`、`mysql`）。
- その後のトークンは、目次にまったく同じ名前があれば名前、なければ名前の接頭辞: `php array_` は一覧を出し、`php array_filter array_map` は一覧を飛ばす。

手順
1. `jq '{private, url}' ~/.config/parapara-code/config.json` を実行する。ファイルがなければ、`/vocab:review` の初回実行でログインすると一行で言い、止まる。config.json を Read したりトークンを出したりしない。
2. `curl -sS <server>/api-notes/<lang>` を jq で絞る: `summary` が null で、`name` がトークンに合う行。
3. 名前を渡されたら、その名前で手順 4 へ。接頭辞なら、合うものを番号付きリスト `<n>. <package> · <name>` で最大 30 個出して止まる。ユーザーは番号か「上から 5 個」で答える。
4. 選んだ行ごとに `doc_url` を WebFetch で読み、そのページだけを見てノートを書く:
   - `summary`: 簡単な単語の平易な英語一文、効果を先に
   - `example`: 単独で動く 2〜5 行、リテラル値、値が出る行には行末コメントで結果
   - `gotcha`: 初めて使う人がやるミスを一行、なければ省く
   番号付きの項目として `Does`、0 列目のフェンスブロックの例、`Gotcha`、`Docs` を見せてから保存する: `{lang, package, name, doc_url, summary, example, gotcha}` を scratchpad の JSON ファイルに書き、`curl -sS -X POST <server>/api-notes -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d @<file> -w '\n%{http_code}\n'` を送る。`first_seen_in` は省き、遭遇回数が作られないようにする。
5. 200 は目次の行が埋まったということ、409 は誰かが先に埋めたということ。それ以外は `save failed: <name> <status>`。200 か 409 の後は、勉強したものとして残す: `curl -sS -X POST <server>/api-notes/studied -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d '{"lang": "…", "package": "…", "name": "…"}' -o /dev/null -w '%{http_code}\n'`。204 なら完了、それ以外は `studied failed: <name> <status>`。
6. 最後の行、触ったパッケージごとに: `<package> <summary のある行> / <全行>`。もう一度 GET して数える。

ルール
- 公式ページだけを見て書く。どの端末でも、ユーザーのコードから来たものはノートに入らない。
- ラベル（`Does` · `Docs` · `Gotcha`）は英語のまま。`Does`、例、`Gotcha` は保存されるノートなので、出力言語にかかわらず英語。ほかの文は言語トークンに従う。

ARGUMENTS: $ARGUMENTS
