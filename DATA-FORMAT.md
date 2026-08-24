# Monthly Snapshot — JSON Format Reference

This is the exact shape the app expects when you paste a snapshot into
**Widget 09 · Monthly Data Update**. Give this file to whatever process
generates your monthly JSON (e.g. a Claude conversation fed your bank
statement) so its output loads cleanly.

## How loading works

- Paste JSON into the Widget 09 box, click **Apply Locally** to preview
  in your browser only, then **Save to Cloud** to make it live for both
  of you.
- **Partial updates are fine.** The JSON only needs to contain the
  top-level keys you're actually changing. Any key you omit is left
  exactly as it was. So a typical monthly update usually only needs
  `netWorth`, `spend`, `contrib`, `story`, `actions`, and `meta` —
  `household`, `principles`, `savingsGoals` and `nextMonthConfig` rarely
  change month to month.
- Every number is a plain JSON number — no `£` signs, no commas
  (`1234.56`, not `"£1,234.56"`).
- Every month is `"YYYY-MM"` (e.g. `"2026-09"`). Every date is
  `"YYYY-MM-DD"` (e.g. `"2026-09-03"`).
- Months arrays must be in chronological order, oldest first. The **last**
  entry in `spend.months` is treated as "the current month" throughout
  the app (drives the 6-month rolling average window, the forecast
  target month, etc.) — so when a new month closes, append it to the end
  of `spend.months` (and `contrib.months`) rather than replacing the
  array.

## Top-level keys

```json
{
  "household": { ... },
  "principles": [ ... ],
  "netWorth": { ... },
  "moneyApproaches": [ ... ],
  "spend": { ... },
  "contrib": { ... },
  "savingsGoals": [ ... ],
  "nextMonthConfig": { ... },
  "story": { ... },
  "actions": [ ... ],
  "meta": { ... }
}
```

### `household`
```json
{ "contributorAName": "Tom", "contributorBName": "Hannah" }
```
Rarely changes. Used as display labels throughout the app.

### `principles`
```json
[
  { "label": "Monthly Budgetary Spend", "ratioA": 5, "ratioB": 4, "basis": "Based on relative days worked" },
  { "label": "Mortgage Contribution", "ratioA": 1, "ratioB": 1, "basis": "Split evenly" },
  { "label": "Savings Contribution", "ratioA": 1, "ratioB": 1, "basis": "Split evenly" }
]
```
**The three `label` values above must be spelled exactly like this** —
Widget 07's forecast looks each one up by exact string match to decide
the split ratio for the Ongoing Monthly Budget, Mortgage, and Savings
Target lines. If a label doesn't match, that line silently falls back to
a 1:1 split instead of erroring, so a typo here is easy to miss — double
check if next month's forecast splits look off.

### `netWorth`
```json
{
  "asOf": "2026-09-24",
  "assets": [
    { "label": "Santander 1/2/3 Account", "amount": 120.50 }
  ],
  "property": { "marketValue": 455000, "mortgageBalance": 195800 },
  "pensions": {
    "personA": { "name": "You", "pots": [ { "label": "Pension", "amount": 316000 } ] },
    "personB": { "name": "Hannah", "pots": [ { "label": "Pension", "amount": 0 } ] }
  }
}
```
Asset `label`s matter beyond display: **Widget 06's savings goals link to
an asset by matching its `label` exactly** (see `savingsGoals` below), so
keep account labels stable month to month rather than renaming them.

### `moneyApproaches`
Normally managed entirely through the app's UI (Widget 01's "Add"
button), not through the monthly JSON — but can be included if you want
to seed or bulk-edit it:
```json
[ { "text": "We'll each keep back £50/month for personal spending before the joint split", "addedDate": "2026-09-01" } ]
```
Including this key **replaces the whole list**, so only include it if
you mean to overwrite what's currently there.

### `spend`
```json
{
  "months": ["2026-01", "2026-02", "...", "2026-09"],
  "categories": [
    "Mortgage/Loan", "Car Finance", "Council Tax", "Utilities",
    "TV/Subscriptions", "Insurance", "Groceries", "Fuel/Transport",
    "Dining Out/Takeaway", "Entertainment & Family Activities",
    "Kids' Activities", "Household Maintenance & Contractors",
    "Retail/Shopping", "Travel/Trips", "Cash Withdrawals", "Bank Fees",
    "Personal Transfer – Discretionary"
  ],
  "monthly": {
    "Groceries": { "2026-01": 725.44, "2026-02": 945.99 }
  },
  "transactions": {
    "Groceries": {
      "2026-01": [
        { "date": "2026-01-06", "desc": "CARD PAYMENT TO TESCO STORES", "amount": 12.89 }
      ]
    }
  }
}
```
**The 17 category names above are fixed** — they're hardcoded into the
app's `SPEND_GROUPS` (the Mortgage / Ongoing Monthly / One-Offs / Needs
Further Analysis grouping in Widget 04). A category name that doesn't
match one of these exactly won't crash anything, but it also **won't
appear anywhere in Widget 04 or feed into Widget 07's budget forecast** —
it'll just be silently invisible. If you introduce a genuinely new
category, it needs to be added to `SPEND_GROUPS` in `index.html` (a code
change, not a data change) — ask for that separately rather than
inventing a new category name in the JSON.

For each category, `monthly[category][month]` should equal the sum of
`transactions[category][month][].amount` for that same month — the
`monthly` figure is what every calculation actually uses; `transactions`
is only used to populate the click-to-drill-down modal. If they don't
match, the totals shown won't match what the modal shows when you click
into them.

### `contrib`
```json
{
  "months": ["2026-01", "...", "2026-09"],
  "rows": {
    "Tom — Regular": {
      "monthly": { "2026-09": 3198 },
      "transactions": { "2026-09": [ { "date": "2026-08-31", "amount": 3198, "desc": "..." } ] }
    },
    "Hannah — Regular": { "monthly": { "2026-09": 2480 }, "transactions": { "2026-09": [] } },
    "Top Ups — Tom": { "monthly": { "2026-09": 0 }, "transactions": { "2026-09": [] } },
    "Top Ups — Hannah": { "monthly": { "2026-09": 0 }, "transactions": { "2026-09": [] } },
    "Savings Drawdowns": { "monthly": { "2026-09": 0 }, "transactions": { "2026-09": [] } }
  },
  "excludeMonths": []
}
```
Unlike `spend.categories`, **row names here are not fixed** — Widget 05
just renders whatever keys exist in `rows`, in the order they appear. The
five shown above are just the convention used so far; keep using them
for consistency unless you deliberately want to restructure this widget.

### `savingsGoals`
```json
[
  {
    "label": "Everyday Saver 7427",
    "purpose": "Emergency slush fund",
    "targetAmount": 3800,
    "targetDate": "2027-02-27"
  },
  {
    "label": "Monzo General Investment Account",
    "purpose": "General investment — no fixed target, tracked for growth",
    "targetAmount": null,
    "targetDate": null
  }
]
```
`label` must exactly match an asset `label` in `netWorth.assets` — that's
how the current balance gets pulled in. Use `null`/`null` for a goal
that's tracked but has no target (shown as balance-only, excluded from
the required-monthly-savings total).

### `nextMonthConfig`
```json
{ "additionalItems": [] }
```
For one-off amounts that should appear in Widget 07's forecast alongside
the regular budget/mortgage/savings lines:
```json
{ "additionalItems": [ { "label": "School shoes — both kids", "amount": 90, "ratioA": 1, "ratioB": 1 } ] }
```
`ratioA`/`ratioB` default to `1`/`1` if omitted. Usually left empty most
months.

### `story`
```json
{
  "month": "2026-09",
  "narrative": [
    "First paragraph of the write-up.",
    "Second paragraph."
  ]
}
```
Each array entry becomes one `<p>` paragraph in Widget 03.

### `actions`
Including this key **replaces the whole list**, same as every other key
— it does not append. If you want to add new rolling actions without
losing old ones, copy the current list first (Widget 09 → **Copy Current
Data as JSON**) and append the new item(s) to it before pasting back.
```json
[
  { "text": "Check whether the emergency fund deadline should move.", "raisedMonth": "2026-09", "done": false },
  { "text": "Resolved item, kept for the record.", "raisedMonth": "2026-08", "done": true }
]
```

### `meta`
```json
{ "currentAsOfDay": 24 }
```
The day-of-month the *current* (last) month in `spend.months` is partial
through — drives the "to the 24th" wording in Widget 04's header. Set
this to whatever day you pulled the bank statement data up to; if the
month is fully closed, use the month's last day number.

## A safe monthly workflow

1. In the app, click **Copy Current Data as JSON** (Widget 09) to get
   the current full state — this is your starting point, so nothing gets
   silently dropped.
2. Hand that, plus your new month's bank statement, to whatever process
   builds the update (e.g. a Claude conversation) with instructions to
   return a JSON object containing only the keys that changed, following
   this document.
3. Paste the result into Widget 09, click **Apply Locally**, and eyeball
   the numbers before trusting them.
4. Click **Save to Cloud** once it looks right.
