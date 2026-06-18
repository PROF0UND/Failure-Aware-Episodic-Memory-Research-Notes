The system prompt includes explicit instructions to avoid blocking network commands, preventing agent hangs during automated experiments.

Built using [[LangGraph]]

If the agent retains full conversation history regardless of memory condition, the memory system becomes redundant — the LLM already has access to everything. Option B ensures the memory variant is the only source of inter-turn context, making the comparison meaningful.

Base agent node output is of the form:
```python
{"messages": [response]}
```

`input_state` is just `{"messages": [SystemMessage(content=new_prompt, id="system_prompt")]}`. The `+all_messages` concatenation is gone because the checkpointer already holds everything else

Your state schema only has one channel: `messages` (a single list, `Annotated[list, add_messages]`). `AIMessage`, `SystemMessage`, `HumanMessage`, `ToolMessage` aren't separate channels — they're message _types_ that all live inside that one `messages` list together, distinguished by their class, not by which dict key they're under. So you wouldn't add a second top-level key like `ai_message` to `input_state`; you'd still send everything through `{"messages": [...]}`, and whatever's inside that list — be it a `SystemMessage` or an `AIMessage` — gets merged by the same `add_messages` reducer based on `id` matching, same mechanism either way.

`InMemorySaver` does exactly what the name says: it keeps that list sitting in a Python dict, in RAM, inside the running process. That's it — no database, no file, no network call. As long as the process that created it is still alive, `checkpointer.get(config)` just looks it up in that in-memory dict and hands you the list back. The reason this is fine for your setup is exactly what you confirmed last message: one continuous process from container spinup to teardown, so the dict never needs to outlive anything.

`InMemorySaver` just saves stuff in the RAM

---
## update_state()
Lets you inject states.

---
# Change dump
## Turn-by-turn shape of `run()`

**Setup (before the loop):**

- Fresh `uuid4().hex` `thread_id` per `run()` call → one thread per episode, never reused.
- `last_message_count = 0`, `serialized_history = []`.

**Turn 0:**

- `memory.before_turn([])` — no history yet.
- `input_state` seeds two messages with fixed IDs: `SystemMessage(id="system_prompt")` and `HumanMessage(id="task_description")`.
- `graph.invoke` runs the llm→(tools)→END sub-graph and writes the checkpoint.
- `new_messages = result["messages"][0:]` — the full initial state: `[SystemMessage, HumanMessage, AIMessage, ToolMessages...]`.
- `_serialize_messages` silently skips `SystemMessage` (no case for it), serializes `HumanMessage` as `user`, `AIMessage` as `assistant`, `ToolMessage` as `tool`.
- `memory.after_turn(serialized)`, `serialized_history = [] + serialized`.
- `last_message_count` updated to `len(result["messages"])`.

**Turn N (N > 0):**

- `memory.before_turn(serialized_history)` — receives everything from all prior turns (no SystemMessage in the list since it was silently skipped on turn 0 and never re-emitted as "new" on subsequent turns).
- `input_state` is just `[SystemMessage(content=updated_prompt, id="system_prompt")]`. The `add_messages` reducer replaces the existing SystemMessage in the checkpoint in-place; all prior HumanMessage/AIMessage/ToolMessages are already stored and the LLM sees them.
- `new_messages = result["messages"][last_message_count:]` — only the AIMessage(s) and ToolMessages added this turn.
- `serialized_history` grows by concatenation with `serialized` (the new slice only).

---

## Assumptions I made that you should confirm

1. **`_serialize_messages` is correct as-is.** It silently skips `SystemMessage` (falls through all `isinstance` checks without appending). This means the `SystemMessage` never appears in `serialized_history` or in anything passed to `before_turn`/`after_turn`. I confirmed this is the right behavior rather than changing the function.
    
2. **`serialized_history` is built by accumulation (`+`), not by re-serializing `result["messages"]`.** Both approaches produce equivalent output (since `_serialize_messages` skips SystemMessage either way), but accumulation avoids a redundant pass over the full history every turn. If you ever need `serialized_history` to include retrospectively updated content (e.g., a message ID was mutated by the graph), re-serializing from `result["messages"]` each turn would be safer — flag if that matters.
    
3. **Turn 0's `new_messages` includes the `HumanMessage`.** Since `last_message_count = 0`, the slice `result["messages"][0:]` covers `[SystemMessage, HumanMessage, AIMessage, ...]`. After skipping SystemMessage, the `HumanMessage` is serialized as `{"role": "user", "content": task_description}` and passed to `memory.after_turn`. This means memory variants will see the task description as a `user` message in the history starting from turn 1. In the old code, `before_turn` on turn 0 also saw the HumanMessage — so this is semantically equivalent (just shifted: old code put it in `before_turn`'s input, new code puts it in `after_turn`'s output, so it appears in `before_turn` from turn 1 onward instead).
    
4. **`raw_replay` note:** You flagged it as deprecated but it's still wired in `MEMORY_VARIANTS` in `run_experiment.py`. I haven't touched it — removing it deliberately is your call.