---
name: study
description: diff 없이 공부한다 — 목차에서 아직 노트가 없는 함수(예: php array_)를 골라, 하나씩 공식 문서로 채운다. 하나하나 내가 공부한 것으로 남고, diff에서 만난 횟수와는 따로 센다. 첫 인자로 출력 언어 선택: ko|en|ja. 기본값: ko.
argument-hint: "[ko|en|ja] <lang> <접두어 | 이름 …>"
allowed-tools: Bash(curl:*), Bash(jq:*), Read, Write, WebFetch
---

당신은 사용자가 아직 diff에서 만나지 않은 함수의 노트를 공식 문서만 보고 채운다. 서버는 `https://api.parapara.app/code`이고, 아래에서는 <server>라고 부른다. 설정에 `url`이 있으면 그 주소로 바뀐다.

인자
- 첫 토큰이 ko|en|ja면 출력 언어다(기본 ko). 그다음 토큰은 목차에 적힌 그대로의 코드 언어다(`php`, `go`, `postgres`, `mysql`).
- 그 뒤의 토큰은 목차에 똑같은 이름이 있으면 이름, 없으면 이름 접두어다: `php array_`는 목록을 보여 주고, `php array_filter array_map`은 목록을 건너뛴다.

순서
1. `jq '{private, url}' ~/.config/parapara-code/config.json`을 실행한다. 파일이 없으면 `/vocab:review`를 처음 돌릴 때 로그인한다고 한 줄로 말하고 멈춘다. config.json을 Read하거나 토큰을 찍지 않는다.
2. `curl -sS <server>/api-notes/<lang>`을 jq로 거른다: `summary`가 null이고 `name`이 토큰에 맞는 행.
3. 이름을 받았으면 그 이름들로 4번에 간다. 접두어면 맞는 것을 번호 목록 `<n>. <package> · <name>`으로 최대 30개 찍고 멈춘다. 사용자는 번호나 "위에서 5개"로 답한다.
4. 고른 행마다 `doc_url`을 WebFetch로 읽고, 그 페이지만 보고 노트를 쓴다:
   - `summary`: 쉬운 단어로 쓴 평이한 영어 한 문장, 효과를 먼저
   - `example`: 혼자 돌아가는 2–5줄, 리터럴 값, 값이 나오는 줄마다 끝에 주석으로 결과
   - `gotcha`: 처음 쓰는 사람이 하는 실수 한 줄, 없으면 뺀다
   플러그인 루트의 `entry.tmpl`(이 스킬 폴더 기준 `../../entry.tmpl`, 첫 항목 전에 한 번 Read)을 채워서 보여 준다. review 전용 줄 `Signature` · `Params` · `Returns` · `Diff`는 뺀다. `{version}`은 문서 페이지에 보이는 버전이고, 없으면 뺀다. 그 뒤 저장한다: `{lang, package, name, doc_url, summary, example, gotcha}`를 scratchpad JSON 파일에 쓰고 `curl -sS -X POST <server>/api-notes -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d @<파일> -w '\n%{http_code}\n'`를 보낸다. `first_seen_in`은 빼서 본 횟수가 생기지 않게 한다.
5. 200은 목차 행을 채운 것, 409는 누가 먼저 채운 것이다. 그 밖은 `save failed: <name> <status>`. 200이나 409 뒤에는 공부한 것으로 남긴다: `curl -sS -X POST <server>/api-notes/studied -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d '{"lang": "…", "package": "…", "name": "…"}' -o /dev/null -w '%{http_code}\n'`. 204면 끝, 그 밖은 `studied failed: <name> <status>`.
6. 마지막 줄, 손댄 패키지마다: `<package> <summary 있는 행> / <전체 행>`. GET을 한 번 더 해서 센다.

규칙
- 공식 페이지만 보고 쓴다. 어느 컴에서든 사용자의 코드에서 나온 건 노트에 들어가지 않는다.
- 라벨과 어느 빈칸이 영어로 남는지는 `entry.tmpl`이 말하는 대로. 나머지 문장은 언어 토큰을 따른다.

ARGUMENTS: $ARGUMENTS
