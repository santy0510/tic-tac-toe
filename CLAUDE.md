# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Browser tic tac toe. The entire application is one file: `index.html` — markup, CSS, and JS inlined, with no dependencies, no build step, and no network calls.

There is no package manager, test runner, linter, or CI. Treat that as a deliberate constraint: **keep it a single dependency-free file** unless asked otherwise. Introducing a bundler or npm would break the "clone and double-click" property the README advertises.

## Running and deploying

```sh
start index.html   # Windows — just open it; works from file://
```

Deployed via GitHub Pages from the `main` branch at root, live at https://santy0510.github.io/tic-tac-toe/. Pushing to `main` triggers a rebuild (~45s).

**The filename `index.html` is load-bearing** — Pages serves it as the site root. Renaming it breaks the published URL.

Verify a deploy:
```sh
gh api repos/santy0510/tic-tac-toe/pages/builds/latest --jq .status   # expect "built"
curl -sSI https://santy0510.github.io/tic-tac-toe/                    # expect HTTP 200
```

`gh` is not on the Bash PATH on this machine. Full path:
`C:\Users\SPanda01\AppData\Local\Microsoft\WinGet\Packages\GitHub.cli_Microsoft.Winget.Source_8wekyb3d8bbwe\bin\gh.exe`

This repo has a **local** `user.email` of `332385711+santy0510@users.noreply.github.com`, overriding the global work address. The repo is public — preserve this override.

## Architecture

State lives in four module-scope variables (`index.html:97`): `board` (9-element array of `'X' | 'O' | null`), `current`, `over`, and `vsComputer`. The `board` array is the single source of truth; `cells` is a parallel array of 9 button elements built once at `index.html:100`.

`render()` is the only function that writes marks to the DOM. Mutate `board`, then call `render()` — never set cell text directly, or the two will drift.

### `winner(b)` is tri-state and does triple duty

At `index.html:110`, it serves as win detection, draw detection, *and* the minimax terminal test:

- `{ player: 'X'|'O', line: [...] }` — someone won
- `{ player: null, line: [] }` — board full, draw
- `null` — game still in progress

Callers distinguish "draw" from "keep playing" by `result.player`, not by truthiness of `result`. A refactor that collapses these cases will silently break both `checkEnd()` and `minimax()`.

### Computer opponent

Full minimax with no alpha-beta pruning (`index.html:149`). Fast enough because it only ever runs on ≤8 empty cells — the computer never moves first.

Two coupled assumptions: **the computer is always `O` and the human is always `X`**. Minimax hardcodes the scores `O = +1`, `X = -1`. Letting the computer play `X`, or adding computer-vs-computer, requires parameterizing those scores, not just changing who calls `computerMove()`.

Minimax mutates the passed array in place and restores it (`b[i] = ...` then `b[i] = null`). It is called with the live global `board`, so it must always undo every write — an early return that skips the restore corrupts the real game state.

### The move-delay race

`computerMove()` fires behind a 250ms `setTimeout`. During that window cells are still enabled, so a click would otherwise let the human move *as* `O`. The guard at `index.html:120` (`if (vsComputer && current === 'O') return;`) is what prevents this. Keep it if you touch input handling or the delay.

### Score lifecycle

`score` persists across `reset()` and is cleared only by a page reload. The mode toggle calls `reset()`, so switching modes clears the board but intentionally keeps the running tally.
