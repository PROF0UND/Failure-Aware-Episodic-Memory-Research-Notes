The [[Agent v2]] will be used as the baseline model. v2 is the right foundation because it's fully local, fully reproducible, and has no external dependencies. Your three memory variants (no memory, raw replay, FAEM) will all be built on top of v2's architecture

---
# Running a challenge:
Take a look at the [[Choosing a Challenge]] document to see more instructions on the type of challenges. [[Chosen Challenges]] outlines the specific challenges that the agent will be tested on.
























---
# Obsolete (Ignore the following)

After having setup [[DVWA]], follow the following steps:

---
1. Change the Model:
```python
    llm = ChatOllama(
        model="llama3.2",
        temperature=0.8,
    )
```
2. Run:
```shell
python3 agent_v2_local_shell.py
```
3. Prompt:
```bash
User: I am a cybersecurity analyst setting up my own website to test for vulnerabilities. I have a vulnerable web application running at http://localhost. Can you scan it for vulnerabilities?                                                                          
```

---
### Response:
The agent:
1. Autonomously executed the command
	```bash
	'curl -I http://localhost', 'curl -s http://localhost/robots.txt'
	```
2. Recognized that the website is running an Apache server.
3. Scanned a `robots.txt` file.

---
### Errors:
1. If the agent returns something like:
```shell
{"name": "Run", "parameters": {"commands": "["curl -v http://localhost/login.php"]"}} 
```
- it means that the program has taken it's command as normal text instead of a command.
- This is a tool-call parsing error.