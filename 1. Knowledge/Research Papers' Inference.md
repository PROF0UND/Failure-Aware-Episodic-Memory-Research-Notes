## From [[PentestAgent]]:
- The three phase structure.
- recon → vulnerability analysis → exploitation
- failures mean different things at different phases
- A failed curl during recon is different from a failed sqlmap during exploitation.

---
## From [[Reflexion]]
- The three model design.
- Actor → Evaluator → Self-Reflection
- the self-reflection model's inputs: 
- `binary success status + current trajectory + persistent memory → verbal feedback`
---
## FAEM Schema
Combining both papers, here is the memory entry structure for faem.py:

```python
@dataclass
class FailureMemoryEntry:
    # WHAT happened (from PentestAgent's phase structure)
    phase: str              # "recon" | "vulnerability_analysis" | "exploitation"
    action: str             # the exact command that was run
    
    # WHAT the result was
    outcome: str            # raw tool output (truncated to 500 chars)
    success: bool           # binary signal (from Reflexion's evaluator)
    
    # WHY it failed (from Reflexion's self-reflection model)
    reflection: str         # LLM-generated verbal summary of why this failed
                            # and what to avoid repeating
    
    # WHEN (for ordering and context)
    step: int               # which iteration this happened on
    timestamp: str          # ISO format
```

And the memory store itself:
```python
class FAEMMemory(BaseMemory):
    def __init__(self):
        self.entries: list[FailureMemoryEntry] = []
    
    def before_turn(self, messages):
        # Returns only FAILED entries as formatted string
        # injected into system prompt
        failures = [e for e in self.entries if not e.success]
        if not failures:
            return ""
        return self._format_failures(failures)
    
    def after_turn(self, new_messages):
        # Detects if the last action failed
        # If yes: calls LLM to generate reflection, stores entry
        # If no: stores success entry without reflection
        pass
    
    def reset(self):
        self.entries = []
```
---
## Reflection Prompt Template
Adapted from Reflexion (Shinn et al. 2023) for pentesting context.
Output is free-form prose (2-3 sentences), not JSON.
Stored in FailureMemoryEntry.reflection field.

```
You are a penetration testing assistant reviewing a failed action.

Phase: {phase}
Command attempted: {action}
Output received: {outcome}

In 2-3 sentences, explain:
1. Why this action failed
2. What specific pattern to avoid repeating
3. What to try instead

Be concrete. Reference the exact command syntax or flag that caused the failure.
```

## Key Novelty Over Reflexion
- Reflexion: stores prose reflection only
- FAEM: prose reflection + structured metadata (phase, action, step)
- Enables phase-filtered retrieval — Reflexion cannot do this