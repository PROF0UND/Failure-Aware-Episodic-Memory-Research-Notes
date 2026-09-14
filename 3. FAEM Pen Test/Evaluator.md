evaluation of the step can be deterministic
```python
def evaluate_step(action: str, output: str, step: int, history: list) -> dict:
    """
    Deterministic failure detector. No LLM involved.
    Returns a result dict with success flag and failure reason.
    """
    
    # 1. FLAG FOUND — always a success
    if "HTB{" in output or "flag{" in output.lower():
        return {"success": True, "reason": "flag_found"}
    
    # 2. REPEATED ACTION — agent is looping
    recent_actions = [h["action"] for h in history[-3:]]
    if recent_actions.count(action) >= 2:
        return {"success": False, "reason": "repeated_action"}
    
    # 3. HTTP ERROR CODES — explicit failure signal
    if any(code in output for code in ["400", "401", "403", "404", "500"]):
        return {"success": False, "reason": "http_error"}
    
    # 4. KNOWN TOOL FAILURE PATTERNS
    if any(phrase in output.lower() for phrase in [
        "command not found",
        "connection refused", 
        "no route to host",
        "permission denied",
        "syntax error",
        "invalid option"
    ]):
        return {"success": False, "reason": "tool_failure"}
    
    # 5. EMPTY OUTPUT — usually means something went wrong
    if not output.strip():
        return {"success": False, "reason": "empty_output"}
    
    # 6. Default — assume partial progress
    return {"success": None, "reason": "ambiguous"}
```
---
> "we can't stop that unless a better model evaluates, at which point just use the better model"

That's exactly right, and it's actually a known open problem in agent research. The academic term for it is: 
>*semantic failure detection* vs *syntactic failure detection*. 

Your rule-based evaluator handles syntactic. Semantic requires either a stronger model or domain-specific heuristics — and if you have a strong enough model to detect strategic failures, you probably don't need the memory system as much anyway.