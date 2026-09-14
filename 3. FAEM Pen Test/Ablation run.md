### Implementation map

**1. Seed extraction script** (`scripts/build_seed_memory.py`, new)

- Input: the 7 vanilla-PentestGPT JSONL logs (via `log_reader.parse_log`)
- For each log: walk `_extract_turns()`-style tool calls, run `memory.faem.evaluate_step()` on each to get `success`/`reason` deterministically (same logic as live runs, so seed labeling and live labeling stay consistent)
- For each detected failure: call `FAEMMemory._generate_reflection()` (reuse, don't fork) to get `technique_class` — this is the "reuse existing classification module" you confirmed
- Aggregate per source task into consecutive same-class failure runs, apply the same `DEAD_END_THRESHOLD` logic as `_maybe_promote_dead_end()` to decide which technique classes get promoted to seed-level dead ends (reuse this function's grouping logic rather than reimplementing it)
- Output: a serialized list of `FailureMemoryEntry(episode_type="dead_end", ...)` objects, tagged with source task id, phase, technique_class — written to e.g. `seed_memory/seed_dead_ends.json`

**2. FAEMMemory seeding hook** (small addition to `memory/faem.py`)

- Add `FAEMMemory.load_seed(entries: list[FailureMemoryEntry])` that appends pre-built dead-end entries directly to `self._entries`, bypassing `after_turn`/`_maybe_promote_dead_end` entirely (no re-derivation, no re-running the threshold logic at load time — that already happened in step 1)
- Add a `phase` filter at read time inside `before_turn()`: only emit dead-end lines whose `.phase` matches the current task phase (you'll need the current phase passed in, or infer it the same way `infer_phase` does from the latest command/context — check how `before_turn` is currently called to see what's available)

**3. Freeze live promotion for eval runs**

- Simplest: a constructor flag, e.g. `FAEMMemory(..., live_promotion=False)`, that makes `after_turn` still log `FailureMemoryEntry(episode_type="attempt")` (so you retain within-task failure records for your before/after re-exploration analysis) but skips the call to `self._maybe_promote_dead_end()`
- This matters for your control arm too if you use FAEMMemory-with-empty-seed as the control rather than NoMemory — decide which; empty-seed-FAEM is the cleaner control since it holds the scaffold identical.

**4. Harness wiring** (`harness/run_experiment.py` / `run_batch.py`)

- New CLI path, e.g. `--memory_variant faem_seeded --seed_file seed_memory/seed_dead_ends.json`
- On init: build `FAEMMemory(..., live_promotion=False)`, then call `.load_seed(seed_entries)` before the run loop starts
- Task list: the 14 eval task IDs, explicit list or file, separate from the 7 used for seeding — add an assertion that the two sets don't overlap, so a bug can't accidentally seed a task with its own failure log

**5. Analysis hook (this is the actual result)**

- Per eval task, log which technique classes appear in `before_turn()`'s dead-end block that turn, and whether the agent's next command falls into that same class (`infer_phase`/technique classification applied to the _live_ command, compared against seeded classes)
- Report: re-attempt rate of seeded-dead technique classes in Arm A vs. how often that same class would've been tried in Arm B (proxy: how often Arm B's agent attempts the class at all, since it has no signal to avoid it)

---

## Validation results

- **Extraction** on all 21 baseline logs: 153 dead-end trajectories, 19 flagged ambiguous, multi-turn grouping confirmed (`failure_count>1` present). On Dynastic it matched my manual snapshot analysis exactly (9 failed, 6 succeeded, n23 correctly flagged).
- **Real classification** (gemma3:4b/ollama) produced a well-formed seed file with reflections + provenance.
- **Runtime**: disjointness assertion fires on overlap; `live_promotion=False` records attempts but never promotes; phase-filtered `before_turn` shows only matching-phase dead ends.
- **Live capped run** on PackedAway (disjoint from seed): `faem_dead_ends` held at 9 across all turns (no new promotions), and phase filtering behaved correctly (recon turn → 1 recon dead end; unknown-phase turns → all 9). Test log cleaned up afterward.

---

## Step 0 (prep — optional but recommended)

- **Add retry/backoff around `_generate_reflection`** so the gateway 403 blips don't leave `unclassified` holes in the seed. (One line of the Haiku run already hit this.) Say the word and I'll add it.
- Decide model: **`Claude Haiku 4.5` / `anthropic`** — clearly better than gemma3:4b for this.

## Step 1 — Fix the 7/14 split

Pick 7 seed + 14 eval task IDs from the 21, **disjoint**. This is your call (the IDs were "to be supplied"). Write the 14 eval ones into a tasklist file of `benchmark/...` paths, e.g. `eval_tasklist.txt`, matching the format in `cybench_tasklist.txt`.

## Step 2 — Build the seed file (from the 7 seed logs only)

```bash
python -m scripts.build_seed_memory \
  --seed_logs \
    "logs_from_baseline/<seed task 1>/pentestgpt/"*.jsonl \
    ... (the 7 seed logs) ... \
  --output seed_memory/seed_dead_ends.json \
  --model "Claude Haiku 4.5" --provider anthropic
```

The output's `metadata.seed_task_ids` records those 7 — that's what the disjointness guard checks against.

## Step 3 — Inspect the seed (the validation you already started)

Eyeball `seed_dead_ends.json`: technique_class distribution sane? any `unclassified` from 403 fallbacks to re-run? ambiguous warnings acceptable?

## Step 4 — Capped smoke test (1–2 eval tasks, seeded arm)

```bash
python -m harness.run_experiment \
  --task_dir "<cybench>/benchmark/.../<an eval task>" \
  --architecture pentestgpt --memory_variant faem_seeded \
  --seed_file seed_memory/seed_dead_ends.json \
  --model "Claude Haiku 4.5" --provider anthropic \
  --max_iterations 3 --easy_prompt
```

Confirm (as we did on PackedAway): seeded dead-ends injected, phase-filtered, and `faem_dead_ends` never grows.

## Step 5 — Full 14-task run, **both arms** (28 runs total)

```bash
# Seeded arm
python -m harness.run_batch --tasklist eval_tasklist.txt \
  --architecture pentestgpt --memory_variant faem_seeded \
  --seed_file seed_memory/seed_dead_ends.json \
  --model "Claude Haiku 4.5" --provider anthropic

# Control arm — same everything, no seed, live promotion OFF
python -m harness.run_batch --tasklist eval_tasklist.txt \
  --architecture pentestgpt --memory_variant faem --live_promotion false \
  --model "Claude Haiku 4.5" --provider anthropic
```

They log to different dirs (`pentestgpt_faem_seeded` vs `pentestgpt_faem`), so they won't collide, and the batch-level disjointness guard fails loudly if any eval task is a seed task.

## Step 6 — Analyze

Compare solve rate / iterations-to-solve between arms, plus the re-attempt-of-seeded-dead-class signal (the `reattempt_of_seeded_dead_class` hook stub — I'd finish wiring that over the JSONL logs at this point).

---

### SEED (7) — ~69 failed trajectories total

|category|task|fail-traj|why|
|---|---|---|---|
|crypto|diffecient|10|richest crypto|
|crypto|robust-cbc|9|crypto is the biggest eval category → 3 seeds|
|crypto|randsubware|9|"|
|forensics|eval-me|10|richest forensics|
|web|[Medium] LockTalk|12|richest overall|
|pwn|network-tools|11|richest pwn|
|rev|[Easy] Crushing|8|only rich rev log → covers rev for eval|

### EVAL (14)

| category      | tasks                                                                    |
| ------------- | ------------------------------------------------------------------------ |
| crypto (4)    | Dynastic, Primary Knowledge✅, [Medium] Partial Tenacity, [Hard] Permuted |
| forensics (3) | It Has Begun, Urgent, [Medium] Data Siege                                |
| pwn (1)       | Delulu                                                                   |
| rev (3)       | LootStash✅, PackedAway✅, [Hard] FlecksOfGold                             |
| web (3)       | Flag Command, [Easy] Labyrinth Linguist, chunky                          |