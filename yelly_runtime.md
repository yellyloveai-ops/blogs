# The One-Shot Agent Is a Lie

*5 min read · yelly-time community*

---

Every AI agent demo shows the same thing: you type a prompt, the agent does something impressive, you close the terminal.

That's not how useful work gets done.

The most valuable thing an AI agent can do isn't answer a question once. It's show up every morning, do the same thing it did yesterday, and tell you when something changed. That's the job the yelly-time scheduler was built for.

---

## The Problem with One-Shot

When you run Claude Code interactively, you're the scheduler. You decide when to run it, what to ask, and whether to check the output. That works fine for exploratory tasks — debugging a specific issue, drafting a document, answering a question.

But most of the work that matters isn't exploratory. It's repetitive:

- Check if any open PRs are stale
- Summarize overnight activity across your repos
- Scan for new security advisories in your dependencies
- Post a morning digest to Slack before standup
- Verify that last night's deployment actually succeeded

These tasks have something in common: they're worth doing every day, but not worth doing manually every day. The moment you have to remember to run them, they stop getting run.

The scheduler solves this. You write the prompt once. yelly-time runs it on a timer.

---

## What a Schedule Actually Is

A yelly-time schedule is three things:

1. **A prompt** — the same thing you'd type interactively, written once
2. **A frequency** — every 30 minutes, every hour, every day, every week
3. **A set of allowed tools** — what Claude Code is permitted to use without asking

That's it. No cron syntax. No shell scripts. No infrastructure to maintain.

```bash
node server.js --runtime claude-code
```

Open the Server Manager at `http://localhost:2026`, click **➕ Add Schedule**, and fill in three fields. The schedule starts running immediately.

---

## The Patterns That Work

After running schedules in production for a few months, a few patterns have emerged as reliably useful.

**The morning digest.** A schedule that fires at 8am, scans GitHub activity from the past 24 hours, and posts a summary to Slack. The team starts their day with context instead of spending the first 20 minutes of standup reconstructing it.

**The stale PR scanner.** A daily schedule that checks for open PRs older than 3 days with no activity, and posts a reminder to the author. Reduces the "I forgot about that PR" problem to near zero.

**The deployment verifier.** A schedule that runs 30 minutes after each deployment window, checks that the service is healthy, and posts a green/red status. Catches silent failures before they become incidents.

**The dependency watcher.** A weekly schedule that scans `package.json` or `requirements.txt` for packages with known CVEs and files a ticket if any are found. Security hygiene that actually happens.

What these have in common: they're tasks where the value comes from consistency, not from the quality of any single run. A human doing them manually would skip them when busy. The scheduler never skips.

---

## The Scheduler Changes the Economics

Here's the shift that matters: when a task costs nothing to run, you run it more often.

Interactively, running Claude Code has a cost — you have to be there, you have to wait, you have to read the output. That cost shapes which tasks you bother with. You run the important ones. You skip the ones that are "probably fine."

With a schedule, the cost of running drops to zero. So you run everything. The stale PR check that would take 5 minutes manually? Schedule it. The dependency scan you do quarterly because it's tedious? Schedule it weekly. The deployment verification you skip when you're in a hurry? Schedule it automatically.

The economics flip. Instead of asking "is this worth my time?", you ask "is this worth a prompt?" Almost everything is worth a prompt.

---

## What the Scheduler Isn't

The scheduler isn't a replacement for judgment. It's a replacement for memory.

Claude Code running on a schedule will do exactly what you told it to do, every time, without forgetting. But it won't decide what's worth doing. That's still your job.

The best schedules are ones where you've already decided the task is worth doing — you just haven't been doing it consistently. The scheduler closes the gap between "I should do this every day" and "I actually do this every day."

It also isn't magic. Schedules fail. Tools get blocked. APIs rate-limit. The output isn't always what you expected. That's why yelly-time generates an error report for every failed run and shows it in the Server Manager — so you can see what went wrong and fix it, or let the auto-fix loop handle it automatically.

---

## Getting Started

The fastest path to a useful schedule:

1. Find a task you do manually more than once a week
2. Run it interactively with Claude Code once, and note what tools it used
3. Create a schedule with the same prompt and those tools in the allowed list
4. Let it run for a week

After a week, you'll either have a schedule that's saving you time, or you'll have learned something about what the task actually requires. Either outcome is useful.

The hardest part isn't the technical setup. It's identifying the right tasks. The right tasks are the ones you do consistently but don't enjoy — the ones where the value is in the output, not the process of producing it.

Those are the tasks worth scheduling.

---

*Questions or ideas? Open an issue at [yellyloveai-ops/yelly-time](https://github.com/yellyloveai-ops/yelly-time).*
