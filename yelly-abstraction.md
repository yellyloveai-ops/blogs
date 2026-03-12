# The Runtime Doesn't Matter (Until It Does)

*5 min read · yelly-time community*

---

When we first built yelly-time, it ran one thing: Claude Code.

The server spawned `claude --print`, streamed the output, and called it done. The code was simple. The assumption was simpler: everyone uses Claude Code, so why abstract it?

Six months later, the assumption was wrong. Some teams were on codex-cli. Others were running kiro-cli internally. A few wanted to swap runtimes per schedule — Claude Code for code tasks, a lighter model for digest summaries. The "just hardcode it" decision had become a wall.

So we built the runtime abstraction. This is what we learned.

---

## What "Runtime" Actually Means

An agent runtime, in yelly-time's model, is anything that:

1. Accepts a prompt as input
2. Executes it with access to tools
3. Produces output (stdout/stderr)
4. Exits with a code

That's it. Claude Code does this. codex-cli does this. kiro-cli does this. A shell script that calls an API and prints the result does this. The interface is just a process.

The abstraction isn't about the AI model underneath. It's about the execution contract: give me a prompt, give me a working directory, tell me which tools are allowed, and I'll run it and tell you what happened.

---

## The Concrete Problem

Before the abstraction, `runKiro()` was the only execution path. It hardcoded the binary name, the argument format, the tool trust flags — everything specific to one CLI.

```javascript
// Before: hardcoded to one runtime
const proc = spawn('claude', ['--print', '--allowedTools', tools.join(' '), prompt]);
```

Switching to codex-cli meant rewriting the spawn call, the argument format, the tool flag syntax, and the output parsing. There was no seam to cut at.

The abstraction creates that seam. The server doesn't know which binary it's calling. It knows the contract:

```javascript
// After: runtime-agnostic
const proc = runtime.spawn(prompt, { tools, workDir, interactive });
```

Each runtime adapter translates the contract into the specific CLI invocation it needs.

---

## What Each Runtime Looks Like

The adapters are thin. Most of the complexity lives in the contract, not the implementation.

**Claude Code** (`claude`):
- Tool trust: `--allowedTools tool1 tool2` (space or comma-separated)
- Non-interactive: `--print` (omit for interactive session)
- Output: streams to stdout/stderr, exits 0 on success

**codex-cli** (`codex`):
- Tool trust: `--tools tool1 --tools tool2` (repeated flag)
- Non-interactive: `--quiet` flag
- Output: streams to stdout, exits 0 on success

**kiro-cli** (`kiro chat`):
- Tool trust: `--trust-tools tool1,tool2` or `--trust-all-tools`
- Non-interactive: `--no-interactive`
- Output: streams to stdout/stderr with ANSI codes, exits 0 on success

The differences are real but shallow. Comma-separated vs. repeated flags. Different flag names for the same concept. The adapter pattern handles this without leaking it into the rest of the server.

---

## The Config Flag

The runtime is selected at server startup:

```bash
node server.js --runtime claude-code
node server.js --runtime codex
node server.js --runtime kiro
```

The `--runtime` flag loads the corresponding adapter. Everything else — session management, scheduling, error reports, the Server Manager UI — stays the same. The runtime is a plugin, not a core assumption.

Per-schedule runtime overrides are also supported. A schedule can specify `"runtime": "codex"` to use a different CLI than the server default. This is useful when you want a fast, cheap runtime for high-frequency digest schedules and a more capable one for complex code tasks.

---

## What the Abstraction Unlocks

**Testability.** The fake runtime adapter (`--runtime fake`) runs prompts through a local script that returns canned output. Unit tests for scheduling, error reports, and session management no longer require a real AI CLI to be installed. The test suite runs in CI without credentials.

**Fallback behavior.** When a runtime exits with a non-zero code, the error report layer doesn't need to know which CLI produced the output. It pattern-matches against the output text. The patterns are runtime-specific (Claude Code and codex-cli have different error message formats), but the report structure is the same.

**Runtime health checks.** The `/health` endpoint now reports which runtime is active and whether the binary is reachable:

```json
{
  "status": "ok",
  "runtime": "claude-code",
  "bin": "/usr/local/bin/claude",
  "binReachable": true
}
```

If the binary isn't found, the server starts but marks itself degraded. Schedules won't fire until the runtime is healthy.

**Gradual migration.** Teams running kiro-cli internally can switch to Claude Code by changing one flag. Their schedules, session history, and error reports carry over unchanged. The migration is a config change, not a code change.

---

## What We Didn't Abstract

Not everything belongs behind the interface.

**Tool names are runtime-specific.** Each CLI has its own tool namespace and naming conventions. The preapproval rules in `preapproval-rules.json` are keyed to one runtime's tool names. When you switch runtimes, you may need to update your tool lists — the abstraction doesn't hide this.

**Output format varies.** Claude Code and codex-cli produce different ANSI sequences, different spinner patterns, different progress indicators. The `stripAnsi()` function handles the common cases, but the session logs look different depending on which runtime produced them. This is intentional — the raw output is preserved for debugging.

**Interactive mode semantics differ.** Claude Code's interactive mode waits for stdin. codex-cli's interactive mode is different. The `/sessions/:id/input` endpoint works for runtimes that support stdin injection, but not all do. The session page shows an input field only when the runtime reports `interactive: true`.

---

## The Lesson

The abstraction wasn't planned. It was extracted from pain.

The first version of yelly-time had no seam between "run a prompt" and "run kiro-cli." That was fine when there was only one runtime. It became a problem the moment a second one appeared.

The lesson isn't "always abstract early." It's "know where your assumptions are." The assumption that everyone uses the same CLI was invisible until it wasn't. Once we named it — "this is a runtime assumption, not a core requirement" — the abstraction was obvious.

The same pattern shows up everywhere in agent infrastructure: session storage, tool preapproval, error parsing. Each one starts as a hardcoded assumption. Each one eventually needs a seam. The question is whether you find the seam before or after it becomes a wall.

---

## What's Next

The runtime abstraction opens a few directions we haven't explored yet:

- **Remote runtimes**: instead of spawning a local process, POST the prompt to a remote endpoint and stream the response. The contract is the same; the transport changes.
- **Runtime composition**: run the same prompt through two runtimes and diff the outputs. Useful for evaluating model quality on real tasks.
- **Cost tracking**: each runtime adapter can report token usage. The scheduler can use this to enforce per-schedule budgets.

None of these require changes to the core server. They're adapter implementations. That's the point of the abstraction.

---

*Questions or ideas? Open an issue at [yellyloveai-ops/yelly-time](https://github.com/yellyloveai-ops/yelly-time).*
