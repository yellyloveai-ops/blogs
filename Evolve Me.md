# Evolve Me: The Feature That Lets Your Software Improve Itself

*5 min read · YellyRock team*

---

Every software team has a backlog of "nice to have" improvements that never get prioritized. The feature is small — a badge here, a keyboard shortcut there — but it's never urgent enough to displace real work. It sits in the backlog for months. Sometimes forever.

**Evolve Me** is YellyRock's answer to that problem. It's a tab in the YellyRock panel where you describe what you wish the tool did better, click one button, and an AI agent reads the codebase, implements the change, and raises a PR for review. No ticket. No sprint planning. No waiting.

---

## The 5-Minute Loop

A user wanted to configure YellyRock to use Claude's 1M context model. They opened a GitHub issue asking if it was possible.

The answer: "Use Evolve Me to make the change."

Here's what happened next:

```
14:47  User types feedback in Evolve Me:
       "set claude-opus-4.6-1m as default model in agent spec"

14:47  Clicks [🧬 Mutate Now]

14:50  Claude Code raises a PR:
       agents/yellyrock-spec.yaml
       + "model": "claude-opus-4.6-1m"

14:52  Maintainer approves → merged ✅

Total: 5 minutes · 2-minute PR lifecycle
```

The user didn't need to know the codebase. They didn't need to find the right file, understand the agent-spec format, or figure out the commit conventions. They described what they wanted in plain English. The agent did the rest.

---

## How It Works

The Evolve Me tab lives inside the YellyRock full-view panel (the 🔮 icon). The UI is intentionally minimal:

1. **Describe the change** — a textarea with examples to prompt specificity
2. **Click [🧬 Mutate YellyRock Now]** — triggers Claude Code via yelly-time, or falls back to a cloud runtime
3. **Review the PR** — the agent posts the link; you approve or comment

Under the hood, the agent follows a fixed workflow:

```
User feedback
      │
      ▼
Claude Code invoked via yelly-time
  1. Clone the YellyRock repo locally
  2. Read AGENTS.md + relevant source files
  3. Implement minimal change
  4. Commit (conventional commit format)
  5. git pull --rebase && open PR
  6. Post PR link back to user
      │
      ▼
PR on github.com
      │
      ▼
Maintainer reviews & approves
      │
      ▼
Merge → Production ✅
```

The agent reads `AGENTS.md` first — the architecture guide that tells it which files to touch, which patterns to follow, and which decisions are one-way doors. That's what keeps the changes coherent rather than random.

---

## What "Each Software" Actually Means

The phrase "for each software" is the key insight. Evolve Me isn't just for YellyRock. The same pattern works for any codebase that has:

- An `AGENTS.md` with architecture context
- A git repo the agent can clone
- A PR workflow for review

The Evolve Me prompt template is parameterized — it injects the user's feedback into a structured workflow that any agent-friendly codebase can execute. If your repo has an agent readiness score above 80, you can wire up the same loop.

The `eval-agent-readiness-score` skill tells you where you stand. The `improve-agent-readiness-score` skill closes the gaps.

> [▶ Try eval-agent-readiness-score](https://github.com/yellyloveai-ops/yellyrock?yellyrock_id=eval-agent-readiness-score)

---

## The Feedback History

Every submission is saved to cross-site storage (last 5 entries). The history panel shows what you've asked for and when — useful for tracking what's been implemented and what's still pending.

The textarea also persists across page reloads. If you start drafting feedback and close the panel, it's still there when you come back.

---

## What Makes This Different from a Feature Request

A feature request goes into a backlog. Someone triages it. Someone estimates it. Someone schedules it. Six weeks later, maybe it ships.

Evolve Me skips all of that. The feedback goes directly to an agent that implements it now. The only human in the loop is the PR reviewer — and that review takes minutes, not weeks.

The tradeoff: the agent implements what you described, not necessarily what you meant. Specificity matters. "Show a badge on new skills added in the last 7 days" is actionable. "Make the UI better" is not.

The examples in the textarea placeholder are there for a reason:
- *In the Discover tab, show a "New" badge on skills added in the last 7 days*
- *In the dash button mini view, add a tooltip showing the skill description on hover*
- *Add a keyboard shortcut (Alt+A) to toggle the panel*

Each example names a location, a behavior, and a trigger. That's the level of specificity the agent needs to produce a clean, minimal change.

---

## What We Learned

**The bottleneck was never implementation — it was prioritization.** Small improvements die in backlogs because they're never urgent enough. Evolve Me removes the prioritization step entirely. If you can describe it in a sentence, it can ship this afternoon.

**AGENTS.md is load-bearing.** The agent's ability to make coherent changes depends entirely on the architecture context in `AGENTS.md`. Repos without it produce changes that technically work but don't fit. The readiness score reflects this — agent-friendly codebases get better Evolve Me results.

**The PR is the safety net, not the bottleneck.** Some teams worried that removing the planning step would produce low-quality changes. In practice, the PR review catches the edge cases. The agent's first attempt is usually 80–90% right; the reviewer catches the rest. That's a better ratio than most junior engineers on their first PR.

**Feedback quality compounds.** The more specific your feedback, the better the implementation. Teams that started with vague requests ("improve the UX") learned quickly to be precise. After a few iterations, the feedback quality improved — and so did the implementation quality.

---

## Try It

Open YellyRock → click the 🔮 icon → **Evolve Me** tab.

Describe one thing you wish YellyRock did better. Be specific about where the change should appear. Click **🧬 Mutate YellyRock Now**.

The PR will be ready for review before your next meeting.

---

*Questions or ideas? Open an issue at [yellyloveai-ops/yelly-time](https://github.com/yellyloveai-ops/yelly-time).*
