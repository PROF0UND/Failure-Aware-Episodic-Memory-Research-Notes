
## Definition
Episodic memory is a term borrowed from *cognitive psychology*. In humans, it refers to your memory of specific past experiences — not general facts, but actual episodes you lived through.

For example:

- Knowing that "fire is hot" is *semantic memory* (general knowledge)
- Remembering the specific time _you_ burned your hand on a stove is *episodic memory* (a personal experience)

The key feature is that episodic memory is *tied to context* — what happened, when, where, and what the outcome was.

---

## Relevance to LLM
For an LLM, episodic memory would be recording past findings and using those experiences to guide future decisions. 

Without it, the LLM would have the general knowledge it was trained on but not the specific incidents it went through.

---

## Three types: 
1. *Successful episode memory*
	- "I tried exploit X on port 22 and it worked"
	- Most prior research focuses on this
2. *Failure episode memory* ← your contribution
	- "I tried exploit X on port 22 and it failed because the service was patched"
	- Structured, summarized, and stored so the agent doesn't repeat it
	- This is what makes your project novel
3. *Dead-end path memory*
	- "I went down this entire attack path and hit a wall"
	- Higher-level than individual failures — remembering whole strategies that didn't work

---

