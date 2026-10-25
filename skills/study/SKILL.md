---
name: study
description: Study without a diff — pick functions from the index that have no note yet (e.g. php array_) and fill each one from its official docs. Each one is recorded as studied for you, apart from how many diffs you met it in. First argument may select the language: ko|en|ja. Default: ko.
argument-hint: "[ko|en|ja] <lang> <prefix | name …>"
allowed-tools: Bash(curl:*), Bash(jq:*), Read, Write, WebFetch
---

You fill notes for functions the user has not met in a diff yet, straight from the official docs. The server is `https://api.parapara.app/code`, called <server> below; a `url` in the config replaces it.

ARGUMENTS
- An optional first token ko|en|ja picks the output language (default ko). The next token is the code language as the index spells it (`php`, `go`, `postgres`, `mysql`).
- Every token after that is a name when the index has that exact name, and a name prefix otherwise: `php array_` lists, `php array_filter array_map` skips the list.

STEPS
1. Run `jq '{private, url}' ~/.config/parapara-code/config.json`. With no file, say in one line that `/vocab:review` logs in on its first run, then stop. Never Read config.json or print the token.
2. `curl -sS <server>/api-notes/<lang>` and filter with jq: rows whose `summary` is null and whose `name` matches the tokens.
3. Names given → go to step 4 with them. A prefix → print the matches as a numbered list, `<n>. <package> · <name>`, at most 30, and stop. The user answers with numbers or "the first 5".
4. For each picked row, WebFetch its `doc_url` and write the note from that page only:
   - `summary`: one plain-English sentence with easy words, effect first
   - `example`: 2–5 lines that run on their own, literal values, the output as a trailing comment on each line that makes one
   - `gotcha`: the mistake a first-time user makes, one short line, or left out
   Print it by filling `entry.tmpl` at the plugin root (`../../entry.tmpl` from this skill's base directory; Read it once before the first entry), leaving out the review-only lines `Signature` · `Params` · `Returns` · `Diff`. `{version}` is the version the doc page shows, left out when it shows none. Then save it: write `{lang, package, name, doc_url, summary, example, gotcha}` to a scratchpad JSON file and send `curl -sS -X POST <server>/api-notes -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d @<file> -w '\n%{http_code}\n'`. Leave `first_seen_in` out, so no seen count is made.
5. 200 means the index row was filled, 409 means someone filled it first; anything else is `save failed: <name> <status>`. After a 200 or a 409, record it as studied: `curl -sS -X POST <server>/api-notes/studied -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d '{"lang": "…", "package": "…", "name": "…"}' -o /dev/null -w '%{http_code}\n'`. 204 is done; anything else is `studied failed: <name> <status>`.
6. Last line, per package touched: `<package> <rows with a summary> / <all rows>`, from one more GET.

RULES
- Write only from the official page. Nothing from the user's code goes into a note, on any machine.
- Labels, and which slots stay English, are as `entry.tmpl` says; every other sentence follows the language token.

ARGUMENTS: $ARGUMENTS
