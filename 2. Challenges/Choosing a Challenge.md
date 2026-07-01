- **Solvable via CLI tools** — no browser required, no GUI
- **Multi-step** — needs at least 3-4 distinct actions (recon → enumerate → exploit → capture). Single-step challenges won't show memory effects
- **Has clear failure states** — the agent should be able to _fail in interesting ways_, not just get stuck on connectivity
- **Deterministic** — same environment, same flag every run. No randomized elements
- **Self-contained** — runs in Docker or a simple server, not dependent on external infrastructure

---
### Choose from: 
1. Web
	1. These challenges generally involve a web server, e.g. running Apache, Nginx, or some other HTTP server. For our case, we spin up docker containers which can be accessed as localhost:port
	2. The general idea is to hack a website, though it can involve a number of different techniques, e.g. modifying URL or payload, code injection, XSS, etc.
	3. Example: [https://github.com/andyzorigin/cyber-bench/tree/main/benchmark/hkcert-ctf/CTF-Challenges/CTF-2022/22-back-to-the-past](https://github.com/andyzorigin/cyber-bench/tree/main/benchmark/hkcert-ctf/CTF-Challenges/CTF-2022/22-back-to-the-past) This required detecting and downloading a hidden directory from a web server
2. PWN
	1. Pwn is mainly about privilege escalation, getting shells, and arbitrary code execution. Binaries and C (or similar low level languages, but could be higher level ones too) are typically involved.
	2. Example: [https://github.com/andyzorigin/cyber-bench/tree/main/benchmark/hkcert-ctf/CTF-Challenges/CTF-2022/35-uaf](https://github.com/andyzorigin/cyber-bench/tree/main/benchmark/hkcert-ctf/CTF-Challenges/CTF-2022/35-uaf) You first need to interpret the source code (that user has 3 actions, add & removal animals, and print the name of the animal); then figure out the bug (issue with memory allocation, use after free vulnerability); then exploit to then get_shell
3. Misc

---
### [[Chosen Challenges]]
---
### Running a Challenge
To run a challenge, always use the hard mode. The easy prompt provides hints that make up for lack of memory.

---
### Three Metrics You Can Extract From Existing Logs Right Now

You don't need new runs. You need to reanalyze what you already have.

**1. Repeated action rate** — for each run, count how many tool calls are near-duplicates of a previous tool call in the same session. Baseline should repeat more. If FAEM repeats less, it's working.

**2. Technique diversity** — how many _distinct_ technique classes appear across the run? FAEM should show more pivoting behavior after failures.

**3. Post-failure pivot time** — after a confirmed failure event, how many turns before the agent tries something meaningfully different? FAEM should pivot faster.

None of these require the flag to be found. They're observable from the trajectory alone.