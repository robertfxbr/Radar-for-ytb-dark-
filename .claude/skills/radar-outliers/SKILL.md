---
name: radar-outliers
description: Runs a manual outlier-scan cycle against competitor/adjacent channels to extract title and thumbnail patterns before finalizing a video's packaging. Adapts to whatever channel or niche it's installed into — faceless, dark, or on-camera. Use whenever the user is about to lock in a title or thumbnail, when they mention "radar", "outliers", competitor research, benchmarking other channels, "what's working right now", or ask how similar channels are packaging a topic. Trigger it proactively before finalizing packaging on a new video if no cycle exists yet for that topic/niche, or the most recent one is stale (roughly 2+ weeks old).
---

# Radar de Outliers

Most channels calibrate packaging only from their own history — a small sample, and one that
can't tell you whether an idea is universally strong or just strong *for this channel*. The Radar
adds an external signal: scan channels in the same or an adjacent niche, find the videos that
broke away from that channel's own median, and extract *why* — not to copy, but to calibrate a
hypothesis before writing the title/thumbnail.

**This is a manual, no-tooling process by design.** No API key, no scraper, no scheduled job.
Don't build or suggest automation unless the user explicitly asks for it — this skill formalizes
the *process*, not a tool. If a future iteration should use an API or script, that's the host
project's decision to make and record, not something this skill assumes.

## First run in a new project — adapt before scanning

This skill ships generic. Before the first cycle in a given project, spend one pass establishing
context instead of assuming EveryPlaceExplained-style "1 country / 10 cities" packaging:

1. **Find the channel's own context.** Look for a README, CLAUDE.md/AGENTS.md, a strategy or
   style-guide doc, or a decisions log already in the project. Read whatever exists: niche,
   audience, current title/thumbnail conventions, on-camera vs. faceless, any packaging rules
   already in force. If nothing exists, ask the user directly rather than guessing — a wrong
   assumption here (e.g., treating a finance channel like a travel channel) poisons every cycle
   that follows.
2. **Find where to log a cycle.** Look for an existing strategy/docs folder pattern in the
   project. If nothing fits, ask the user where cycles should live, or default to creating
   `radar-outliers/CYCLES.md` at the project root and say so explicitly in your first reply so the
   user can redirect you.
3. **Confirm the corpus.** Ask the user for their competitor/adjacent-channel list if one isn't
   already documented somewhere in the project, rather than inventing channel names. A wrong
   corpus produces confident-sounding nonsense.

Once that context exists (in the project's own docs, not hardcoded here), every later cycle can
skip straight to scanning.

## When to run a cycle

- Before finalizing the title and visual promise of a video — this Radar step is meant to sit
  *inside* whatever packaging workflow the channel already has, feeding it external data rather
  than replacing editorial judgment.
- Periodically (every ~2 weeks is a reasonable default, adjust to the channel's upload cadence)
  even without a video in flight, to keep the corpus and patterns current.
- Whenever the target topic/niche changes — a cycle run for one topic doesn't transfer cleanly to
  another; scan fresh, focused on the actual next video.

## The cycle

### 1. Read what's already known

Check the project's own cycle log (see "First run" above) for the most recent cycle on this
topic/niche. Don't re-run a scan whose corpus and hypothesis are still fresh — read it, decide if
it still applies, and only re-scan if the topic changed or the last cycle is stale.

### 2. State the target

One sentence: which video/topic this cycle serves, and what question it needs to answer (title?
thumbnail? both? is the underlying angle/thesis even settled yet?). Skipping this turns the scan
into unfocused data collection.

### 3. Scan the corpus

For each channel in the corpus: open the **"Latest" / "Most recent" tab, never "Most popular."**
"Most popular" mixes videos from different years and different algorithm regimes — it shows what
worked *once*, not what's working *now*, which is the only thing actionable.

- Window: last ~60–90 days (tighten for a fast-moving niche, widen for a slow-cadence channel).
- Compute the **median** (not mean) view count of the last ~10 uploads. A single viral outlier
  distorts a mean and breaks the whole comparison.
- Flag anything at **≥3× that channel's own median** as an outlier for this cycle. Treat 3× as a
  starting threshold, not a law — adjust it after 2–3 cycles if it produces too many or too few
  outliers, and log the adjustment as a decision, not a silent tweak.
- Record: multiple, view count, days since publish, views/day. A 4× outlier at 30 days is a
  stronger signal than a 4× outlier at 400 days.
- If there's no official Data API access configured, use public search/the channel's own recent
  uploads page directly — but **force the results into English (or the corpus's actual language)**
  before reading titles; browser auto-translate silently corrupts a title-pattern analysis and is
  easy to miss until the data already looks wrong.

**Statistical independence rule:** multiple outliers from the same channel using the same visible
template (same title formula, same thumbnail layout) count as **one piece of evidence**, not one
per video. Don't let five near-identical uploads from one prolific channel outweigh five
independent channels agreeing on something.

### 4. Extract attributes — present *and* absent

Build two lists, from title and thumbnail, **before** drafting any title candidates — extracting
attributes after you already have candidates in mind contaminates the read; you'll "find" the
pattern you wanted to find. A starting list to adapt per niche:

**Title:** explicit decision/command word ("Don't", "Stop", "Avoid") · concrete number/price ·
named place/subject · question · implied conflict or mistake · direct comparison · year present ·
payload before the mobile-truncation point (~character 45–50) · recurring word across the cycle ·
superlative · second person ("you/your").

**Thumbnail:** single dominant subject · human presence/face · large number · 1 vs. 2 vs. 3+ text
zones · high contrast · recognizable landmark/object · map/diagram · arrow/circle callout.

> **Absence is data.** If *none* of this cycle's outliers use a year, or *none* use a face, that's
> as informative as what's present — and it's the kind of signal that gets missed when you only
> catalog what shows up. Log both lists every cycle, not just the positive one.

### 5. Write the hypothesis

One testable sentence per cycle, in the form: *"the dominant pattern in this cycle's outliers is
X, appearing in N of M independent sources; hypothesis: Y should outperform the current
convention."* Without this sentence written down, there's no way to grade the cycle's usefulness
after the video ships.

### 6. Produce at most 5 actionable patterns

```
PATTERN: <what's observed, in one sentence>
BASIS:   <how many outliers, from how many independent channels/voices, support this>
ACTION:  <what concretely changes in the next video's title/thumbnail>
```

A pattern that doesn't change a real decision doesn't belong in the list. If a cycle produces zero
actionable patterns, say so explicitly — that's a wasted cycle, and it's still worth recording as
one (it tells you the corpus or niche needs revisiting).

### 7. Feed the title / visual-reference flow

Apply the cycle's patterns inside whatever title and thumbnail process the host channel already
has. If none exists yet, a reasonable default:

- **Title:** `OUTLIERS → ATTRIBUTES → FREQUENCY → HYPOTHESIS → CANDIDATES → SCORE → GATES → TITLE`.
  Score candidates by attribute adherence (+1 per present pattern followed, −1 per recurring-absent
  attribute a candidate reintroduces). Gates are pass/fail regardless of score — typically: reads
  correctly when truncated on mobile, the promise is actually delivered early in the video, the
  packaging family/angle isn't repeated from the last video or 3 of the last 5, and title +
  thumbnail don't say the *same half* of the message. **Never take a competitor's actual title and
  swap in your own subject** — it was calibrated for their audience and their video, not yours.
- **Visual reference:** `REFERENCE → PATTERN DESCRIPTION → CLOSE THE REFERENCE → ORIGINAL CREATION`.
  Look at the outlier's thumbnail, write down the abstract pattern in words ("one dominant subject,
  two large words, high contrast, no map"), then close the image and never look at it again while
  building the actual thumbnail. **Never feed a competitor's thumbnail into an AI image tool as a
  style/composition reference**, and never reproduce a composition near pixel-for-pixel — swapping
  a color doesn't make a cloned layout original. If the host project has a rule against
  AI-generated imagery for real assets, that rule still applies in full; this skill never creates
  an exception to it.

### 8. Log the cycle

Append the cycle to wherever cycles are being tracked in this project (see "First run" above):
objective, corpus used, outlier table, present/absent attributes, the hypothesis sentence, the
patterns (≤5), and the real limitations of that cycle's sample. If the resulting editorial call
(final title, a topic-order change) came from more than just this cycle's data, say so — and if
the host project has its own decisions log, record the decision there too, not just the finding.

## What this skill is not

- **Not "chase trends."** Editorial choice should stay some version of
  `EXTERNAL SIGNAL + right-fit topic + strong angle + fits this channel` — a topic can still be
  chosen for purely editorial reasons with no Radar signal behind it, but that should be explicit,
  not implied.
- **Not a replacement for fact-checking.** Numbers, prices, and claims in the actual video still
  need their own verified sourcing. The Radar informs angle and packaging, never facts.
- **Not a source of content.** Nothing observed in a competitor's video becomes a script line,
  image, or thumbnail composition in the host channel's own work — only the *abstract pattern*
  ever crosses that line, never the specific execution.
- **Not automation.** This phase is browser + a table. If a user wants to automate collection,
  treat that as a new decision to make and document, not a natural extension of this skill.

## Honest limits

Corpora built this way are small (handfuls of channels, often under 20 independent "voices" per
cycle once same-channel/same-template clusters are collapsed to one vote each), and collection via
a channel's public uploads page is fragile — it can change layout or behavior without notice. A
pattern backed by 3 of 14 independent sources is the strongest evidence available in that cycle,
not proof. Always cite the basis (how many, out of how many) alongside any pattern — never present
a small-sample pattern as a settled rule.
