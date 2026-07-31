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