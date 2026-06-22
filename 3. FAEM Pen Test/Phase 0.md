You have three documented bugs that will silently corrupt your results if left unfixed. These are not optional pre-work — they are blockers.

**Priority 1 — `evaluate_step` app-level failure detection**  
This is the most critical bug. If `evaluate_step` classifies `{"message":"Invalid user or password"}` as "ambiguous," FAEM generates _zero_ `FailureMemoryEntry` objects. That means FAEM and Baseline become functionally identical — your entire experimental distinction collapses. Fix: add a phrase-matching layer that catches common app-level failure strings (401-adjacent messages, "invalid," "unauthorized," "forbidden" in body text) before falling through to "ambiguous."

**Priority 2 — `infer_phase` returning "unknown"**  
If every failure gets tagged as phase "unknown," your phase-filtered retrieval injects nothing useful. This is a quieter version of the same problem — FAEM degrades toward Raw Replay behavior. Fix: since `think: False` means `assistant_content` is empty, you need to infer phase from the _command string_ or _tool output_ itself, not the assistant's reasoning text.

**Priority 3 — Fuzzy repeated-action detection**  
This one can wait until after your first real runs, since you'll be able to see it in the logs. But you need it before you report repetition-reduction metrics.

**Verification step after each fix:** write a tiny unit test that feeds a synthetic `FailureMemoryEntry` through the fixed function and asserts the correct output. This takes 10 minutes per fix and saves you from debugging inside a 45-minute run.