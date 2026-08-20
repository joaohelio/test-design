---
name: test-design
description: Apply industry-standard, ISTQB-aligned test-design techniques whenever creating, extending, or reviewing tests (unit/integration/e2e, any language) — equivalence partitioning, boundary value analysis, decision tables, state transition testing, event-sequence/idempotency testing (duplicate, out-of-order, or redelivered messages; event consumers; webhooks), branch coverage, error guessing. Also use for testing data migrations/backfills, infrastructure-as-code, and rendered output (golden-file/snapshot testing), and for test plans, test strategy, test case prioritization, defect reports, risk-based testing, acceptance criteria/ATDD, and testing terminology questions (test levels, test types, coverage items, testing principles, ISTQB concepts).
---

# Systematic Test Design

Whenever tests are being written, the cases must be **derived from a model**,
not improvised: identify the test basis, pick techniques by the shape of the
logic, derive test conditions → coverage items → test cases, and state the
coverage achieved.

**Scale the rigor to the risk.** A trivial pure function, a getter, or a fix to
one broken case gets the case that closes the gap — no partition model, no
coverage statement. The full workflow earns its cost where the logic branches,
or where money, state, external effects, or a defect history are involved. When
the model is skipped, skip it deliberately: never produce a partial one and
report it as complete coverage.

## Workflow for creating tests

1. **Identify the test basis** — the function/endpoint/story under test and its
   specified behavior (signature, docs, acceptance criteria, or the code's
   evident contract).

2. **Select techniques by the shape of the logic** — combine all that apply:

   | Logic shape | Technique | Full coverage means |
   |---|---|---|
   | Ordered input ranges / domains | Equivalence partitioning + boundary value analysis | every partition (valid **and** invalid) hit once; every boundary neighborhood (three-point for critical logic, two-point otherwise) |
   | Combinations of conditions / business rules | Decision table | every feasible rule column of the minimized table exercised (infeasible = impossible, not unlikely; past ~15–20 columns or 4–5 conditions, fall back to pairwise + full columns for risk-critical rules) |
   | Stateful / lifecycle behavior (status fields, state machines, sagas) | State transition | all valid transitions minimum; invalid transitions attempted **one per test case**; **redundant** transitions (a valid trigger redelivered) shown to no-op, not reject |
   | Event/message consumers under at-least-once delivery (queue handlers, webhooks, multi-consumer propagation) | Event-sequence & idempotency | every event replayed in each state where it applies (no-op asserted, **including no repeated side effect**); every ordering-dependent event pair delivered out of order; redelivery after simulated mid-handler failure |
   | Multiple independent parameter sets | Each-choice coverage, escalating to pairwise | each partition of each parameter hit at least once (combinations not required) — but each-choice is blind to interactions: once the parameters affect each other's outcome, or past ~4–5 of them, switch to pairwise ([logic-and-state-testing.md](references/logic-and-state-testing.md)) |
   | Everything else / final sweep | Error guessing over the fault categories: input, output, logic, computation, interfaces, data & state, third-party integrations | — |

   When one object matches two rows (stateful **and** rule-gated is the common
   pair), cover each model independently by default; cross state × rule only
   for rules whose outcome plausibly depends on the originating state, chosen
   by risk — and say which was done in the coverage statement.

   Details and worked examples:
   [references/input-domain-testing.md](references/input-domain-testing.md),
   [references/logic-and-state-testing.md](references/logic-and-state-testing.md),
   [references/event-driven-testing.md](references/event-driven-testing.md),
   [references/coverage-and-heuristics.md](references/coverage-and-heuristics.md).

3. **Derive systematically**: test conditions → coverage items → test cases.
   Each case must **assert the behavior its coverage item names** — a case that
   would still pass with that behavior removed is decorative, and a model of
   perfect partitions asserting only `err == nil` catches nothing
   ([assertion quality](references/coverage-and-heuristics.md#assertion-quality)).
   When presenting the tests, briefly state which coverage items they cover and
   the coverage level reached (e.g., "all 4 partitions, three-point boundaries
   at both edges, all 6 valid transitions").

4. **Complement with structural coverage**: once specification-based cases
   exist, check branch coverage of the test object (branch subsumes statement
   coverage); each uncovered decision outcome is a candidate test case or dead
   code worth questioning. Remember coverage can't see defects of omission.

5. **Map to repo conventions** — techniques decide *which* cases exist, never
   the style. Use the repo's parameterized idiom — Go table-driven,
   `pytest.mark.parametrize`, `test.each` — with one entry per partition /
   boundary value / decision rule / transition sequence, named after its
   coverage item. Follow the surrounding test file's idioms; never impose
   test-documentation formats onto code.

6. **Comments**: only where the case name can't carry the rationale — why this
   value (which boundary/partition/rule), never what the test does.

Reviewing existing tests uses the same table in reverse: map cases to coverage
items and report the gaps (missed partitions, untested boundaries, uncovered
decision-table columns, unexercised transitions, unreplayed/unreordered event
deliveries).

## Routing for other tasks

| Task | Reference |
|---|---|
| Data migrations/backfills (parity, re-run idempotency, cutover, rollback), infrastructure-as-code (plan diffs, policy checks, restore drills), rendered output — PDFs, documents, layout (golden-file/snapshot testing) | [references/migration-infra-and-output-testing.md](references/migration-infra-and-output-testing.md) — these artifact classes have their own disciplines; don't force the logic techniques onto them |
| Test plan content, entry/exit criteria, estimation (ratios, three-point, estimation poker), prioritization, test pyramid, quadrants, risk-based testing, metrics/reports, **defect reports** | [references/test-process-management.md](references/test-process-management.md) |
| User stories (3 C's, INVEST), acceptance criteria formats, ATDD/TDD/BDD | [references/specification-collaboration.md](references/specification-collaboration.md) |
| Terminology and concepts: testing principles, test levels/types, confirmation vs regression, static testing & review types, tools/automation, glossary | [references/terminology.md](references/terminology.md) |

When answering terminology questions, use the standard industry definition and
name the concept precisely (e.g., "all-valid-transitions coverage, also called
0-switch coverage").
