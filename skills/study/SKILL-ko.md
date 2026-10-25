---
name: study
description: diff 없이 공부한다 — 목차에서 아직 노트가 없는 함수(예: php array_)를 골라, 하나씩 공식 문서로 채운다. 하나하나 내가 공부한 것으로 남고, diff에서 만난 횟수와는 따로 센다. 첫 인자로 출력 언어 선택: ko|en|ja. 기본값: ko.
argument-hint: "[ko|en|ja] <lang> <접두어 | 이름 …>"
allowed-tools: Bash(curl:*), Bash(jq:*), Read, Write, WebFetch
---

당신은 사용자가 아직 diff에서 만나지 않은 함수의 노트를 공식 문서만 보고 채운다. 서버는 `https://api.parapara.app/code`이고, 아래에서는 <server>라고 부른다. 설정에 `url`이 있으면 그 주소로 바뀐다.

인자
- 첫 토큰이 ko|en|ja면 출력 언어다(기본 ko). 그다음 토큰은 목차에 적힌 그대로의 코드 언어다(`php`, `go`, `postgres`, `mysql`).
- 그 뒤의 토큰은 목차에 똑같은 이름이 있으면 이름, 없으면 이름 접두어나 패키지 이름이다: `php array_`는 목록을 보여 주고, `js express`는 그 패키지의 목록을, `php array_filter array_map`은 목록을 건너뛴다.

순서
1. 로그인 확인: `jq '{private, url}' ~/.config/parapara-code/config.json`을 실행한다. 파일이 없으면 "`/vocab:review`를 처음 돌릴 때 로그인한다"고 한 줄로 말하고 멈춘다. config.json을 Read하거나 토큰을 찍지 않는다.
2. 서버에 묻기: 사용자가 친 말(토큰)마다 `curl -sS '<server>/api-notes/<lang>?q=<토큰>'`을 한 번 부른다. 서버는 그 말이 패키지 이름이거나 함수 이름의 앞부분인 행을 돌려준다. 받은 행 중 `summary`가 null인 것(아직 노트가 비어 있는 것)만 남긴다(jq). `[]`가 오면 그 말은 목차가 없는 것이다. 운영자에게 알리는 건 서버가 하니, 출력 언어로 `<토큰>: 목차가 아직 없어 추가 요청을 보냈다` 한 줄만 적고 다음 말로 넘어간다.
3. 고르기: 친 말과 똑같은 이름의 행이 있으면 바로 그 행으로 4번에 간다. 아니면 맞는 행을 번호 목록 `<n>. <package> · <name>`으로 최대 30개 보여 주고 멈춘다. 사용자가 번호나 "위에서 5개"로 고른다.
4. 노트 쓰기: 고른 행마다 `doc_url`을 WebFetch로 읽고, 그 페이지만 보고 쓴다:
   - `summary`: 쉬운 단어로 쓴 평이한 영어 한 문장, 효과를 먼저
   - `example`: 혼자 돌아가는 2–5줄, 리터럴 값, 값이 나오는 줄마다 끝에 주석으로 결과
   - `gotcha`: 처음 쓰는 사람이 하는 실수 한 줄, 없으면 뺀다
   - `Signature`, `Params`, `Returns`: 같은 페이지에서. `Signature`는 저장도 하고, `Params`·`Returns`는 보여 주기만 한다 — 문서가 적는 호출 형태, 인자마다 한 줄, 뭘 돌려주고 가장자리에서 뭘 돌려주는지
   플러그인 루트의 `entry.tmpl`(이 스킬 폴더 기준 `../../entry.tmpl`, 첫 항목 전에 한 번 Read)을 채워서 보여 준다. diff가 있어야 쓸 수 있는 `Diff` 줄만 뺀다. `{version}`은 문서 페이지에 보이는 버전이고, 없으면 뺀다. 그 뒤 저장한다: `{lang, package, name, doc_url, summary, example, gotcha, signature}`(`signature`는 Signature 줄의 백틱 안, 문서에 시그니처가 없으면 뺀다)를 scratchpad JSON 파일에 쓰고 `curl -sS -X POST <server>/api-notes -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d @<파일> -w '\n%{http_code}\n'`를 보낸다. `first_seen_in`은 빼서 본 횟수가 생기지 않게 한다.
5. 저장 결과 보기: 200이면 목차 행을 채운 것, 409면 누가 먼저 채운 것이다. 그 밖은 `save failed: <name> <status>`. 200이나 409면 공부한 것으로 남긴다: `curl -sS -X POST <server>/api-notes/studied -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d '{"lang": "…", "package": "…", "name": "…"}' -o /dev/null -w '%{http_code}\n'`. 204면 끝, 그 밖은 `studied failed: <name> <status>`.
6. 진행률: 마지막 줄에 손댄 패키지마다 `<package> <summary 있는 행> / <전체 행>`. GET을 한 번 더 해서 센다.

규칙
- 공식 페이지만 보고 쓴다. 어느 컴에서든 사용자의 코드에서 나온 건 노트에 들어가지 않는다.
- 라벨과 어느 빈칸이 영어로 남는지는 `entry.tmpl`이 말하는 대로. 나머지 문장은 언어 토큰을 따른다.

ARGUMENTS: $ARGUMENTS
