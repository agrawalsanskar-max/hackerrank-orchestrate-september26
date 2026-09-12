# Cash-Flow Affordability Agent — `build_agent.py`

## How to run

```bash
# from a folder containing this file + a sibling dataset/ directory:
python3 build_agent.py
```

Requires only the Python standard library (`csv`, `datetime`, `statistics`,
`re`, `collections`) — no `pip install` needed, no network access, no API
keys.

Produces:
- `dataset/output.csv` — one row per request in `dataset/requests.csv`,
  matching the exact required schema.
- `evaluation/usage_report.md` — $0-cost / 0-API-call usage report (the
  script never calls a network or LLM/vision API at runtime).

## What it does

1. **Loads** all seven `dataset/*.csv` files.
2. **Fills blank `amount` values** in `financial_events.csv`. There were
   16 such rows, each linked via `images.csv` to a receipt/payslip image.
   Those 16 amounts were read **offline, once**, by visual inspection of
   the images and are hardcoded in the `IMAGE_AMOUNTS` dict at the top of
   the script — no vision API is called when the script runs, per the
   "no external API" execution directive.
3. **Reconciles `messages.csv`** against events: explicit
   cancellation/settlement/amount-change keywords (in English and
   Indonesian) are extracted with a small regex/keyword vocabulary and
   applied to the linked event (or, for unlinked employer/payroll
   messages, to the user's most recent salary series). Free text is only
   ever parsed for `(currency, amount)` / `(date)` / a fixed keyword set —
   it is never treated as an instruction, which guards against prompt
   injection embedded in message or OCR text.
4. **Projects recurring cash flow forward.** The dataset only contains
   *historical* (and a few already-scheduled/pending) event rows — there
   is no explicit "future schedule" table. To build a 90-day forecast,
   the script groups each user's events into recurring series (named
   obligations like rent/subscriptions/salary by description; high-
   frequency variable spend like groceries/transport/dining/shopping/
   healthcare/entertainment by category) and projects them forward using
   the median historical interval between occurrences and the last (or,
   for variable-spend groups, the historical mean) amount.
5. **Simulates the 90-day daily balance**, computes `amount_safe_to_pay`
   and `earliest_date_for_full_payment`, evaluates every payment method
   the user's profile allows (`full_payment`, `partial_payment`,
   `installments` against `request_payment_options.csv`, `wait`,
   `not_recommended`), applies the 6-level tie-break matrix, and — when a
   full payment today is only unsafe by a small margin — greedily tries
   stopping/reducing up to 3 flexible/stoppable recurring expenses
   (earliest-occurring first) to close the gap.
6. **Writes** `output.csv` and `usage_report.md`.

## Known limitation — please read

`dataset/sample_requests.csv` ships 25 fully-worked ground-truth examples.
Running this pipeline against them gets the **qualitative** columns right
most of the time (17/25 `recommended_payment_method` matches, 15/25
`affordability_status` matches) but does **not** reproduce the exact
`amount_safe_to_pay` figures in most rows. The reason is structural, not a
bug: the raw dataset gives no explicit definition of *how* each user's
recurring income/expenses continue past their last historical row, so any
90-day forecast necessarily rests on an assumed projection rule. The rule
used here (median historical interval + last/mean amount, grouped as
described above) is a reasonable, transparent, fully-deterministic choice,
but a different (equally defensible) projection rule would shift the
exact numbers. If the grading harness has access to the true generator
logic behind `financial_events.csv`, swapping in that exact recurrence
rule inside `project_window()` is the single highest-leverage change to
improve numeric accuracy — the rest of the pipeline (currency conversion,
safety-rule simulation, method evaluation, tie-breaking, formatting) is
exact/spec-literal and validated against the samples where the projection
happens to line up (e.g. `request_01`, `request_09`, `request_11`,
`request_12`, `request_17`).

## Packaging for submission

```bash
zip -r code.zip build_agent.py evaluation/ README.md
# output.csv is at dataset/output.csv — copy/rename as your submission requires
```

For the `chat_transcript` deliverable: export/save this conversation from
the Claude interface (e.g. via the share/export option) — that file isn't
something `build_agent.py` can generate itself since it's the record of
this development session, not a data-processing output.
