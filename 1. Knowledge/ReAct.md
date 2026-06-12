[Paper Link](https://arxiv.org/pdf/2210.03629)
- The current agent state 

---
prior approaches treat reasoning and acting as separate capabilities — reasoning models hallucinate because they're disconnected from the environment, while acting models retrieve information but can't reason over it. ReAct interleaves "thought" steps and "action" steps so each one informs the other.

Uses a though -> Act -> Observe loop. The thought is the reasoning part of the agent, which does not affect the environment but still gets added to the context. 