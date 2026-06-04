```
faem-pentest-agent/
├── README.md
├── .gitignore
├── requirements.txt
├── setup_challenges.sh
│
├── agent/
│   ├── __init__.py
│   └── base_agent.py          ← the core agent loop (Ollama + shell tool)
│
├── memory/
│   ├── __init__.py
│   ├── base_memory.py         ← abstract base class all 3 variants inherit from
│   ├── no_memory.py           ← Baseline 1
│   ├── raw_replay.py          ← Baseline 2
│   └── faem.py                ← Your contribution
│
├── harness/
│   ├── __init__.py
│   └── run_experiment.py      ← orchestrates a single run: setup → agent → log
│
├── evaluation/
│   ├── __init__.py
│   └── analyze.py             ← reads logs, computes metrics, outputs tables
│
└── logs/                      ← gitignored, created at runtime
```

---
