## Structure

In the PTT loop, each turn: the reasoning module proposes a **subtask node**, generation produces **one command** for it, it executes, parsing condenses the output, and the next turn proposes a _new_ subtask. Later, that node's status resolves (via the model's `SET` op or a heuristic) to one of `success/completed/failed`.

So a **failure trajectory = the lifecycle of one subtask node whose status resolved to `failed`** — which, given one-command-per-node, is in practice a single **(subtask description → command → output → `failed`)** unit. Build_seed_memory groups by `subtask_id`, checks the terminal status in the final PTT snapshot, and if `failed`, sends the command(s)+output to the classifier to produce one dead-end entry.

## Example A — a _clean_ failure trajectory (network-tools)

```
subtask   : "Test ping command with malicious input for command injection"
command   : echo "ping 127.0.0.1; cat /flag" | nc -w5 network-tools 1337
output    : ...Network Tools menu... "1. ping  2. traceroute ..." (input rejected as invalid choice)
PTT status: failed
      ↓ classified into ↓
SEED ENTRY: phase=exploitation | technique_class=command_injection
  reflection: "The command injection attempt failed because the application rejected
  'ping 127.0.0.1; cat /flag' as an 'Invalid choice!' … expects numeric menu selections
  (1–5) … inject within the parameters of a legitimate menu choice rather than bypassing
  the menu with shell syntax."
```

This is the ideal case: a genuine offensive technique (command injection), a clear reason it failed (menu-driven input, metacharacters not interpreted), and an actionable reflection that even names the _right_ next move.

## Example B — a _mislabeled_ failure trajectory (robust-cbc)

```
subtask   : "Use Python pexpect … to interact with service menu"
command   : python3 -c "import pexpect; child=pexpect.spawn('nc -w 5 robust 1337');
             child.sendline('1'); child.expect('flag:'); print(child.read())"
output    : Traceback … pexpect.expect() timed out (service showed a menu prompt, not 'flag:')
PTT status: failed
      ↓ classified into ↓
SEED ENTRY: phase=exploitation | technique_class=authentication_bypass
  reflection: "…pexpect.expect() failed because the anticipated response pattern doesn't
  match what the service returns … probe interactively first / adjust the expect pattern."
```

Here the _reflection is accurate_ but the **`technique_class` is wrong** — the real failure is a `pexpect` pattern-matching / scripting bug, not an authentication bypass. The classifier forced a security label onto a plumbing error. This is the taxonomy-fit problem in miniature.

## Two things this makes concrete

1. **`failure_count` ≠ commands in the trajectory.** Example A shows `failure_count=6`, but the trajectory is 1 command. That 6 is the **post-dedup merged weight** — 6 separate `(exploitation, command_injection)` trajectories across the seed collapsed into this one representative. Pre-dedup it was 1. (Example B is `1` because its `(phase,class)` was unique.)
    
2. **The "trajectory vs command" distinction in the original spec is largely moot for this data.** Because PentestGPT uses one subtask-node-per-command, trajectories are length-1 almost everywhere (only 1 of ~150 had 2 commands). So "trajectory-level classification" is effectively "command-level classification" here — the richer multi-command arcs the design anticipated never materialized with this agent's behavior.

So a failure trajectory is really: _one attempted command, the tool output that showed it didn't work, the PTT marking that node failed, and the LLM's one-paragraph post-mortem + technique label_ — and the quality of the whole seed rests on that label being right (Example A) vs forced (Example B).

