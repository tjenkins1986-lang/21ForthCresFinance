# 21 Forth Cres — Financial Tracking & Planning

**Purpose:** Run the monthly household finance review for Tom and Hannah,
and keep the live tracker up to date with the latest statement data.

**Live tracker:** https://tjenkins1986-lang.github.io/21ForthCresFinance/
(Google sign-in required — authorized to Tom's and Hannah's accounts only.)

**JSON format:** see `DATA-FORMAT.md` in this project's knowledge for the
exact schema Widget 09 expects, and the widget-by-widget checklist of what
to ask about each month. This document doesn't repeat that — if the two
ever seem to disagree, `DATA-FORMAT.md` is the one to trust for format
questions, since it's kept directly in sync with the app's code.

## Monthly cycle

Statements release on the 26th of each month. A recurring all-day calendar
reminder fires on the 27th — "21 Forth Cres — Monthly Finance Review."

Each cycle:

1. Start a new chat in this Project. Don't continue an old thread — start
   fresh so context stays clean.
2. Upload the latest bank statement (PDF, CSV, or screenshot — whatever's
   available).
3. Claude runs validity checks before anything else:
   - Reconciles the statement total against what's expected from prior
     months' patterns
   - Flags any transaction that looks duplicated, misdated, or
     inconsistent with the established categorisation rules (see "Data
     conventions" below)
   - Flags anything that breaks an existing pattern (e.g. a Regular
     payment landing on an unusual date, an unusually large or small
     category total)
4. Review together, widget by widget, per the checklist in
   `DATA-FORMAT.md` — Claude asks probing questions on anything unclear
   or judgement-based: new one-off costs, ambiguous transactions, whether
   something is a Top Up vs a Savings Drawdown vs Regular, pension
   values, savings goal changes. Don't guess silently; ask. Don't skip a
   widget just because the statement didn't mention it.
5. Agree what changes, across whichever widgets are affected this cycle
   (Net Worth, Where We Spend, What We Contribute, Savings Goals, Story
   of the Month, Rolling Actions — Next Month's Contribution is
   computed automatically, nothing to agree there).
6. Claude produces an updated JSON snapshot matching `DATA-FORMAT.md`'s
   schema — typically `netWorth`, `spend`, `contrib`, `story`, `actions`,
   and `meta`, since the rest change rarely. `spend`/`contrib` only need
   the new month (they merge in); everything else needs its complete
   current value if included.
7. On the live tracker: sign in with Google, paste the snapshot into
   Widget 09 ("Monthly Data Update"), click **Apply Locally** to preview
   it, check the numbers look right, then click **Save to Cloud** — this
   is what actually makes it live for both of you. Applying locally
   alone doesn't persist anything.
8. New action items surfaced during the review get appended to Rolling
   Actions (Widget 08) — they don't replace the existing list (use
   Widget 09's "Copy Current Data as JSON" to get the current list before
   adding to it). Resolved items get ticked off in the app directly —
   this now saves to the cloud immediately, no extra step needed.

## Data conventions (don't relitigate these each month — apply them)

- **Regular contribution dating:** a payment dated at the start of a
  month counts toward that month; one dated at the very end of a month
  counts toward the following month. Top Ups and Savings Drawdowns are
  never shifted — they stay on the date they happened.
- **Category groups (Widget 04):** Mortgage (own group, 50:50 split) ·
  Ongoing Monthly — Budget-able (5:4 split) · One-Offs · Needs Further
  Analysis. Every spend category must sit in exactly one group — the
  fixed list of category names is in `DATA-FORMAT.md`.
- **Contribution principles (Widget 01):** Monthly Budgetary Spend 5:4
  (Tom:Hannah) · Mortgage 50:50 · Savings 50:50.
- **Savings goals (Widget 06):** required monthly contribution =
  (target − current balance) ÷ months remaining, counting from next
  month through the goal's target month inclusive. The Monzo General
  Investment Account has no target — tracked for growth only, not
  included in the monthly requirement (confirm this is still true each
  cycle rather than assuming — goals can change).
- **Current balances in Widget 06** pull from Widget 02's asset snapshot
  by exact label match — check labels match if a goal ever shows £0
  unexpectedly.

## Widget reference

01 Household Principles & Contributions (contribution ratios, plus an
"Agreed Approaches to Money" free-text list) · 02 Net Worth Summary ·
03 Story of the Month · 04 Where We Spend · 05 What We Contribute ·
06 Savings Goals · 07 Next Month's Contribution (computed, nothing to
input) · 08 Rolling Actions · 09 Monthly Data Update.

This instructions file itself should be revised as the process evolves —
treat it the same way as the widget data: update it when something
changes, don't let it drift out of sync with what the tracker actually
does. If you change how Widget 09 works, update `DATA-FORMAT.md` first
(it's the schema source of truth), then update this file's references to
match rather than re-describing the format here.
