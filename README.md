# Tic Tac Toe

A tic tac toe game that runs in the browser. One self-contained HTML file — no build step, no dependencies, no network calls.

**[▶ Play it here](https://santy0510.github.io/tic-tac-toe/)**

## Features

- **Two modes** — pass-and-play with a friend, or take on the computer.
- **Unbeatable computer** — the opponent uses a full [minimax](https://en.wikipedia.org/wiki/Minimax) search over the game tree, so it plays perfectly. You can force a draw, but you can't win.
- **Winning line highlights** when a game ends.
- **Running scoreboard** tracking X wins, O wins, and draws across games.

## Running locally

Clone the repo and open `index.html` in any browser:

```sh
git clone https://github.com/santy0510/tic-tac-toe.git
cd tic-tac-toe
open index.html     # macOS
start index.html    # Windows
xdg-open index.html # Linux
```

No server required — it works straight from the filesystem.
