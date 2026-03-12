# When Your AI Agent Fixes Its Own Mistakes

*5 min read · yelly-time community*

---

It was 2am when the Claude Code schedule failed.

Nobody was watching. The schedule had been running cleanly for two weeks — summarizing overnight GitHub activity, filing issues, posting a morning digest to Slack. Then one night it hit a new repository that required a different tool, and Claude Code stopped with a permission error. The schedule paused. The morning digest never arrived. The team noticed at standup.

The fix took 30 seconds once someone looked at the logs: add `web_fetch` to the schedule's allowed tools. But those 30 seconds required a human to wake up, find the error, understand it, and apply the fix. At 2am, that's a lot to ask.

This is the problem `autoRunUntilNoError()` solves.

---

## The Pattern: Error Report → Auto-Fix → Retry

When a Claude Code session exits with a non-zero code, yelly-time now does something new: it generates a dedicated **error report** alongside the session logs.

The report is simple — two fields:

```
Root Cause: Tool 'web_fetch' requires approval but was not in the allowed list.
Resolution: Add 'web_fetch' to the schedule's Additional Tools setting.
```

That's it. No stack trace archaeology. No grepping through 400 lines of Claude output. The server parses the output, matches known error patterns, and writes a human-readable `error-report.md` into the session folder.

The patterns it recognizes:

| What Claude Code says | Root Cause | Resolution |
|---|---|---|
| `tool not approved` / `approval required for X` | Tool `X` blocked | Add `X` to allowTools |
| `permission denied` / `EACCES` on exec | Shell execution blocked | Add `shell` to allowTools |
| `claude: command not found` | Claude Code not installed | `npm i -g @anthropic-ai/claude-code` |
| `rate limit exceeded` / `429` | API quota hit | Wait and retry |
| Non-zero exit, no match | Unknown error | Review session logs |

The first two patterns — tool approval and shell permission — are **auto-fixable**. The server knows exactly what to change and can apply the fix without human input.

---

## `autoRunUntilNoError()`: The Retry Loop

The new `POST /schedules/:id/run-until-no-error` endpoint triggers a self-healing loop:

```
1. Run the schedule's prompt via Claude Code
2. Wait for the session to complete
3. Read the error report
4. If no error (exit 0): done ✅
5. If auto-fixable error: apply the fix, update the schedule, retry
6. Repeat up to 3 times
7. If still failing: surface the error report for human review
```

For the 2am scenario above, the loop would have:
1. Run the digest prompt → Claude Code hits `web_fetch` approval error
2. Read error report: `{ autoFix: { type: 'add-tool', tool: 'web_fetch' } }`
3. Append `web_fetch` to the schedule's `allowTools` → save to `~/.yelly-time/schedules.yaml`
4. Retry the prompt → Claude Code succeeds
5. Morning digest arrives on time

No human required. The schedule heals itself and persists the fix to disk so it doesn't regress on the next run.

---

## What It Looks Like in the UI

The yelly-time Server Manager gets two new affordances.

**On the session page** (when a session fails): a collapsible error report panel appears below the output, showing root cause and resolution in plain language. No more scrolling through ANSI-colored terminal output to find the one line that matters.

**On the schedules table** (when `lastRunFailed` is true): a `🔁 Auto-fix` button appears next to the schedule's run button. One click triggers `run-until-no-error`. The button shows a live attempt counter as the loop runs.

The design principle: **errors should be actionable at a glance.** The session logs are still there for deep debugging. But the error report is the first thing you see, and it tells you exactly what to do.

---

## What Can and Can't Be Auto-Fixed

The loop is deliberately conservative. It only applies fixes it's certain about:

**Auto-fixable:**
- Missing tool in `allowTools` (the most common failure mode for scheduled Claude Code runs)
- Shell permission blocked (common when a schedule needs to run a script)

**Report-only (human required):**
- Claude Code not installed — the server can't install software on your behalf
- API rate limits — the fix is "wait," which the server can do, but the root cause is external
- Unknown errors — anything that doesn't match a known pattern gets surfaced for human review

The `maxRetries` default is 3. After three attempts with the same error, the loop stops and marks the schedule as failed. This prevents infinite loops on errors that auto-fix can't actually resolve.

---

## Why This Matters for Scheduled AI Agents

Scheduled agents are fundamentally different from interactive ones. When you're sitting at a terminal running Claude Code, you see the error immediately and fix it. When a schedule fires at 2am, the error sits there until someone checks.

The longer an error sits, the more it compounds. A digest that doesn't arrive means the team starts their day without context. A weekly report that silently fails means decisions get made on stale data. A nightly code analysis that stops running means regressions accumulate undetected.

`autoRunUntilNoError()` closes this gap for the class of errors that have deterministic fixes. Tool permission errors are the most common failure mode for new schedules — you set up the prompt, forget to add a tool, and the first run fails. With the auto-fix loop, that first failure is also the last one.

---

## Setting It Up

If you're running yelly-time with Claude Code:

```bash
node server.js --runtime claude-code
```

The error report is generated automatically for every failed session — no configuration needed. The `run-until-no-error` endpoint is available for any schedule.

To trigger it manually from the UI: find a schedule with `lastRunFailed: true` in the schedules table, click `🔁 Auto-fix`. To trigger it via API:

```bash
curl -X POST http://localhost:2026/schedules/<id>/run-until-no-error \
  -H "X-YellyTime-Token: $(curl -s http://localhost:2026/token | jq -r .token)"
```

The response tells you how many attempts it took and whether it succeeded:

```json
{
  "success": true,
  "attempts": 2,
  "sessionId": 47,
  "fixesApplied": [{ "type": "add-tool", "tool": "web_fetch" }]
}
```

---

## What We Learned

**The most common schedule failure is a missing tool.** In our testing, roughly 60% of first-run failures for new Claude Code schedules were tool approval errors. These are entirely preventable with `autoRunUntilNoError()` — the schedule heals itself on the first run and never fails the same way again.

**Error reports change the debugging experience.** Before: open session logs, scroll to the end, grep for "error," try to understand what Claude Code was attempting when it failed. After: read two lines. The time savings compound across every failed session.

**Auto-fix should be conservative.** We considered auto-fixing rate limit errors by adding a retry delay. We didn't — because rate limits often signal a deeper problem (prompt too expensive, schedule too frequent) that deserves human attention. The loop fixes what it's certain about and surfaces everything else.

**The fix persists.** When the loop adds a tool to `allowTools`, it writes to `~/.yelly-time/schedules.yaml`. The next scheduled run uses the updated config. You don't have to remember to update the schedule manually.

---

*Questions or ideas? Open an issue at [yellyloveai-ops/yelly-time](https://github.com/yellyloveai-ops/yelly-time).*
