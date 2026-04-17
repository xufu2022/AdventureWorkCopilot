your-project-root/
│
├── .copilot/
│   │
│   ├── copilot-instructions.md                    # ⭐ PRIMARY ENTRY POINT (MISSING BEFORE)
│   │   # This is the FIRST file Copilot reads
│   │   # Contains global instructions + agent/skill discovery rules
│   │
│   ├── skills/
│   │   └── sql-testing.md                        # Referenced from copilot-instructions.md
│   │
│   ├── agents/
│   │   ├── sql-orchestrator.md                   # Referenced from copilot-instructions.md
│   │   ├── sql-scanner.md
│   │   ├── sql-generator.md
│   │   └── sql-executor.md
│   │
│   ├── hooks/
│   │   ├── sql-testing-hooks.json
│   │   ├── validate-sql-operation.py
│   │   ├── log-test-generation.sh
│   │   └── notify-sql-error.sh
│   │
│   ├── templates/
│   │   ├── Function.ReadOnly.liquid
│   │   ├── Procedure.ReadOnly.liquid
│   │   └── Procedure.Mutation.liquid
│   │
│   ├── logs/
│   └── memory/
│
├── .github/                                      # Alternative location (lower priority)
│   ├── copilot-instructions.md                   # Fallback if .copilot/ missing
│   ├── skills/
│   └── hooks/
│
├── AGENTS.md                                      # Root-level alternative (legacy)
├── CLAUDE.md                                      # Cross-platform compatibility
│
├── DatabaseProject/                               # Your source SQL files
└── TestProject/                                   # Generated tests

is above able to unit test sp/functions in microsoft database AdventureWorks2014?
