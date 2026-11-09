# Wrongway bot research and implementation plan

Status: **research and design only**. `choose_action` has not been written and the Arena has not been modified. The goal is to select strong practical moves within one second per turn, 128 MB RAM, standard-library Python, and one independent submission file. No algorithm can guarantee a globally optimal move for every position under these limits; “optimal” means the strongest result from completed search and measured matches.

## Project review

| File | Role and design consequence |
| --- | --- |
| [`wrongway.md`](../wrongway.md) | Rules, 3–1–0 scoring, one-file submission, one-second turn limit, and 128 MB Colab CPU environment. |
| [`environment.py`](../wrongway-student/wrongway/environment.py) | Executable rules: 9×9 board, ten walls each, straight/diagonal jumps, path validation, goal win, and max-turn draw. |
| [`agent_runner.py`](../wrongway-student/wrongway/agent_runner.py) | Each bot is a separate process. The one-second timer includes computation and JSON communication; import has a separate five-second limit. Timeouts and errors forfeit a turn. |
| [`arena.py`](../wrongway-student/wrongway/arena.py) | Runs matches, swaps roles through `--player1`/`--player2`, accepts seeds, saves JSON replays, and reports failures. |
| `TEMPLATE.py`, `greedy.py`, `random_agent.py`, `wall_builder.py` | Baselines: shortest-distance movement, random actions, and one-ply BFS wall scoring. |

`TeamLeaderStudentID.py` is empty and this directory is not a Git repository. `wrongway.md` and `PROBLEM_STATEMENT.md` substantially overlap; the root document also specifies resource limits.

Rule details to reproduce: P0 starts at `(8, 4)` and targets row 0; P1 starts at `(0, 4)` and targets row 8; coordinates never flip. There are at most 128 geometric wall placements with `H`/`V` anchors in `0..7`. A wall cannot overlap, cross, or eliminate either player’s route. An adjacent opponent can be jumped straight over; a diagonal is available only when the square behind the opponent is blocked. `distances` is only the current player's wall-aware, pawn-ignoring BFS map. Root actions must come from the current `legal_moves` or `legal_walls` and cannot be reused in simulations.

The initial state has 3 legal pawn moves, 128 legal walls, and distance 8 for P0: 131 actions. One local match in each direction found `wall_builder` beating `greedy` after 28 turns as P1 and 23 turns as P0, with no errors or timeouts. This validates the runner only; it is not a statistical strength result. The assignment specifies 128 MB, while the local runner does not enforce a memory cap, so memory must be measured separately.

## Algorithm research and decision

| Technique | Value | Decision |
| --- | --- | --- |
| Multi-source BFS from the goal row | Exact wall-aware shortest paths on an 81-cell, unit-weight graph. | Use for evaluation and wall analysis. It does not model opponent replies or pawn jumps by itself. |
| Dijkstra/A* | Useful for weighted graphs or much larger boards. | BFS is simpler and exact here; benchmark A* only if path queries become a bottleneck. |
| Negamax/minimax with alpha-beta | Models best opponent responses and retains a completed answer using iterative deepening. | Main search method, with move ordering, wall filtering, and bounded caching. |
| MCTS/UCT | Has direct Quoridor research support in high-branching positions. | Experimental alternative only. Published results use 60k–120k simulations per decision, which cannot be assumed feasible in one second of Python. Random rollouts also waste walls and loop. |
| Deep learning/AlphaZero | Can provide high-quality evaluations with data and training resources. | Excluded from the first version: no model or GPU is available at grading time, and the deliverable is a 128 MB standalone file. |

Glendenning uses alpha-beta, iterative deepening, and path features, emphasizing the value of move ordering. Mertens shows that simplistic forward-progress features can enter dead ends, so evaluation must track goal paths. Respall, Brown, and Aslam improved Quoridor MCTS with shortest-path-guided rollouts because random rollouts spent walls too early. A 2026 result proves forced-win determination PSPACE-complete for generalized Quoridor; it does not solve the standard 9×9 opening. [Glendenning 2005](https://www.labri.fr/perso/renault/working/teaching/projets/files/glendenning_ugrad_thesis.pdf), [Mertens 2006](https://project.dke.maastrichtuniversity.nl/games/files/bsc/Mertens_BSc-paper.pdf), [Respall et al. 2018](https://www.researchgate.net/publication/327679826_Monte_Carlo_Tree_Search_for_Quoridor), [Drop et al. 2026](https://arxiv.org/abs/2605.22747).

## Proposed one-file bot architecture

1. **Normalize the observation.** Read positions, inventories, walls, turn data, and legal root actions. Encode cells as `0..80` and walls as `0..127` or bit sets. Do not require global state because a worker can restart.
2. **Choose an immediate fallback.** Take a winning move at once. Otherwise select the legal move with the best path distance. Pass only when no move and no useful wall exist.
3. **Build an internal simulator.** Represent wall-blocked edges; generate ordinary, straight-jump, and diagonal-jump moves; validate overlap/crossing and both routes with BFS. Validate it against `Environment` on random legal positions. At the root, still submit only Arena-provided legal actions.
4. **Evaluate positions.** Run BFS to both goal rows and calculate `d_opponent − d_self`, then add wall inventory, side to move, and only features that improve match results. Terminal outcomes and immediate opponent wins dominate heuristic terms. Use pawn-aware jump distances near adjacent pawns and in endgames.
5. **Create wall candidates.** At the root, score every legal wall by both path changes, its one-turn cost, wall inventory, and shortest-path effects. Retain the best candidates plus mandatory tactical/defensive walls. Use cheap geometric ranking before exact simulation validation.
6. **Search adversarially.** Use iterative-deepening negamax alpha-beta. Order immediate wins, forced defense, good pawn moves, pre-scored walls, then remaining moves. Cache complete state keys with a fixed size. Return only a fully completed depth and reserve 0.2–0.3 seconds for communication and fallback until benchmarks tune the deadline.
7. **Handle endgames and return safely.** With no walls, search the pawn race more deeply. Near either goal, force relevant block/jump replies. Verify the selected root action remains legal; on interruption or a fault, return the fallback. The submission must not write files or import `wrongway-student`.

Decision flow: `observation → immediate win/fallback → board model → two-sided BFS and candidate scoring → iterative alpha-beta → root legality check → return`.

## Implementation and validation sequence

| Phase | Work | Completion condition |
| --- | --- | --- |
| A. Rules and representation | Implement the 81-cell/128-wall model inside `TeamLeaderStudentID.py`; match moves, walls, wins, and draws with `Environment`. | Check opening and random positions, straight and edge diagonal jumps, overlapping/crossing walls, and route-preserving validation. |
| B. Robust heuristic | Add two-sided BFS, immediate wins, and wall evaluation that prices path change and tempo. | No invalid actions or timeouts; beat `greedy` and `TEMPLATE` from both sides across many games; also test `wall_builder`. |
| C. Search | Add iterative deepening, alpha-beta, tactical-preserving wall filtering, bounded cache, and a soft deadline. | All returned actions legal; no timeouts in opening, middlegame, endgame, and wall-heavy positions; p95/p99 retain margin below one second. |
| D. Evaluation | Test fixed seed sets from both sides against supplied bots, prior versions, and self-play. | Report W/D/L, 3–1–0 score, turns, invalid/error/timeout counts, p50/p95/p99 time, and peak memory. Accept only consistent improvements. |
| E. Submission | Tune candidate counts, depth, and weights from results; check Python 3.9+, standard library, and one-file independence; rename to the required student ID. | Works in both roles under the one-second/128 MB target in a Colab-like environment. |

Use the same seed set for every configuration and alternate P0/P1. Separate tuning games from final evaluation games. Retain losing replays and classify each failure: useless wall, missed immediate block, incorrect jump, repetition, timeout, or path-evaluation error. Add MCTS, beam search, or deeper tactical extensions only when they fix a measured failure and outperform alpha-beta under the same budget.

## Project organization

Keep `TeamLeaderStudentID.py` in the root because it is the specified target and must be standalone. Retain all `wrongway-student/wrongway` paths so the runner and documentation commands work. Keep `wrongway.md` at the root. `README.md` maps the project and this file stores research and design. A later coding session may add `tests/` for rule-model tests and `artifacts/replays/` for match logs; neither belongs in the submission.
