[Paper Link](https://arxiv.org/pdf/2210.03629)
- The current agent state 

---
### About:
Prior approaches treat reasoning and acting as separate capabilities — reasoning models hallucinate because they're disconnected from the environment, while acting models retrieve information but can't reason over it. ReAct interleaves "thought" steps and "action" steps so each one informs the other.

Uses a though -> Act -> Observe loop. The thought is the reasoning part of the agent, which does not affect the environment but still gets added to the context.

---
### Baselines:
1. *Standard prompting*: which removes all thoughts, actions, observations in ReAct trajectories.
2. _Chain-of-thought_: Built on existing knowledge and reasoning-only
3. _Act-only_: removes thoughts in ReAct trajectories, loosely resembling how WebGPT interacts with the Internet to answer questions, though it operates on a different task and action space, and uses imitation and reinforcement learning instead of prompting.

---
### Strengths:
1. Reasoning grounds the actions (prevents acting blindly), and acting grounds the reasoning (prevents hallucination).
2. So ReAct uses the strength of one to account for the weakness of the other.

---
### Weaknesses:
![[Pasted image 20260612123651.png]]
1. Failure to recover from repetitive steps (47%)
2. Signal-to-noise problem: 
	1. The relevant failure is buried among dozens of irrelevant successes, tool outputs, and reasoning steps
	2. The model has no mechanism to _prioritize_ "this specific past failure is relevant to my current decision"
	3. So functionally, even though the information is "in context," it's not _retrievable_ at the moment of decision
	4. In essence, nothing tells the models what's important in the context to remember.

---
### Inferences:
ReAct (2023) first identified that LLM agents fall into repetitive loops because they fail to recognize when their current situation matches a past one. [[PentestGPT]] (2023) inherited this problem — its task tree records that an action failed, but doesn't help the agent recognize relevance to a new situation. This suggests the repetition problem isn't a quirk of one architecture — it's a fundamental limitation of giving agents undifferentiated history. FAEM addresses this directly by tagging failures with phase information and generating targeted reflections, so the relevant failure becomes salient precisely when the agent is in a matching phase — rather than buried in general context.