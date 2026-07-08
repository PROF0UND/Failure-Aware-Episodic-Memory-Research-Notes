- **Context/problem:** LLM-based autonomous pentesting agents exist, but two of the field's own documented findings (from PentestGPT) are that LLMs lose long-term context over multi-step tasks and default to a depth-first, over-commit-to-one-path strategy rather than exploring alternatives.

> Recent advances in LLM-based penetration testing agents have revealed that LLMs lose long-term context over multi-step tasks and default to a depth-first action plan, and over-commit to one strategy rather than exploring alternatives. 


- **Prior work:** Note the established architectures (PentestGPT's three-session design — Reasoning/Generation/Parsing with a Pentesting Task Tree) and the broader landscape (single-agent systems are comparatively rare and underperforming; established, better-performing systems tend to be multi-module).

> Existing works have contributed to resolve this caveat by introducing multi-agent frameworks with each agent focusing on a particular task in penetration testing activities.

- **Your research question:** Does giving an agent failure-aware episodic memory (FAEM) — explicit memory of _why_ past attempts failed, not just what was tried — reduce this premature-commitment / exploration-exploitation failure mode?

> This project aims to bring light to the question: Can failure-aware structured episodic memory reduce repeated ineffective exploration and improve the performance of autonomous penetration-testing agents?

- **Motivation:** This failure mode isn't hypothetical — state that you've now observed it directly (your Scanner Service trajectory: 9 iterations fixating on tab-injection variants without pivoting) as concrete evidence the problem is real and reproducible on an established baseline.

> AI models have advanced quite a bit, however, their performance in penetration testing tasks still remains underwhelming. Tests conducted on benchmark penetration testing frameworks revealed agents are very succpetable to over-commiting to a stratergy before pivoti 

- **Methods (one sentence):** Reproduced PentestGPT's architecture on local open-source models, logging full trajectories, to compare no-memory / raw-replay / FAEM conditions on CTF benchmarks.

> 

- **Significance framing ("if X, then Y"):** If FAEM measurably reduces repeated/redundant actions and improves recovery after failure (vs. no-memory and raw-replay baselines), it would suggest that _failure-structured_ memory — not just longer context — is what's needed to fix this well-documented LLM pentesting weakness, with implications beyond security agents to any long-horizon agentic task.

> 