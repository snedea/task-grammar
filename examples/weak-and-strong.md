# The same work, written twice

One feature: add CSV export to an expense-tracking web app's report
page. First as tasks are usually written, then with Task Grammar
applied. The deltas are keyed to the seven rules at the bottom.

## Weak

```
- [ ] Add CSV export like we discussed
- [ ] Wire up the export button, don't touch the auth code or the
      billing module
- [ ] Make sure everything works
- [ ] Update docs
```

Every line above executes. That is the trap: an agent will happily run
this list and report success. The defects surface later, at machine
speed.

## Strong

```
- [ ] T1.1: Add a CSV export endpoint for the expense report
  - Files: src/api/reports.ts, src/services/exportService.ts (new),
    tests/exportService.test.ts (new)
  - Opens: the report data shape is defined in src/types/report.ts
    (ReportRow, decided in SPEC.md section 4); no export code exists yet
  - Constraint: amounts are formatted from the stored integer cents,
    never from the floating-point display value (SPEC.md, Money rules)
  - Verification: npm test -- exportService exits 0, and the new test
    asserts a row with amount_cents=1050 exports as "10.50"

- [ ] T1.2: Add the export button to the report page
  - Files: src/pages/ReportPage.tsx, src/services/api.ts
  - Opens: T1.1 shipped GET /api/reports/:id/export returning text/csv;
    the API service module pattern is one function per endpoint
  - Verification: npm run build exits 0, and grep -c "reports/.*export"
    src/services/api.ts returns 1

- [ ] T1.3: Document the export feature
  - Files: README.md, docs/reports.md
  - Opens: T1.1 and T1.2 are merged; the endpoint and button exist as
    built, so document from the code, not from this list
  - Verification: grep -qi "csv export" docs/reports.md && grep -qi
    "csv" README.md, both exit 0
```

## The deltas, keyed to the rules

- **Rule 1 (cold start):** "like we discussed" became an `Opens:` bullet
  naming the file and spec section where the decision lives. The
  executing agent was not in the discussion.
- **Rule 2 (runnable check):** "make sure everything works" became one
  verification per task, each a command with an expected result. The
  cents-formatting test converts the money rule from a claim into a
  fact.
- **Rule 3 (allowlist):** "don't touch the auth code or the billing
  module" is gone entirely. The `Files:` allowlists steer the same
  boundary without ever naming auth or billing, so neither enters the
  agent's attention.
- **Rule 4 (restated constraint):** the integer-cents rule already lives
  in the spec, and T1.1 repeats it anyway, because T1.1 is where it
  must survive.
- **Rule 5 (word budget):** the detail concentrates at the one place an
  agent would guess (money formatting). T1.3 stays short and points at
  the code as its source of truth instead of restating it.
- **Rule 6 (artifact carries state):** T1.2 and T1.3 open from what
  earlier tasks SHIPPED, not from what anyone remembers. Every
  reference is to a file or a merged artifact.
- **Rule 7 (size to the engine):** three one-pass tasks for a mid-tier
  model. For a frontier model, T1.1 and T1.2 could merge into one pass;
  for a small local model, T1.1 likely splits (service, then endpoint).
  The list states no size constant; the shape moves with the engine.
