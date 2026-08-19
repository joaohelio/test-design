# Logic and State Testing: Decision Tables and State Transitions

## Decision Table Testing

When an outcome depends on a **combination** of conditions — pricing rules,
eligibility checks, feature-flag interactions — enumerate the combinations
explicitly instead of testing conditions one at a time. One-at-a-time testing
silently assumes the conditions are independent; the bugs live exactly where
they aren't.

Building the table:
- One row per condition, one row per resulting action; one column per **rule**
  (a distinct combination of condition values and the actions it triggers).
- Entries: `T`/`F` for boolean conditions (or concrete values/partitions for
  multi-valued ones), `–` when a condition doesn't matter for that rule, `N/A`
  when a combination can't occur.
- Start from all 2^n combinations mentally, then shrink: drop impossible
  columns, merge columns where a condition's value doesn't change the outcome.
- Full coverage = one test per remaining feasible column.

The payoff isn't only the tests — drawing the table routinely exposes
requirement gaps ("what *should* happen for paid + expired?") and
contradictions before any code runs. If the table is still huge after
minimizing, cut it by risk, and say so rather than pretending the suite is
exhaustive.

### Worked example (Go)

Charge routing: instant when the account is verified and the amount is under
the review threshold; queued for review when verified but at/over threshold;
always blocked when unverified.

```go
tests := []struct {
    name           string
    verified       bool
    underThreshold bool
    want           Route
}{
    {"verified small amount", true, true, Instant},   // T,T
    {"verified large amount", true, false, Review},   // T,F
    {"unverified", false, false, Blocked},            // F,– (amount irrelevant)
}
```

Three feasible rules, three tests — the `F,T` column merged into `F,–` because
the threshold can't rescue an unverified account. That merge is itself a
requirement worth confirming.

## State Transition Testing

For anything with a lifecycle — an order status, a connection, a saga, a
document workflow — the bug surface is the *transition structure*, not
individual states. Model it before testing it:

- A **state diagram** lists states and the valid transitions between them, each
  labeled `event [guard] / action`.
- The equivalent **state table** (states × events) is often more useful for
  testing because its empty cells make the **invalid** transitions visible —
  the "can't happen" moves users and retries will absolutely attempt.

A test case is a sequence of events driving a path through the model; one
sequence typically covers several transitions.

Coverage criteria, weakest to strongest:

1. **All states** — every state visited at least once. Too weak on its own: a
   single lucky path can visit everything while skipping most transitions.
2. **All valid transitions** — every edge of the diagram exercised. This is
   the sensible default; achieving it implies all-states.
3. **All transitions** — valid transitions exercised *plus* every invalid
   (empty-cell) move attempted and shown to be refused. Attempt only **one
   invalid move per test case**: if a test stacks two, the first rejection can
   mask a defect in the second. This level is the norm for safety- and
   money-critical state machines.

For a payment intent with states `created → authorized → captured` and
`created/authorized → canceled`, all-valid-transitions needs paths covering 4
edges; all-transitions additionally tries e.g. `captured + cancel` and
`created + capture` and asserts the refusal, one per test.

---
*Decision tables and state-based testing are classic techniques: Beizer,
"Software Testing Techniques"; Binder, "Testing Object-Oriented Systems";
ISO/IEC/IEEE 29119-4; also covered by the ISTQB CTFL syllabus (istqb.org).*
