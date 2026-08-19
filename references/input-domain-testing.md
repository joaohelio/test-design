# Input-Domain Testing: Partitions and Boundaries

The core idea of systematic input testing: don't pick test values by feel —
model the input domain first, then let the model dictate the values. The model
produces **coverage items**, and each test case exists to cover one or more of
them. This keeps suites small, complete, and explainable ("this case exists
because of that partition").

## Equivalence Partitioning

Split every data element (inputs, outputs, config values, internal state,
time-related values) into groups the code treats identically. Any single value
from a group stands in for the whole group — if one member finds a bug, the
rest would too, so one test per partition suffices.

Rules for a sound partition model:
- Partitions must not overlap, and none may be empty.
- Model the **rejected** values too, not just the accepted ones. A partition of
  values the code should process is a *valid* partition; a partition it should
  refuse or ignore is an *invalid* partition. Both count toward coverage.
- Full coverage = at least one test per partition, invalid partitions included.

When a function takes several parameters, each with its own partitions, testing
every combination explodes. The pragmatic criterion is **each-choice**: every
partition of every parameter appears in at least one test, and a single test
may tick off partitions of several parameters at once. Reserve combination
testing (see decision tables) for parameters whose *interaction* carries the
logic.

## Boundary Value Analysis

Off-by-one mistakes concentrate at partition edges — a comparison written with
`<` instead of `<=`, a limit checked against the wrong constant, a fencepost in
a loop. So for every **ordered** partition, test the edges, not just an
interior representative.

Two rigor levels:
- **Two-point**: for each boundary, test the boundary value and the nearest
  value on the other side of it.
- **Three-point**: additionally test the nearest value on the *inside*. This
  catches a comparison collapsed to equality: if `if amount < 1000` was
  miscoded as `if amount == 999`, the two-point pair (999, 1000) still passes,
  while the inside neighbor 998 exposes it.

Use three-point on money, limits, and anything with a defect history;
two-point is enough for low-risk ranges. Full coverage = every identified
boundary neighborhood exercised.

### Worked example (Go, table-driven)

`Fee(weightKg int)` — free up to 5 kg inclusive, charged 6–30 kg, rejected
above 30 or non-positive:

```go
tests := []struct {
    name    string
    kg      int
    want    Fee
    wantErr bool
}{
    {"zero weight rejected", 0, 0, true},
    {"minimum accepted weight", 1, Free, false},
    {"last free kg", 5, Free, false},   // boundary free/charged
    {"first charged kg", 6, Charged, false},
    {"last accepted kg", 30, Charged, false}, // boundary charged/rejected
    {"just over the limit", 31, 0, true},
}
```

Six cases, and every one is accounted for: three partitions (invalid-low,
free, charged — invalid-high covered by the last case) and both boundary
neighborhoods. Naming each case after its coverage item makes the model
auditable without comments.

## Reporting coverage

When delivering tests built this way, say what the model was and what the suite
achieves — e.g., "4 partitions (2 invalid), two-point boundaries at 5|6 and
30|31, all covered." A reviewer can then check the *model* for gaps instead of
re-deriving it.

---
*These techniques long predate any certification scheme. Canonical treatments:
Myers, "The Art of Software Testing"; Copeland, "A Practitioner's Guide to
Software Test Design"; ISO/IEC/IEEE 29119-4; the ISTQB CTFL syllabus
(istqb.org) covers them at foundation level.*
