---
name: study
description: Study without a diff — pick functions from the index that have no note yet (e.g. php array_) and fill each one from its official docs. Each one is recorded as studied for you, apart from how many diffs you met it in. First argument may select the language: ko|en|ja. Default: ko.
argument-hint: "[ko|en|ja] <lang> <prefix | name …>"
allowed-tools: Bash(curl:*), Bash(jq:*), Read, Write, WebFetch
---

You fill notes for functions the user has not met in a diff yet, straight from the official docs. The server is `https://api.parapara.app/code`, called <server> below; a `url` in the config replaces it.

ARGUMENTS
- An optional first token ko|en|ja picks the output language (default ko). The next token is the code language as the index spells it (`php`, `go`, `postgres`, `mysql`).
- Every token after that is a name when the index has that exact name, and a name prefix or a package name otherwise: `php array_` lists, `js express` lists that package, `php array_filter array_map` skips the list.

STEPS
1. Login check: run `jq '{private, url}' ~/.config/parapara-code/config.json`. With no file, say in one line that `/vocab:review` logs in on its first run, then stop. Never Read config.json or print the token.
2. Ask the server: for each word the user typed (token), `curl -sS '<server>/api-notes/<lang>?q=<token>'` once. The server returns the rows whose package is that word or whose name starts with it. Keep only the rows whose `summary` is null, the ones with no note yet (jq). `[]` means the word has no index; the server tells the maintainer by itself, so print one line in the output language — `<token>: no index yet, a request to add it has been sent` — and move on to the next word.
3. Pick: when a row has exactly the typed name, go to step 4 with that row. Otherwise print the matches as a numbered list, `<n>. <package> · <name>`, at most 30, and stop. The user picks with numbers or "the first 5".
4. Write the note: for each picked row, WebFetch its `doc_url` and write from that page only:
   - `summary`: one plain-English sentence with easy words, effect first
   - `example`: 2–5 lines that run on their own, literal values, the output as a trailing comment on each line that makes one
   - `gotcha`: the mistake a first-time user makes, one short line, or left out
   - `Signature`, `Params`, `Returns`: from the same page. `Signature` is saved too; `Params` and `Returns` are shown only — the call as the docs write it, one line per parameter, what comes back and what comes back on the edge
   Print it by filling `entry.tmpl` at the plugin root (`../../entry.tmpl` from this skill's base directory; Read it once before the first entry), leaving out only the `Diff` line, which needs a diff. `{version}` is the version the doc page shows, left out when it shows none. Then save it: write `{lang, package, name, doc_url, summary, example, gotcha, signature}` (`signature` is the Signature line without its backticks, left out when the docs give none) to a scratchpad JSON file and send `curl -sS -X POST <server>/api-notes -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d @<file> -w '\n%{http_code}\n'`. Leave `first_seen_in` out, so no seen count is made.
5. Read the save result: 200 means the index row was filled, 409 means someone filled it first; anything else is `save failed: <name> <status>`. After a 200 or a 409, record it as studied: `curl -sS -X POST <server>/api-notes/studied -H "Authorization: Bearer $(jq -r .token ~/.config/parapara-code/config.json)" -H 'Content-Type: application/json' -d '{"lang": "…", "package": "…", "name": "…"}' -o /dev/null -w '%{http_code}\n'`. 204 is done; anything else is `studied failed: <name> <status>`.
6. Progress count: as the last line, per package touched, `<package> <rows with a summary> / <all rows>`, from one more GET.

RULES
- Write only from the official page. Nothing from the user's code goes into a note, on any machine.
- Labels, and which slots stay English, are as `entry.tmpl` says; every other sentence follows the language token.

ARGUMENTS: $ARGUMENTS
