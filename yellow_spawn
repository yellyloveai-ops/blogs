# One Agent Isn't Enough: How yelly-time Spawns Its Own Workforce

*5 min read · YellyRock team*

---

There's a moment in every complex automation task where the agent hits a wall — not because it can't do the work, but because the work is too wide to do sequentially.

Analyze 12 repositories. Run three independent audits. Refactor four modules in parallel. A single agent, working one step at a time, will get there eventually. But "eventually" might mean 40 minutes of wall-clock time for work that could logically run in 10.

**Task self-spawn** is yelly-time's answer to that problem. A running session can create child sessions via HTTP, each executing its own prompt in parallel. The parent coordinates. The children execute. The results come back when they're done.

---

## The Problem with Sequential Agents

Most agent workflows are linear: do step 1, then step 2, then step 3. That works fine when the steps depend on each other. But a lot of real work isn't like that.

Consider a team that runs a weekly dependency audit across 8 microservices. Each audit is independent — service A doesn't care what service B's dependencies look like. But if you run them sequentially, you wait for all 8 to finish before you see any results. The agent is idle 7/8 of the time, waiting for the previous task to complete.

The natural solution is parallelism. But most agent runtimes don't support it — you get one thread, one context, one task at a time.

yelly-time does it differently.

---

## How Self-Spawn Works

Every session running inside yelly-time gets three environment variables injected automatically:

| Variable | Purpose |
|---|---|
| `YELLYCLAW_SESSION_ID` | This session's own ID |
| `YELLYCLAW_PORT` | Server port (default: 2026) |
| `YELLYCLAW_TOKEN` | CSRF token for authenticated requests |

With those in hand, any session can spawn a child:

```bash
curl -s -X POST \
  -H "Content-Type: application/json" \
  -H "X-AgentRock-Token: $YELLYCLAW_TOKEN" \
  -d '{"prompt": "Audit dependencies in service-payments"}' \
  "http://localhost:$YELLYCLAW_PORT/sessions/$YELLYCLAW_SESSION_ID/spawn"
```

The server responds with a new session ID and a logs URL. The parent can fire off multiple spawns in rapid succession, then poll each child's logs until `exitCode` is set. When all children finish, the parent aggregates the results.

The child sessions show a **🌱 spawned** badge in the yelly-time UI — you can watch them run in real time.

---

## A Real Workflow: The Parallel Audit

Here's what the dependency audit looks like in practice. The parent prompt instructs Claude Code to spawn one child per service:

```
Use the self-spawn API to audit these services in parallel:
1. POST /sessions/$YELLYCLAW_SESSION_ID/spawn {"prompt":"Audit deps in service-auth"}
2. POST /sessions/$YELLYCLAW_SESSION_ID/spawn {"prompt":"Audit deps in service-payments"}
3. POST /sessions/$YELLYCLAW_SESSION_ID/spawn {"prompt":"Audit deps in service-notifications"}

Poll each child's /sessions/<id>/logs until exitCode is set.
Then summarize: which services have outdated deps, which are clean.
```

Wall-clock time drops from ~24 minutes (sequential) to ~8 minutes (parallel). The parent session stays alive, collecting results as children finish, then writes the summary.

---

## The Guardrails

Unbounded parallelism is a runaway risk. yelly-time enforces a hard limit: **5 child sessions per parent**. Hit the limit and the spawn API returns `429`. The parent has to wait for a child to finish before spawning another.

Other constraints:
- Children inherit the parent's agent configuration unless explicitly overridden
- The idle timeout (30 minutes of no output) applies to each child independently
- No tool escalation — children use the same preapproval rules as any other session
- Children can't spawn grandchildren (no recursive depth beyond one level)

The 5-session cap is intentional. It's enough to parallelize most real workloads without turning a single prompt into a server-saturating fork bomb.

---

## Rerun: The Other Half of the API

Self-spawn has a sibling: **rerun**. When a session fails partway through, you don't have to start from scratch. The rerun API creates a fresh session with the same original prompt:

```bash
curl -s -X POST \
  -H "X-AgentRock-Token: $YELLYCLAW_TOKEN" \
  "http://localhost:$YELLYCLAW_PORT/sessions/$YELLYCLAW_SESSION_ID/rerun"
```

This is the "Retry" button in the UI. Under the hood it's the same mechanism — a new session ID, same prompt, clean slate. Useful when a child session hits a transient error (network timeout, rate limit) and you want to retry just that subtask without re-running the whole parent.

---

## What We Learned

**Parallelism changes what's worth automating.** Tasks that felt too slow to automate become viable when you can fan out. The dependency audit example wasn't automated before self-spawn — not because it was hard, but because 40 minutes of sequential execution felt worse than just doing it manually. At 8 minutes, the calculus flips.

**The parent prompt is the hard part.** Writing a prompt that correctly fans out, polls for completion, and aggregates results requires more precision than a simple single-task prompt. The agent needs explicit instructions: spawn these N tasks, poll until done, then summarize. Vague instructions produce vague coordination.

**The 5-session cap is the right tradeoff.** Early testing without a cap produced sessions that spawned dozens of children, saturating the local server and producing results faster than the parent could process them. The cap forces the prompt author to think about what actually needs to run in parallel versus what can be sequential.

**Child sessions are first-class citizens.** Because each child has its own logs URL, its own session page, and its own export, you can inspect any individual subtask independently. When one child fails, you know exactly which subtask failed and why — without wading through a monolithic log.

---

*Questions or ideas? Open an issue at [yellyloveai-ops/yelly-time](https://github.com/yellyloveai-ops/yelly-time).*
