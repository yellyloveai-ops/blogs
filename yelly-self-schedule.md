

# Tell the AI What You Want Done. It Schedules Itself.


*5 min read · YellyRock team*


---


There's a specific kind of friction that kills automation before it starts: the setup tax.


You know the task should be automated. You know the agent can do it. But before any of that happens, you have to figure out the cron syntax, pick the right interval, decide when the first run should fire, and wire it all together in a config file. By the time you've done all that, you've spent more time on the scaffolding than the task itself.


YellyClaw's **self-schedule** feature removes the setup tax entirely. You type what you want in plain English. The AI figures out the rest.


---


## The One-Line Interface


The schedule panel in YellyClaw has a single input at the top:


```
e.g. check my issues every day at 9am
```


That's it. You describe the task and the cadence in natural language. Click **⚡ Add with AI**. An agent parses your description, infers the interval, sets the first run time, and calls the schedule API directly — no form to fill out, no cron expression to remember.


The result appears in the schedule table within seconds, ready to run.


---


## What the AI Actually Does


When you click **⚡ Add with AI**, YellyClaw:


1. Fetches a fresh CSRF token from `/token`
2. Constructs a structured prompt that includes your description, the token, and the exact `curl` command format for `POST /schedules`
3. Launches a kiro-cli session via `/run` with `allowTools: ["shell"]`
4. The agent parses your intent, fills in the fields, and executes the curl immediately


The field mapping the agent handles:


| Your words | What gets set |
|---|---|
| "every day at 9am" | `interval: 1d`, `nextRunAt: today 09:00 local` |
| "every hour" | `interval: 1h`, `nextRunAt: now+1min` |
| "check my issues" | `prompt: "Check my open GitHub issues and summarize..."` |
| "weekly on Monday" | `interval: 1w`, `nextRunAt: next Monday` |


The agent also sets `autoPause: true` by default — if a run fails, the schedule pauses rather than hammering a broken task repeatedly.


After the curl succeeds, the session tab opens automatically so you can watch the first run happen in real time. The schedule table polls every 5 seconds and updates as soon as the new entry appears.


---


## A Real Example


Here's what a typical interaction looks like:


```
Input: "summarize my open pull requests every morning at 8am"


Agent creates:
 name:       "Summarize open PRs"
 prompt:     "Check my open pull requests on GitHub and post
              a summary: how many are open, which need my attention,
              which are waiting on others."
 interval:   1d
 nextRunAt:  2026-03-14T08:00:00 (tomorrow morning)
 autoPause:  true


Result: schedule appears in table, first run tomorrow at 8am
```


The agent extracted the task (summarize open PRs), the cadence (daily), and the time (8am) from a single sentence. You didn't touch a form.


---


## The Manual Path Still Exists


The AI path is the default, but it's not the only path. The **➕ Manual** button opens the full schedule form — prompt textarea, name field, agent selector, frequency dropdown, first-run datetime picker, tool checkboxes, and the autoPause toggle.


The manual form is useful when:
- The prompt is long and multi-step (the AI path works best for concise descriptions)
- You need to specify a custom agent (`yellyrock-socialize` instead of the default)
- You want to grant additional tools (`playwright`, `slack_mcp`) that the AI path doesn't configure


Both paths write to the same schedule store. The table shows everything together, sorted by urgency: enabled schedules with the soonest next run at the top.


---


## What Happens After Creation


Once a schedule exists, YellyClaw manages the lifecycle:


- **⏸ / ▶** — pause or resume with one click
- **⚡** — trigger an immediate run outside the normal cadence (with a cooldown to prevent double-fires)
- **✏️** — edit any field in place; the next run time updates immediately
- **🔁** — auto-fix: if the last run failed, retry with automatic error correction via `/run-until-no-error`
- **🗑** — delete with inline confirmation; active sessions keep running but become orphaned


The sessions panel on the right shows every run tagged with a **⏰** badge for schedule-triggered sessions. Click the run count in the schedule table to filter the sessions panel to just that schedule's history.


---


## The Bigger Pattern


Self-schedule is the last piece of a loop that was already mostly there.


YellyClaw could already run any prompt on demand. It could already spawn child sessions in parallel. It could already persist sessions to disk and export them. What was missing was the ability to set up recurring work without thinking about it.


Now the loop is complete:


```
You describe the task once
     │
     ▼
AI creates the schedule
     │
     ▼
YellyClaw runs it on cadence
     │
     ▼
Sessions accumulate in history
     │
     ▼
You review exceptions, not status
```


The tasks that benefit most are the ones you were already doing manually on a rhythm — daily standup prep, weekly dependency checks, morning issue triage, end-of-sprint summaries. Each one is a sentence. Each sentence becomes a schedule. Each schedule runs while you're doing something else.


---


## What We Learned


**Natural language scheduling removes the last excuse.** The friction wasn't the task — it was the setup. When setup costs one sentence, the bar for "worth automating" drops to almost zero.


**The agent is better at interval inference than you'd expect.** "Every weekday morning" → `1d` with a Monday–Friday guard in the prompt. "Twice a week" → `1w` with the prompt specifying two runs. The agent handles ambiguity by making a reasonable choice and explaining it in the session output.


**autoPause is the right default.** Early testing without it produced schedules that ran repeatedly on broken prompts, filling the history with identical failures. Pausing on first failure forces a human to look at what went wrong before the schedule resumes. That's the right tradeoff.


**The manual form is still worth having.** Power users with complex multi-step prompts, custom agents, or specific tool requirements need the full form. The AI path is the 80% case; the manual path handles the rest.


---


*Questions or ideas? Open an issue at [yellyloveai-ops/yelly-time](https://github.com/yellyloveai-ops/yelly-time).*



