---
name: study
description: diff なしで勉強する — 目次からまだノートのない関数（例: php array_）を選び、一つずつ公式ドキュメントで埋める。一つずつ自分が勉強したものとして残り、diff で出会った回数とは別に数える。第一引数で出力言語を選ぶ: ko|en|ja。デフォルト: ko。
argument-hint: "[ko|en|ja] <lang> <接頭辞 | 名前 …>"
allowed-tools: Bash(curl:*), Bash(jq:*), Read, Write, WebFetch
---

あなたは、ユーザーがまだ diff で出会っていない関数のノートを、公式ドキュメントだけを見て埋める。サーバーは `https://api.parapara.app/code` で、以下では <server> と呼ぶ。設定に `url` があれば、そのアドレスに置き換わる。

引数
- 最初のトークンが ko|en|ja なら出力言語（デフォルト ko）。次のトークンは目次の綴りどおりのコード言語（`php`、`go`、`postgres`、`mysql`）。
- その後のトークンは、目次にまったく同じ名前があれば名前、なければ名前の接頭辞かパッケージ名: `php array_` は一覧を出し、`js express` はそのパッケージの一覧を、`php array_filter array_map` は一覧を飛ばす。

手順
1. ログイン確認: `jq '{private, url}' ~/.config/parapara-code/config.json` を実行する。ファイルがなければ、「`/vocab:review` の初回実行でログインする」と一行で言い、止まる。config.json を Read したりトークンを出したりしない。
2. サーバーに聞く: ユーザーが打った言葉（トークン）ごとに `curl -sS '<server>/api-notes/<lang>?q=<トークン>'` を一回呼ぶ。サーバーは、その言葉がパッケージ名か関数名の先頭である行を返す。受けた行のうち `summary` が null のもの（まだノートが空のもの）だけ残す（jq）。`[]` が返れば、その言葉には目次がない。メンテナーへの連絡はサーバーがするので、出力言語で `<トークン>: 目次がまだなく、追加の依頼を送った` と一行だけ書き、次の言葉に進む。
3. 選ぶ: 打った言葉とまったく同じ名前の行があれば、その行で手順 4 へ。なければ、合う行を番号付きリスト `<n>. <package> · <name>` で最大 30 個見せて止まる。ユーザーが番号か「上から 5 個」で選ぶ。
4. ノートを書く: 選んだ行ごとに `doc_url` を WebFetch で読み、そのページだけを見て書く:
   - `summary`: 簡単な単語の平易な英語一文、効果を先に
   - `example`: 単独で動く 2〜5 行、リテラル値、値が出る行には行末コメントで結果
   - `gotcha`: 初めて使う人がやるミスを一行、なければ省く
   - `Signature`、`Params`、`Returns`: 同じページから。`Signature` は保存もし、`Params` と `Returns` は見せるだけ — ドキュメントが書く呼び出しの形、パラメータ一つにつき一行、何が返るかと端のケースで何が返るか
   プラグインルートの `entry.tmpl`（このスキルのフォルダから `../../entry.tmpl`、最初の項目の前に一度 Read）を埋めて見せる。diff がないと書けない `Diff` の行だけ省く。`{version}` はドキュメントページに見えるバージョンで、なければ省く。そのあと保存する: `{lang, package, name, doc_url, summary, example, gotcha, signature}`（`signature` は Signature の行のバッククォートの中、ドキュメントにシグネチャがなければ省く）を scratchpad の JSON ファイルに書き、`curl -sS -X POST <server>/api-notes -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d @<file> -w '\n%{http_code}\n'` を送る。`first_seen_in` は省き、遭遇回数が作られないようにする。
5. 保存結果を見る: 200 なら目次の行が埋まったということ、409 なら誰かが先に埋めたということ。それ以外は `save failed: <name> <status>`。200 か 409 なら、勉強したものとして残す: `curl -sS -X POST <server>/api-notes/studied -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d '{"lang": "…", "package": "…", "name": "…"}' -o /dev/null -w '%{http_code}\n'`。204 なら完了、それ以外は `studied failed: <name> <status>`。
6. 進み具合: 最後の行に、触ったパッケージごとに `<package> <summary のある行> / <全行>`。もう一度 GET して数える。

ルール
- 公式ページだけを見て書く。どの端末でも、ユーザーのコードから来たものはノートに入らない。
- ラベルと、どのスロットが英語のままかは `entry.tmpl` の言う通り。ほかの文は言語トークンに従う。

ARGUMENTS: $ARGUMENTS
