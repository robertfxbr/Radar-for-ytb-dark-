# Radar de Outliers

A Claude Code skill for YouTube channel operators (faceless/dark, or on-camera) that runs a
manual, no-tooling competitor-scan cycle to calibrate video titles and thumbnails against real
external signal — not by copying, but by finding the pattern behind what's currently breaking
away from other channels' own medians.

## What it does

Instead of judging title/thumbnail ideas only from your own channel's small sample of past
videos, this skill walks through a repeatable cycle:

1. Scan a corpus of competitor/adjacent channels' **most recent** uploads (never "most popular").
2. Find each channel's own outliers — videos at **≥3× that channel's recent median** views.
3. Extract title/thumbnail attributes that are **present and absent** across the outliers.
4. Write one testable hypothesis and at most 5 actionable patterns.
5. Feed those patterns into a title-scoring flow and a "describe the pattern, then create
   original" visual-reference flow — never copying a competitor's actual title or thumbnail.
6. Log the cycle so the next one builds on it instead of starting cold.

It's built to **adapt to whatever channel it's installed into** — on first run in a new project it
looks for your existing channel context (README, style guide, decisions log) and asks rather than
assumes, instead of hardcoding one channel's niche or format.

## Install

Copy `.claude/skills/radar-outliers/` into your own project's `.claude/skills/` directory:

```bash
mkdir -p .claude/skills
cp -r radar-outliers-skill/.claude/skills/radar-outliers .claude/skills/
```

Claude Code picks it up automatically — the skill triggers when you mention "radar", "outliers",
competitor research, or when you're about to lock in a title/thumbnail.

## Design principles

- **Manual by design.** No API key, no scraper, no scheduled job. The skill formalizes a
  *process*, not a tool — if you want to automate collection later, that's your own decision to
  make and document, not something the skill assumes for you.
- **Adapts, doesn't assume.** No hardcoded niche, corpus, or file paths. First run in a project
  spends one pass learning your channel's context instead of guessing.
- **Pattern, never composition.** Nothing from a competitor's video — title, thumbnail layout, or
  copy — is ever reused directly. Only the abstract pattern behind it crosses into your own work,
  and it's rebuilt with your own assets.
- **Cites its own uncertainty.** Every pattern the skill produces is reported with its basis (how
  many outliers, from how many independent channels) — never presented as a proven rule from a
  small sample.

## License

MIT — see [LICENSE](LICENSE).
