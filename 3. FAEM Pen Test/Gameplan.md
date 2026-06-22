### Pre-Testing Gameplan for FAEM

The core principle here: **don't run comparative experiments until your instrumentation is trustworthy.** A run that produces uninterpretable logs is worse than no run at all — you've burned time and learned nothing.

---

#### [[Phase 0]]: Fix the Known Bugs First (Before Any Real Run)

---

#### [[Phase 1]]: Dry Run on KORP Terminal (Single Condition, Instrumentation Check)
---

#### [[Phase 2]]: Establish Your Run Matrix Before Starting

Write out your full run matrix explicitly before executing anything:

|Challenge|Condition|Run 1|Run 2|Run 3|
|---|---|---|---|---|
|KORP Terminal|Baseline||||
|KORP Terminal|FAEM||||
|TimeKORP|Baseline||||
|TimeKORP|FAEM||||
|Labyrinth Linguist|Baseline||||
|Labyrinth Linguist|FAEM||||

That's 18 runs minimum. At substantial time per run, this is your budget. Two things follow from this:

**Start with KORP Terminal** — you've already done dry runs on it, Docker is working, and it has a well-defined solve path (SQLi → hash crack). This is where your instrumentation is most likely to produce interpretable results first.

**Run one condition at a time per challenge, not interleaved.** Running Baseline × 3 on KORP, then FAEM × 3 on KORP, is easier to debug than alternating. If something breaks in the environment mid-experiment, you'll know which condition was affected.

---

#### Phase 3: Lock in Metric Definitions Now (Before You See Results)

This is the most important research hygiene point. Define exactly how you will compute each metric _before_ looking at any data, or you risk unconsciously tuning your definitions to favor a good story.

Here's what you need to specify:

**Repeated failures** — Is this the count of `FailureMemoryEntry` objects that share the same `phase` + `command_type`? Or exact command match? Fuzzy match? Decide now.

**Recovery efficiency** — How many turns between a logged failure and the next _distinct_ action? What counts as "recovery" — any new command, or specifically a successful one?

**Token usage** — Are you counting tokens in the injected memory context separately from the agent's generated tokens? This distinction matters for interpreting why FAEM might use more or fewer tokens than Baseline.

Write these down in a `notes/metric_definitions.md` file before your first comparative run.

---

#### Phase 4: The Actual Run Order

Given all of the above, here's the sequence I'd recommend:

1. Fix the three bugs → unit test each fix
2. One instrumentation-check run (FAEM, KORP, qwen3:8b)
3. If logs look correct: write `metadata.json` for TimeKORP and Labyrinth Linguist
4. Lock metric definitions in writing
5. Run Baseline × 3 on KORP Terminal → log everything
6. Run FAEM × 3 on KORP Terminal → log everything
7. Assess: are results interpretable? Are reflections substantive? If yes, proceed to TimeKORP. If no, diagnose before scaling.
8. Repeat for TimeKORP, then Labyrinth Linguist

Switch to `gemma4:31b` only after you've confirmed the full pipeline works end-to-end with `qwen3:8b`. You don't want to be debugging on your most expensive model.