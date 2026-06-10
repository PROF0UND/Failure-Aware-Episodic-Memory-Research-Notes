# [Link to the paper](https://arxiv.org/pdf/2303.11366)
![[Reflexion.pdf]]

---

## Summary
- Reflexion is a method to create and manage episodic memory
- It uses verbal self reflection to create episodic memory
- This involves using three models:
	1. Actor: The model that generates actions and text
	2. Evaluator: Evaluates the generated output. takes in the generated trajectory. 
	3. Self-reflection: An LLM instance that generates verbal reflections. Given a sparse reward signal, such as a binary success status (success/fail), the current trajectory, and its persistent memory mem, the self-reflection model generates nuanced and specific feedback. This feedback, which is more informative than scalar rewards, is then stored in the agent’s memory
- Reflexion Reflector prompt:
```
You are a Python writing assistant. You will be given your previous implementation of a function, a series of unit tests results, and your self-reflection on your previous implementation. Apply the necessary changes below by responding only with the improved body of the function. Do not include the signature in your response. The first line of your response should have 4 spaces of indentation so that it fits syntactically with the user provided signature. You will be given a few examples by the user.
```
- LLMs generate better reasoning in natural language than in forced JSON schemas.
