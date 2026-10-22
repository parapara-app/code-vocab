# code-vocab

A Claude Code plugin that explains the outside APIs a diff uses (built-ins, stdlib, framework and library functions) before you review it.
Each explanation is saved as a note, so an API you have met before shrinks to one line next time.

1. One step (Claude Code 2.1.275 or later)

   Inside a Claude Code session:

   ```
   /plugin install vocab --marketplace parapara-app/code-vocab
   ```

   From a terminal:

   ```sh
   claude plugin install vocab --marketplace parapara-app/code-vocab
   ```

2. Two steps (older Claude Code): add the marketplace first, then install from it

   Inside a Claude Code session:

   ```
   /plugin marketplace add parapara-app/code-vocab
   /plugin install vocab@code-vocab
   ```

   From a terminal:

   ```sh
   claude plugin marketplace add parapara-app/code-vocab
   claude plugin install vocab@code-vocab
   ```

To get new versions without updating by hand, open `/plugin`, go to Marketplaces, select code-vocab, and choose Enable auto-update.

| Command          | What it does                                                         | Example                      |
| ---------------- | -------------------------------------------------------------------- | ---------------------------- |
| `/vocab:review`  | Explains the outside APIs in the current diff or a PR, in ko, en, ja | `/vocab:review en`           |
| `/vocab:config`  | Shows or changes this machine's settings (private, url)              | `/vocab:config private on`   |

The first run asks you to log in with GitHub: open the link it prints, enter the code, and tell it you are done.
Your token is saved in `~/.config/parapara-code/config.json`.
