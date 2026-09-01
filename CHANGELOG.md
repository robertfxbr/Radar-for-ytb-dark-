# Changelog

## 1.1.0 — 2026-09-01

### Added

- `Requirements` section in `SKILL.md`/`README.md`: what tool access the skill needs, and an
  explicit instruction to say so and stop rather than fabricate data when that access is missing.
- Guidance to scan logged out, and to treat public-page scraping as "look, don't scrape at scale"
  given YouTube's Terms of Service.
- Guardrails against blowing context/cost: keep raw page data out of the conversation, and a
  ~30–45 minute time-box per cycle.
- The 3-ring corpus structure (direct competitors / adjacent / rising-rotating), including the
  "check an outlier's suggested videos" method for finding Ring 3 without a paid tool.
- `templates/CYCLE_TEMPLATE.md` — a shipped log template so every cycle is recorded in the same
  shape.
- Step 9, "Grade the cycle once the video has real numbers" — closes the loop between a cycle's
  hypothesis and the video's actual performance (confirmed/rejected/inconclusive), and step 1 now
  checks for any cycle still waiting to be graded.
- Median-calculation fixes: Shorts and long-form tracked as separate baselines, and uploads younger
  than ~7 days excluded from the median itself.

## 1.0.0 — 2026-09-01

Initial public release: generalized, channel-adaptive version of the Radar de Outliers process
(median-multiple outlier detection, present/absent attribute extraction, ≤5 actionable patterns,
title-scoring and visual-reference flows).
