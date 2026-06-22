Before you run any comparison, run one complete end-to-end run of **FAEM only** on **KORP Terminal** with `qwen3:8b` and verify the following checklist from the logs:

- [ ]  `FailureMemoryEntry` objects are actually being created (non-zero count)
- [ ]  Reflections are non-trivial and forward-looking (not just restating the failure)
- [ ]  Phase labels are something other than "unknown"
- [ ]  `before_turn()` is injecting entries at the right moment (check that injection happens _before_ the turn that follows a failure, not after)
- [ ]  `reset()` is clearing state between runs (critical — if this leaks, your conditions are contaminated)
- [ ]  Log files are being written with correct structure

If any of these fail, you haven't started your experiment yet — you're still debugging. That's fine, but you need to know that before burning 3 runs on a broken harness.
