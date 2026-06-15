The system prompt includes explicit instructions to avoid blocking network commands, preventing agent hangs during automated experiments.

Built using [[LangGraph]]

The agent loop needs its own separate message list for a different reason: to pass to LangGraph each turn so the LLM has conversation context.
