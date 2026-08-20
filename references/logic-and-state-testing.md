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
contradictions before any code runs.

### Combinatorial discipline

"Feasible" is where improvisation sneaks back in unless it's pinned down:
a combination is **infeasible only when it is logically or physically
impossible** (a guest user with an admin role, an amount both zero and
positive). Merely *unlikely* combinations stay in the table — or get cut by
an explicit risk argument, stated in the coverage report, never silently.

When the minimized table is still large, reduce in this order:

1. **Minimize first** — drop impossible columns, merge columns where a
   condition's value doesn't change the outcome (`–` entries).
2. **Past roughly 15–20 remaining columns, or 4–5 conditions, switch to
   pairwise**: every value pair of every condition pair appears in at least
   one test. Pairwise catches the two-way interactions where most
   combination bugs live, at a fraction of the column count.
3. **Keep full columns for risk-critical rules** — money movement, safety,
   compliance outcomes get their exact combination tested even under an
   otherwise pairwise suite.

Always report which discipline the suite uses: "full table, 9 columns" and
"pairwise over 6 conditions, plus 3 full risk columns" are different claims,
and a reviewer can only audit the one you actually make.

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

Valid and invalid are not the whole table. A third cell type exists wherever
triggers can be delivered more than once (retries, at-least-once messaging,
double-clicks): the **redundant** transition — a *valid* trigger arriving when
its effect is already applied. It is not an invalid move to be refused; the
required behavior is a defined no-op that repeats no side effect. Mark these
cells explicitly (a self-loop with "no action") rather than leaving them
blank, because a blank cell reads as "refuse" — the opposite of correct
handling. Deriving cases for redundant and reordered deliveries is its own
technique: [event-driven-testing.md](event-driven-testing.md).

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
`created + capture` and asserts the refusal, one per test. Where the intent's
triggers arrive over a retrying transport, add the redundant cells — e.g.
`authorized + authorize` again must no-op, not double-reserve.

## When One Object Matches Two Models

The same object is often *both* a state-transition subject and a
decision-table subject — a lifecycle plus rule flags gating some transitions.
Crossing the two models (every rule column in every state) multiplies suite
size, so the default is **additive**: cover each model independently, letting
one test tick items in both.

Cross state × rule only where a rule's outcome **plausibly depends on the
state the transition starts from** — and pick those cells by risk, not
exhaustively. A discount rule that reads only the cart is state-independent;
a cancellation-fee rule that differs before and after driver assignment is
exactly the state-dependent kind worth crossing, because the additive suite
would test the rule in whichever state happened to be convenient and miss the
other. Either way, the coverage report must say which was done: "state model
and rule table covered independently" and "fee rule crossed with the 3 states
it can fire from" are different claims.

---
*Decision tables and state-based testing are classic techniques: Beizer,
"Software Testing Techniques"; Binder, "Testing Object-Oriented Systems";
ISO/IEC/IEEE 29119-4; also covered by the ISTQB CTFL syllabus (istqb.org).*
