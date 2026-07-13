# Inflation Nowcasting Replication Kit

Self-guided replication of **Knotek & Zaman (2014), "Nowcasting U.S. Headline and Core Inflation"** (Cleveland Fed WP 14-03), designed to work with any AI tutor model. Built 2026-07-11.

## Files

| File | Role |
|------|------|
| `ROADMAP.md` | The curriculum: milestones M0–M7, checkpoint questions, self-tests, your rules |
| `TUTOR_PROMPT.md` | Paste into every new AI session — contains the anti-hallucination guardrails |
| `paper_reference.md` | Condensed ground truth: all equations, data, results. Tutors must cite it |
| `paper_full_text.txt` | Full extracted paper text, for looking up details |
| `nowcast_skeleton.ipynb` | Where you work. Data download provided; model code is TODO stubs |
| `solutions/nowcast_solutions.ipynb` | Reference implementations. Opening rules are in ROADMAP.md |

## How to start each session

1. Open a new chat, give it access to this folder (or attach the three .md files + your notebook).
2. Paste the contents of `TUTOR_PROMPT.md` as your first message.
3. Tell it which milestone you're on.

## Environment

Python with `pandas`, `numpy`, `statsmodels`, `matplotlib`, `jupyter`. Internet required (data comes from FRED's no-key CSV endpoint — note this endpoint is live, not static: it returns the latest data on every fetch, so it works for real nowcasting too). Yi has a FRED API key: `fredapi` is optional for M0–M6 but required for M7's ALFRED vintage work (`get_series_as_of_date`). Everything was logic-tested on synthetic data on 2026-07-11; the self-tests that hit live FRED data run on your machine.

## If a self-test fails

First suspect your code, second suspect data quirks (FRED series revisions, sample-period edge cases), last suspect the test. If you conclude the test's threshold is wrong for the current data vintage, loosen it *knowingly* and note it in the notebook — don't let a tutor model silently rewrite tests to make them pass.

## Provenance note

FRED series codes and the M6 three-case calendar are kit simplifications, not from the paper (which uses ALFRED vintages, Haver, EIA/FT directly, and 6 monthly / 14 quarterly cases). Deviations are flagged in ROADMAP.md where they occur.
