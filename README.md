# Alpha-Beta Pruning for Connect 4 (Depth-Limited)

An interactive demo of minimax with alpha-beta pruning. You play red against a yellow AI that looks a fixed number of moves ahead. After each AI move, the page shows what it searched, which branches it skipped, and how that compares with plain minimax.

## Live demo

Play it in your browser, no download needed:

**https://parvninama.github.io/alpha-beta-pruning-connect-4/**

The site is hosted with GitHub Pages and updates automatically whenever the `main` branch changes.

## Running it locally

1. Download `index.html` from the repository (or clone it):
   ```bash
   git clone https://github.com/parvninama/connect4-alpha-beta.git
   ```
2. Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge).

There is nothing to install and no internet connection is needed. Everything is in one file.

## How to play

- You are **red**, the AI is **yellow**.
- Click a column to drop a disc into it. It falls to the lowest free cell.
- The first player to get four discs in a row (horizontal, vertical or diagonal) wins. If the board fills up, the game is a draw.
- After your move, the AI searches and replies automatically.
- A dashed ring marks the most recent disc. A thick ring marks the winning four.

## Controls

| Control | What it does |
|---|---|
| **New game** | Clears the board and starts again. |
| **Hint (search for me)** | Runs the same search for your side and marks the suggested column with an arrow. It does not play the move. |
| **Moves first** | Choose whether you or the AI plays first. Changing it starts a new game. |
| **Search depth** | How many plies (single moves by either player) the AI looks ahead, from 1 to 7. Higher is stronger but slower. |
| **Centre-first move ordering** | When on, the search tries columns in the order 4, 3, 5, 2, 6, 1, 7 (centre outwards). When off, it tries them left to right. |
| **Compare with plain minimax** | When on, the page also runs a search without pruning to count how many positions it would have visited. This makes each AI move slower, especially at depth 6 and 7. |

## Reading the panels

**Stats**

- **Depth limit**: the number of plies searched.
- **Nodes (alpha-beta)**: positions examined with pruning.
- **Nodes (plain minimax)**: positions that a search without pruning would examine at the same depth.
- **Nodes saved**: the percentage of positions alpha-beta skipped.
- **Cutoffs**: how many times a branch was abandoned because α ≥ β.
- **Search time**: how long the pruned search took.

**Root moves table** lists each column the AI tried, in the order it tried them:

- **Window (α, β)**: the bounds passed into that move. α is the best score the AI already has from earlier columns, so it rises as better moves are found.
- **Score**: the value of the move. A score shown as "≤ x" is only an upper bound: the search stopped early because the move could not beat the best one so far.
- **Nodes**: positions examined under that move. Moves tried after a good one tend to cost much less.
- **Result**: whether the move raised α ("new best") or was "not better than α; cut short".

A score of **win** or **loss** means the search found a forced win or loss within the depth limit.

## Things to try

- Set the depth to 5 and watch the "Nodes saved" percentage after a few moves.
- Turn off **centre-first move ordering** and play the same opening. Cutoffs drop and the node count rises, because strong moves are tried later.
- Raise the depth to 7 with comparison off, then on. Alpha-beta finishes quickly either way, while plain minimax visits far more positions.
- Use **Hint** before each of your moves and compare it with what you would have played.
- Set the depth to 1 or 2 and try to set up a trap the AI cannot see coming.

## Theory

### The game tree

Each turn a player has up to 7 moves, so searching d plies ahead means up to 7^d positions. A full search of Connect 4 is far too large for a browser, so the AI stops at a fixed depth. This is **depth-limited search**.

### Minimax

The AI is the **MAX** player and wants the highest score. You are the **MIN** player and are assumed to always reply with the move that is worst for the AI. Scores are computed at the leaves of the tree and backed up:

- At a MAX node, the value is the maximum of its children.
- At a MIN node, the value is the minimum of its children.

At the root, the AI plays the move with the highest value.

### Evaluation function

At depth 0 the game is usually undecided, so the position is estimated with a heuristic:

- Every group of four consecutive cells (horizontal, vertical, diagonal) that contains only one player's discs scores by how many it holds: +2 for two of the AI's discs, +5 for three; −2 for two of yours, −6 for three.
- Groups containing both colours score nothing, since nobody can complete them.
- Each AI disc in the centre column adds 3, each of yours subtracts 3, because central discs belong to more possible fours.
- A forced win scores +100000 and a forced loss −100000. A small bonus for the remaining depth makes the AI prefer winning sooner and losing later.

These weights are hand-picked, not tuned, so the AI is decent but not perfect.

### Alpha-beta pruning

Alpha-beta gives the same result as minimax but skips branches that cannot affect the final decision. Two bounds are passed down the tree:

- **α**: the best score MAX is already guaranteed on the path so far.
- **β**: the best score MIN is already guaranteed on the path so far.

After each child is evaluated:

1. At a MAX node, set `α = max(α, score)`. At a MIN node, set `β = min(β, score)`.
2. If `α ≥ β`, stop examining the remaining children. This is a **cutoff**.

Why a cutoff is safe: if MIN already has an option worth β to it, and MAX has already found a way to get at least α ≥ β elsewhere, then MAX would never steer the game into this position. Whatever the remaining children are worth, the choice higher up the tree will not change.

```
function search(depth, α, β, isMax):
    if game over or depth == 0:
        return evaluate()
    if isMax:
        best = -∞
        for each move:
            best = max(best, search(depth-1, α, β, false))
            α = max(α, best)
            if α ≥ β: break        # cutoff
        return best
    else:
        best = +∞
        for each move:
            best = min(best, search(depth-1, α, β, true))
            β = min(β, best)
            if α ≥ β: break        # cutoff
        return best
```

### Move ordering

How much alpha-beta saves depends on the order moves are tried. If the best move comes first, cutoffs happen early and large subtrees are skipped. In the worst case nothing is pruned and the cost equals plain minimax.

- With perfect ordering, about b^(d/2) positions are searched instead of b^d, where b is the branching factor (up to 7) and d the depth. That roughly doubles the depth reachable in the same time.
- Connect 4 has a useful cheap heuristic: central columns are usually stronger, so trying them first improves pruning. This is the "Centre-first move ordering" option.

### Limits of this demo

- The search only sees d plies ahead (the horizon effect). A threat that develops just beyond the limit is invisible to it.
- There is no transposition table, so identical positions reached by different move orders are searched again.
- The evaluation weights are simple and fixed.

## Files

- `index.html`: the interactive demo (HTML, CSS and JavaScript in one file), served as the live site.
- `README.md`: this file.
