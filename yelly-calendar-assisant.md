# Using AI to Self-Optimize an AI Calendar — and It Actually Works


*5 min read · YellyRock team*


Here's a pattern that sounds recursive at first: using an AI agent to manage and optimize the schedule of other AI agents. But once you try it, it stops feeling clever and starts feeling obvious. This is what "AI as a daily assistant" actually looks like in practice.


## The Problem: Schedules Pile Up


You start with one scheduled agent task. Then two. Then a dozen. Before long, everything fires at 9 AM, they compete for resources, and half of them finish late or fail silently. You know the fix — spread them out, group similar tasks, stagger the heavy ones — but who has time to manually reschedule twelve tasks?


That's the exact problem we ran into with YellyClaw (the yelly-time server manager). The solution turned out to be: let an AI agent do it.


## How It Works


The flow is simple:


1. **Read**: The agent reads all current schedules — names, prompts, run times, frequencies
2. **Reason**: It reasons about distribution and grouping (code reviews together, standups in the morning, reports at end-of-day)
3. **Propose**: It outputs a diff — what would change, what stays the same
4. **Human reviews**: You see a preview table before anything is applied
5. **Apply or cancel**: One click to commit, one click to walk away


The key insight is step 4. The agent doesn't just act — it proposes. You stay in control with full visibility into every change before it lands.


## Full Visibility Is the Point


The preview diff looks like this:


| Task | Old Time | New Time | Note |
|------|----------|----------|------|
| Daily standup | 09:00 | 08:30 | — |
| Code review check | 09:15 | 10:00 | Renamed: "Morning code review" |
| Issue triage | 09:30 | 14:00 | — |
| Weekly report | 09:00 | 17:30 | Renamed: "EOD weekly report" |


Green = time changed. Blue = name suggestion. Nothing is hidden. If you don't like it, cancel — your schedules are untouched.


This is what makes human oversight *easy* rather than burdensome. You're not auditing a black box; you're reviewing a clear, bounded proposal. The agent did the tedious reasoning work; you make the final call in 10 seconds.


## What the Agent Can (and Can't) Change


This constraint was deliberate:


| Field | Can change? |
|-------|-------------|
| Time of day (HH:MM) | ✅ |
| Display name | ✅ (optional suggestion) |
| Frequency / interval | ❌ Never |
| Date | ❌ Never |
| Prompt content | ❌ Never |


If you set something to run every 6 hours, it still runs every 6 hours — just maybe at 10:00 instead of 09:00. The agent optimizes *within* your intent, not around it.


## Human Calendar ↔ AI Calendar: Two Sides of the Same Day


The AI schedule doesn't exist in isolation — it maps directly to your human calendar. Every meeting, deadline, or recurring event you have can have a corresponding AI task that runs *before* it.


The simplest example: a 1:1 or design review on your calendar tomorrow at 2 PM. The AI calendar can automatically schedule a **meeting prep task** to run the day before at 5 PM — pulling the agenda, summarizing recent context, and drafting talking points so you walk in prepared.


| Human Calendar Event | AI Calendar Task | Runs |
|----------------------|-----------------|------|
| Weekly 1:1 (Mon 2 PM) | Prep: summarize open items, draft agenda | Sun 5 PM |
| Design review (Thu 10 AM) | Prep: pull latest doc, summarize changes | Wed 5 PM |
| Sprint planning (Fri 9 AM) | Prep: triage backlog, flag blockers | Thu 5 PM |
| Monthly report due (1st) | Draft: pull metrics, generate summary | Last day of month |


This is the **human-AI calendar bridge**: your human schedule drives the AI schedule. When a meeting moves, the prep task moves with it. When a meeting is cancelled, the prep task is skipped.


The agent doesn't replace your calendar — it reads it and acts on it. You stay in control of your time; the AI handles the preparation work that used to fall through the cracks.


## It Self-Adds and Self-Improves Based on Your Goals


The more interesting capability is when you give the agent a goal rather than a task list. Tell it: *"I want to stay on top of open PRs and keep my inbox under control"* — and it can propose new scheduled tasks, not just rearrange existing ones.


This is the self-add loop:
- You describe a goal or pain point
- The agent proposes one or more new scheduled tasks to address it
- You review and approve
- Those tasks run on schedule, and the agent can later suggest refinements based on what's working


Over time, your schedule becomes a living artifact that reflects your actual priorities — not a static list you set up once and forgot.


## What We Learned


**Async execution matters.** Optimization prompts can take 30–60 seconds for large schedule sets. Launching a real agent session and polling for completion keeps the UI responsive and gives you a live log to inspect if something goes wrong. A synchronous spinner would have felt broken.


**JSON output contracts are fragile.** The prompt says "output ONLY a valid JSON array." Models sometimes wrap it in a code fence anyway. The parser scans the full session output for the last valid JSON array matching the expected shape — this handles most model quirks without needing a strict parser.


**"No changes needed" is a valid result.** If your schedules are already well-distributed, the preview doesn't appear — you get a green status message instead. This avoids the awkward "AI suggested nothing" empty state.


**The calendar view makes it real.** Seeing your agent tasks on a 2-week grid — past runs, active sessions, upcoming occurrences — turns an abstract schedule list into something you can reason about at a glance. The optimization feels meaningful because you can see the before/after in context.


## The Bigger Pattern


What we built is a small instance of a general idea: **AI agents that manage other AI agents, with humans staying in the loop through visibility rather than micromanagement**.


The human's job shifts from "configure every detail" to "review proposals and set goals." The agent's job is to handle the tedious reasoning in between. That division of labor is what makes AI feel like a genuine daily assistant rather than a fancy script.


The results have been promising. Schedules are more evenly distributed. Tasks get better names. New tasks get added that we wouldn't have thought to add manually. And the whole thing takes about 10 seconds of human attention per optimization cycle.


*Questions or ideas? Open an issue at [yellyloveai-ops/yelly-time](https://github.com/yellyloveai-ops/yelly-time).*



