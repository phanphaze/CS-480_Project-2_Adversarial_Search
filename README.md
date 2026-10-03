# Pacman Multi-Agent Search

**Cadence Centers · CS 480 · Project 2: Adversarial Search**

Implementation of reflex, minimax, alpha–beta, and expectimax agents for the UC Berkeley Pacman framework, plus a custom state evaluation function. This project explores how search, assumptions about opponents, and heuristic weights affect gameplay.

## Implemented questions

| Question | Implementation | Approach |
| --- | --- | --- |
| Q1 | `ReflexAgent` | Scores immediate successor states using game score, food attraction, capsule proximity, and ghost opportunities/danger. |
| Q2 | `MinimaxAgent` | Maximizes Pacman's score while treating every ghost as a minimizing opponent. |
| Q3 | `AlphaBetaAgent` | Computes minimax decisions while pruning branches using alpha and beta bounds. |
| Q4 | `ExpectimaxAgent` | Maximizes Pacman's expected value with uniform probabilities over each ghost's legal actions. |
| Q5 | `betterEvaluationFunction` | Evaluates states using game score, food proximity, capsule proximity, remaining objectives, and ghost status. |

The assignment implementations are in `multiAgents.py`. The engine and supporting files originate from the provided framework.

## Setup

Use Python 3 and run commands from the complete assignment directory containing `pacman.py`, `multiAgents.py`, and `layouts/`. Graphical gameplay requires Python's Tkinter support. The autograder also requires the original `test_cases/` and supporting grading files.

No third-party Python packages are required by the provided gameplay code. A ZIP containing only `multiAgents.py` is a solution file package, not a standalone runnable game.

## Run the agents

```bash
# Reflex agent
python pacman.py -p ReflexAgent -l testClassic

# Minimax with two rounds of lookahead
python pacman.py -p MinimaxAgent -a "depth=2" -l smallClassic

# Alpha–beta search
python pacman.py -p AlphaBetaAgent -a "depth=2" -l smallClassic

# Expectimax with the default score evaluation
python pacman.py -p ExpectimaxAgent -a "depth=2" -l smallClassic

# Expectimax with the custom Q5 evaluation
python pacman.py -p ExpectimaxAgent -a "evalFn=better,depth=2" -l openClassic -k 2
```

Use layout names available in your `layouts/` directory, without the `.lay` extension. `better` is an alias for `betterEvaluationFunction`.

### Useful options

| Option | Meaning |
| --- | --- |
| `-l NAME` | Choose a layout. |
| `-k N` | Maximum ghost count, limited by the layout's ghost starts. |
| `-g DirectionalGhost` | Use directional ghosts instead of the default random ghosts. |
| `-n N` | Play N games. |
| `-q` | Disable graphics and print results. |
| `-f` | Use the framework's fixed random seed for repeatable experiments. |
| `--frameTime 0` | Remove the configured animation delay. |
| `-z 1.5` | Increase the graphics window scale. |

Example batch experiment:

```bash
python pacman.py -p ExpectimaxAgent -a "evalFn=better,depth=2" -l openClassic -k 2 -n 10 -q -f
```

## Search design

Agent index 0 is Pacman; indexes 1 onward are ghosts. Search generates successors recursively without constructing a separate tree data structure. One depth unit is a complete round of Pacman's move followed by all ghosts' moves. A terminal state or the depth limit triggers evaluation.

Minimax and alpha–beta maximize at Pacman nodes and minimize at every ghost node. Alpha–beta preserves legal-action order and uses strict cutoff comparisons (`value > beta` or `value < alpha`) to match the project's expansion requirements.

Expectimax replaces each ghost's minimum with the arithmetic mean of its legal successor values. Averaging occurs separately at each ghost node. Root action selection retains the first action achieving the best value.

## Custom evaluation

The final uploaded Q5 implementation combines:

- The current game score.
- Food attraction: a sum of `20 / (distance + 1)` over remaining pellets.
- Nearest-capsule attraction: `5 / (distance + 1)`.
- Scared-ghost attraction: `40 / (distance + 0.1)` when `distance + 3 < scaredTimer`.
- Dangerous-ghost penalty: `6 / (distance + 0.1)^2`, plus 100 when distance is at most 2.
- Remaining-objective penalties: 4 per food pellet and 20 per capsule.

All distances are Manhattan distances. A win returns game score plus 1000; a loss returns game score minus 1000. Finite terminal values allow expectimax to average terminal outcomes without combining opposite infinities.

Q1 uses its own immediate-action evaluation and weights; changing Q5 does not change the reflex agent.

### Limitations and experiments

Manhattan distance ignores walls, so attraction can favor positions whose actual routes require detours. Scared-timer comparisons estimate opportunities rather than guarantee a catch. Strong danger penalties can encourage waiting, and capsule proximity rewards must be balanced against the benefit of actually consuming a capsule.

Weights were adjusted through gameplay experimentation. Evaluate changes using both win rate and average score across multiple runs. A fixed seed helps compare versions; additional seeds test robustness. No benchmark scores or autograder results are claimed here.

## Autograder

```bash
python autograder.py --no-graphics
```

Or test individual questions:

```bash
python autograder.py -q q1 --no-graphics
python autograder.py -q q2 --no-graphics
python autograder.py -q q3 --no-graphics
python autograder.py -q q4 --no-graphics
python autograder.py -q q5 --no-graphics
```

Keep the grading files and test cases in the homework project. Follow the instructor's submission requirements for which files to upload.

## Optional recording and replay

The framework records actions and layout data, not video:

```bash
python pacman.py -p ExpectimaxAgent -a "evalFn=better,depth=2" -l openClassic -k 2 -n 8 -q -r
python pacman.py --replay "RECORDING_FILENAME" --frameTime 0.08
```

The provided recording path includes legacy Python 2 code. For a separate experimentation copy, make these Python 3 compatibility fixes in `pacman.py` before recording/replaying:

```python
# Saving: replace file(fname, 'w')
f = open(fname, 'wb')

# Loading: replace open(options.gameToReplay)
f = open(options.gameToReplay, 'rb')
```

These changes support binary pickle files. Only load trusted replay files. Video export and procedural map generation are future extensions, not features of this assignment implementation.

## Attribution and distribution

Based on the [UC Berkeley Pacman AI Projects](http://ai.berkeley.edu/). The original framework credits John DeNero and Dan Klein; student-side autograding credits Brad Miller, Nick Hay, and Pieter Abbeel.

Retain the original licensing and attribution notices. The supplied source notice permits educational use and extension subject to conditions including **not distributing or publishing solutions**. Keep this solution repository private; this README does not grant a new license to the framework or solutions.
