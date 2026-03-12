# The Prompt Marketplace That Lives in a JSON File

*5 min read · YellyRock team*

---

Before YellyRock, sharing an AI skill or prompt meant writing a README.

You'd build something useful — a prompt that summarized pull requests, or one that filed tickets from runbooks — and then you'd write a doc explaining how to install it. Maybe a pip package. Maybe a shell script. Maybe just a Slack message with a code block and the instruction "paste this into your terminal."

Most prompts never got shared at all. They lived in someone's `~/.claude/` folder, or in a private repo, or in a wiki page that nobody found. The friction wasn't in building the prompt. It was in the gap between "I have something useful" and "my team can use it."

YellyRock was built to close that gap.

---

## Skills/Prompts Are Just Markdown

The first insight was structural: an AI skill is a prompt, and a prompt is text, and text lives in a file.

A `SKILL.md` file has two parts: a YAML frontmatter block that declares metadata, and a prompt body that tells the agent what to do.

```yaml
---
name: review-doc-against-guidelines
compatibility: cloud
description: Review the current doc against guiding docs and produce structured feedback.
metadata:
  parameters:
    GUIDELINES_URL:
      hint: "URL of the guidelines doc to review against"
      word_limit: 200
---

# Review Doc Against Guidelines

You are reviewing the document currently open in the browser...
```

That's the whole format. No package manifest. No dependency tree. No build step. A skill/prompt is a markdown file you can read, edit, copy, and share like any other document.

This matters because it means prompts can live anywhere code lives — in a git repo, referenced by path, versioned alongside the product they support. When the runbook changes, the prompt that reads it can change in the same commit.

---

## The Catalog: A JSON File That Knows Where You Are

The second insight was about discovery. A prompt that nobody finds is a prompt that doesn't exist.

The YellyRock catalog is a JSON array. Each entry says: here's a skill/prompt, here's where it lives, and here's which pages it should appear on.

```json
{
  "yellyrock_id": "review-doc-against-guidelines",
  "title": "Review Doc vs Guidelines",
  "include": ["docs.mycompany.com/*", "wiki.mycompany.com/*"],
  "exclude": [],
  "file": "skills/review-doc-against-guidelines/SKILL.md",
  "trackingUrl": "https://github.com/my-org/my-repo/issues/42"
}
```

The `include` patterns are the UX. When you open a document, YellyRock checks the current URL against every catalog entry and surfaces only the skills/prompts that make sense on this page. A prompt for reviewing pull requests appears on `github.com/*/pull/*`. A prompt for summarizing wiki pages appears on `wiki.mycompany.com/*`. You never see a prompt that doesn't apply to what you're looking at.

This is the "3.0 way" of prompt distribution: instead of asking users to install a tool and remember when to use it, the skill/prompt shows up at the moment it's relevant. The catalog is the delivery mechanism.

---

## Two Catalogs, One Upgrade Path

YellyRock ships with two catalogs embedded in the browser extension via `@require`:

- **`catalog.json`** — production skills/prompts, visible to all users
- **`alpha.catalog.json`** — prompts in testing, visible to users who opt into alpha

The upgrade path is deliberate. A new prompt starts in alpha. Usage data accumulates. When invocations cross a threshold and the error rate is low, the skill/prompt gets promoted to production via a PR. The catalog is the gating mechanism — not a release pipeline, not a feature flag system, just a JSON file and a code review.

Contributing a skill/prompt means writing a `SKILL.md`, adding an entry to `alpha.catalog.json`, and raising a PR. That's the entire process. No npm publish. No package registry. No approval queue outside the normal code review flow.

---

## BYOS: The Long Tail of Prompts

The production catalog covers broadly useful skills/prompts. But the most valuable prompts for your team are probably the ones that know your specific codebase, your deployment patterns, your runbooks.

Those prompts don't belong in the global catalog. They belong in your team's repo.

BYOS (Bring Your Own Skills) lets any team host a catalog JSON file anywhere — a raw blob URL in a git host, a public gist, an S3 bucket — and share it as a URL. Teammates import it once via the Discover tab, and the skills/prompts appear immediately on matching pages. The catalog syncs every 24 hours.

```
YellyRock Discover tab → paste catalog URL → ⬇ Fetch & Import
```

The BYOS model means prompt distribution doesn't require central coordination. A team can build, share, and iterate on their own catalog without touching the main YellyRock repo. The best prompts from team catalogs eventually get nominated for the production catalog — but that's optional, not required.

---

## The Builder: Prompts Without Touching Code

The third piece is the skill builder. Not everyone who has a useful prompt wants to write YAML frontmatter and raise a PR.

The YellyRock Build tab takes a plain-language description and generates a `SKILL.md` draft:

> *"Build a skill that reviews the current document against our team's writing guidelines and posts feedback as a comment."*

The builder produces a complete skill/prompt file — frontmatter, parameter definitions, prompt body — that you can refine in the editor and test immediately. The test button runs the prompt end-to-end with the parameter collection dialog, so you can verify it works before it goes anywhere near a catalog.

The builder doesn't replace the craft of writing a good prompt. It removes the activation energy of starting from scratch. Most prompts that get contributed to the catalog started as a builder draft that someone refined over a few iterations.

---

## What the Marketplace Actually Looks Like

The result is a marketplace that operates entirely through files and URLs.

| Layer | Mechanism | Who controls it |
|-------|-----------|-----------------|
| Prompt content | `SKILL.md` in a git repo | Prompt author |
| Discovery | `include`/`exclude` URL patterns in catalog | Catalog maintainer |
| Distribution | Bundled in browser extension (prod/alpha) or BYOS URL | YellyRock team / any team |
| Contribution | PR to `catalog.json` or `alpha.catalog.json` | Anyone |
| Team-specific | BYOS catalog hosted anywhere | Any team |

There's no marketplace server. No prompt registry. No install command. The "marketplace" is a set of JSON files that the browser extension reads at page load, matched against the current URL, and rendered as buttons in the panel.

The simplicity is the point. A system that requires no infrastructure to operate is a system that can't go down.

---

## What We Learned

**URL patterns are the hardest design decision.** A prompt with `include: ["github.com/*"]` appears on every GitHub page — useful for broadly applicable prompts, noisy for specialized ones. Getting the patterns right is the difference between a skill/prompt that feels like it belongs and one that feels like an intrusion. The discipline is specificity: if a prompt only makes sense on pull request pages, it should only appear on pull request pages.

**The alpha catalog is underused.** The upgrade path from alpha to production exists, but most prompts stay in alpha longer than they should. The friction isn't technical — it's that nobody checks the usage data. The `analyze-statistics` skill exists specifically to close this loop, but it requires someone to run it. This is a workflow problem, not a tooling problem.

**BYOS is the fastest path to team adoption.** Teams that start with a BYOS catalog — even with two or three prompts — build the habit of using YellyRock before they worry about contributing to the main catalog. The best contributions to the production catalog came from teams that had been running BYOS catalogs for months.

**Prompts get better through use.** The first version of a prompt is almost never the best version. The teams with the highest-quality prompts are the ones that treat `SKILL.md` files like code — iterating on them, reviewing changes, tracking what works. The git history of a well-maintained prompt tells the story of how the team's understanding of the problem evolved.

---

## Try It

The fastest way to contribute a skill/prompt:

1. Open YellyRock → **+** tab
2. Describe what you want the prompt to do
3. Refine the generated draft, test it
4. Add an entry to `alpha.catalog.json` and raise a PR

Or, if you want to keep it within your team: host the catalog JSON anywhere, share the URL, and import it via the Discover tab. No PR required.

The prompt you build this week might be the one your team runs every day next year.

---

*Questions or ideas? Open an issue at [yellyloveai-ops/yelly-time](https://github.com/yellyloveai-ops/yelly-time).*
