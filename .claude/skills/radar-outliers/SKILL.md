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

## Requirements

This skill needs some way to actually view public YouTube pages — a browser tool, or a fetch tool
that renders JavaScript-heavy pages, available to the agent. **If nothing like that is available
in this environment, say so plainly and stop** — ask the user to paste the data manually (channel,
video title, view count, upload date) instead of inventing numbers or guessing at what a channel's
recent uploads probably look like. A fabricated cycle is worse than no cycle: it produces
confident-sounding patterns with no real basis.

**Use a logged-out / private-browsing context when scanning competitors**, not the channel
owner's own logged-in YouTube session — scanning while logged in mixes personal watch-history
signals into recommendations and, more importantly, ties competitor research to the channel's own
account. Public page scraping of this kind sits in a gray area of YouTube's Terms of Service;
treat it as "look, don't scrape at scale" — a handful of manual page loads per channel per cycle,
not thousands of automated requests.

**Keep raw page data out of the conversation.** Don't paste full page HTML, full JSON responses,
or a whole channel's video list into context — extract only title, view count, upload date, and a
short visual description per video as you go, the same way you'd take notes by hand. Scanning
10+ channels' full raw pages verbatim is an easy way to blow through context and cost on a single
cycle for no benefit — the corpus table is the useful artifact, not the raw response that produced
it.

**Time-box the cycle.** A full cycle (scan + attributes + hypothesis + patterns + log) should take
roughly 30–45 minutes of real work, similar to what a human doing this by hand would spend. If a
single channel is taking much longer than a few minutes to read (paywalled stats, broken page,
suspiciously slow), skip it and note the skip rather than burning the whole cycle on one channel.

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
   user can redirect you. Start it from `templates/CYCLE_TEMPLATE.md` in this skill so every cycle
   is logged in the same shape — don't improvise a new structure each time.
3. **Confirm the corpus.** Ask the user for their competitor/adjacent-channel list if one isn't
   already documented somewhere in the project, rather than inventing channel names. A wrong
   corpus produces confident-sounding nonsense.

   A corpus that actually works tends to split into three rings — ask the user to fill each,
   don't assume they'll volunteer all three unprompted:

   | Ring | Suggested size | What it is | What it teaches |
   |---|---|---|---|
   | **1 — Direct competitors** | ~6–10 | Same format, same promise, same audience | Patterns transfer almost 1:1 |
   | **2 — Adjacent** | ~4–8 | Different format, same underlying promise/niche | Teaches the *promise*, not the format |
   | **3 — Rising, rotating** | ~3–5, swapped every couple months | Channel under ~12 months old with uploads well above its own median | The most valuable and most perishable ring — it catches what's *starting* to work, not what already won |

   A working minimum is a handful of channels total (even 5–6 across the first two rings is
   usable); the full 15–20-channel spread is a target to grow into, not a requirement to start.
   **Finding Ring 3 without a paid tool:** open a Ring 1 outlier and look at the channels
   recommended alongside it. Small, unfamiliar channels that keep showing up there are already
   getting distribution next to a channel that matters — that's YouTube itself pointing at who's
   rising.

   **A corpus channel that visibly pivoted away from the niche is a finding, not a chore.** If a
   channel in the corpus has moved on — a travel channel that's now posting gear reviews, a finance
   channel now doing vlogs — don't just quietly swap it out. Log it (channel, what it pivoted to,
   roughly when) before replacing it. One pivot is noise; two or three pivoting toward the same
   adjacent format in the same cycle is a real signal that part of the niche is migrating, and
   that's worth surfacing on its own, separate from whatever title/thumbnail patterns this cycle
   finds.

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
it still applies, and only re-scan if the topic changed or the last cycle is stale. While there,
check whether any older cycle is sitting with a blank grade field on a video that's since had time
to accumulate real numbers — if so, do step 9 for that cycle before starting a new one.

### 2. State the target

One sentence: which video/topic this cycle serves, and what question it needs to answer (title?
thumbnail? both? is the underlying angle/thesis even settled yet?). Skipping this turns the scan
into unfocused data collection.

### 3. Scan the corpus

**Full mode vs. light/qualitative mode.** The full protocol below (median, ≥3× threshold,
independence rule) is the default. But run a **light mode** instead when view counts are hidden by
the creator, when there isn't time for a full cycle, or when the goal is a quick gut-check on one
topic rather than a periodic corpus sweep: skip the median math, just read recent titles/thumbnails
across the corpus and note what repeats. Log light-mode findings as **signal, not outliers** — one
line in the hypothesis making clear this cycle has no statistical basis, just a qualitative read.
Light mode is not a shortcut to skip logging or skip the hypothesis sentence — it's a real cycle,
just a weaker one, and it should say so plainly rather than borrow the confidence of a full cycle.

For each channel in the corpus: open the **"Latest" / "Most recent" tab, never "Most popular."**
"Most popular" mixes videos from different years and different algorithm regimes — it shows what
worked *once*, not what's working *now*, which is the only thing actionable.

- Window: last ~60–90 days (tighten for a fast-moving niche, widen for a slow-cadence channel).
- **Compute Shorts and long-form as two separate baselines, never mixed.** Shorts view counts
  inflate for reasons that have nothing to do with packaging quality (autoplay, feed behavior);
  folding them into the same median as long-form videos manufactures false outliers on one side
  and buries real ones on the other. If the target video is long-form, scan long-form uploads only.
- Compute the **median** (not mean) view count of the last ~10 uploads within that format. A
  single viral outlier distorts a mean and breaks the whole comparison.
- **Exclude uploads younger than ~7 days from the median baseline itself** — they haven't had time
  to reach their real view count yet, and including them drags the median down artificially,
  which then inflates the multiplier of everything else. It's fine to track a very recent upload
  separately and revisit it once it ages past that window.
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

Append the cycle to wherever cycles are being tracked in this project (see "First run" above),
using `templates/CYCLE_TEMPLATE.md`: objective, corpus used, outlier table, present/absent
attributes, the hypothesis sentence, the patterns (≤5), and the real limitations of that cycle's
sample. If the resulting editorial call (final title, a topic-order change) came from more than
just this cycle's data, say so — and if the host project has its own decisions log, record the
decision there too, not just the finding. Leave the cycle's **grade field blank** — it can't be
filled until step 9.

**Archive before the log gets expensive to read.** A biweekly cadence adds up — a year of cycles is
~25 entries, and reading the whole log back in step 1 gets slower and pricier every cycle. Once the
log passes roughly 1,500–2,000 lines (or a year of history, whichever comes first), move older
cycles into a dated archive file (e.g. `CYCLES_2026.md`) and leave a short index of what moved and
where in the live log. Never delete a past cycle, graded or not — an archived rejected pattern is
still worth being able to find later.

### 9. Grade the cycle once the video has real numbers

This is the step that turns cycles into an actual track record instead of a pile of guesses. Once
the video this cycle informed has enough real data, come back to that cycle's log entry and fill
in the grade. **"Enough data" isn't a fixed calendar number** — a channel that gets most of its
views in the first 48 hours and one that's a slow burn accumulating views over months shouldn't
share a flat 7–14 day rule. Check how the *channel's own* past videos typically accumulate views
(if that's known or easy to check) and grade once this video's curve has had a comparable amount of
time to develop; ~7–14 days is a reasonable fallback only when that history isn't available.

- **Confirmed** — the video's title/thumbnail performance (CTR, retention, or whatever the channel
  already tracks) is consistent with the hypothesis.
- **Rejected** — it isn't, and the hypothesis should be treated as weakened, not repeated blindly
  next cycle.
- **Inconclusive** — too little data, too many other variables changed at once, or the metric
  the channel cares about wasn't cleanly attributable to packaging.

Do this even when nobody asks — an ungraded cycle is a cycle nobody ever learned from.

### 10. Meta-review every 5–10 cycles

Grading step 9 one cycle at a time catches obvious repeats, but a lot of the real value of running
this for a year+ only shows up by scanning the *whole* log at once. Every 5–10 cycles (or whenever
it's been a couple months since the last meta-review), skim every graded cycle together and ask:

- Which pattern has been **confirmed across multiple different topics**, not just repeated within
  one series? That's a strong candidate to promote out of "hypothesis" and into a standing rule in
  the channel's own strategy docs — stronger than any single cycle's evidence on its own.
- Which pattern has been **rejected more than once**? Retire it from the attribute checklist in
  step 4 instead of re-testing something the channel's own data has already answered.
- Is one ring (direct/adjacent/rising) producing most of the confirmed patterns, and the others
  mostly noise? That's worth adjusting the corpus mix toward, not just noting.

Log the meta-review's conclusions the same place cycles live, clearly marked as a meta-review
entry rather than a numbered cycle, so it doesn't get graded like one.

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

**Survivorship bias is baked into the method and can't fully be fixed, only kept in mind.** The
corpus only contains channels that still exist and still publish — a channel that tried the same
angle and failed badly enough to quit or pivot is invisible to this scan (the pivot-tracking in
step 3 catches *some* of this, but only for channels that pivoted rather than vanished outright).
Every pattern found here is a pattern that worked for the channels currently standing, not
necessarily a pattern that works in general.
