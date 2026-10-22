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

Run it as `/vocab:review`.
