# Wrongway Arena — bot development plan

This project uses the instructor-provided Arena to test a Wrongway/Quoridor bot. [`TeamLeaderStudentID.py`](TeamLeaderStudentID.py) in the repository root is the intended standalone submission file; it is currently empty. Research, design, and test plans are in [`docs/STRATEGY_PLAN.md`](docs/STRATEGY_PLAN.md).

## Structure

```text
.
├── TeamLeaderStudentID.py            # Standalone bot, not implemented yet
├── wrongway.md                       # Assignment specification and resource limits
├── README.md                         # Project map
├── docs/
│   └── STRATEGY_PLAN.md              # Research, architecture, and test plan
└── wrongway-student/wrongway/        # Original Arena package; paths retained
    ├── environment.py                # Rules and legal-action generation
    ├── agent_runner.py               # Isolated, time-limited bot runner
    ├── arena.py                      # Match runner and results
    ├── replay.py, visualizer.py      # Replay viewer and terminal renderer
    ├── PROBLEM_STATEMENT.md          # Assignment statement
    ├── STUDENT_GUIDE.md              # Protocol and local test guide
    └── submissions/
        ├── TEMPLATE.py               # Basic starter bot
        ├── greedy.py                 # BFS-distance movement bot
        ├── random_agent.py           # Random bot
        └── wall_builder.py           # BFS wall-evaluation example
```

`environment.py` is the local source of truth for rule validation. `wall_builder.py` is only an example bot and does not determine legality. The Arena package and its paths are retained so the commands in `STUDENT_GUIDE.md` continue to work.

After implementation, test the bot by passing the full path to `TeamLeaderStudentID.py` as both `--player1` and `--player2` in separate matches. Rename the final submission to the student ID required by `wrongway.md`.
