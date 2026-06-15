The system prompt includes explicit instructions to avoid blocking network commands, preventing agent hangs during automated experiments.

Built using [[LangGraph]]

If the agent retains full conversation history regardless of memory condition, the memory system becomes redundant — the LLM already has access to everything. Option B ensures the memory variant is the only source of inter-turn context, making the comparison meaningful.

