---
name: test-design
description: Apply industry-standard, ISTQB-aligned test-design techniques whenever creating, extending, or reviewing tests (unit/integration/e2e, any language) — equivalence partitioning, boundary value analysis, decision tables, state transition testing, branch coverage, error guessing. Also use for test plans, test strategy, test case prioritization, defect reports, risk-based testing, acceptance criteria/ATDD, and testing terminology questions (test levels, test types, coverage items, testing principles, ISTQB concepts).
---

# Systematic Test Design

Whenever tests are being written, the cases must be **derived from a model**,
not improvised: identify the test basis, pick techniques by the shape of the
logic, derive test conditions → coverage items → test cases, and state the
coverage achieved.

## Workflow for creating tests

1. **Identify the test basis** — the function/endpoint/story under test and its
   specified behavior (signature, docs, acceptance criteria, or the code's
   evident contract).

2. **Select techniques by the shape of the logic** — combine all that apply:

   | Logic shape | Technique | Full coverage means |
   |---|---|---|
   | Ordered input ranges / domains | Equivalence partitioning + boundary value analysis | every partition (valid **and** invalid) hit once; every boundary neighborhood (three-point for critical logic, two-point otherwise) |
   | Combinations of conditions / business rules | Decision table | every feasible rule column exercised |
   | Stateful / lifecycle behavior (status fields, state machines, sagas) | State transition | all valid transitions minimum; invalid transitions attempted **one per test case** |
   | Multiple independent parameter sets | Each-choice coverage | each partition of each parameter hit at least once (combinations not required) |
   | Everything else / final sweep | Error guessing over the fault categories: input, output, logic, computation, interfaces, data & state | — |

   Details and worked examples:
   [references/input-domain-testing.md](references/input-domain-testing.md),
   [references/logic-and-state-testing.md](references/logic-and-state-testing.md),
   [references/coverage-and-heuristics.md](references/coverage-and-heuristics.md).

3. **Derive systematically**: test conditions → coverage items → test cases.
   When presenting the tests, briefly state which coverage items they cover and
   the coverage level reached (e.g., "all 4 partitions, three-point boundaries
   at both edges, all 6 valid transitions").

4. **Complement with structural coverage**: once specification-based cases
   exist, check branch coverage of the test object (branch subsumes statement
   coverage); each uncovered decision outcome is a candidate test case or dead
   code worth questioning. Remember coverage can't see defects of omission.

5. **Map to repo conventions** — techniques decide *which* cases exist, never
   the style. In Go: one table-driven entry per partition / boundary value /
   decision rule / transition sequence, named after its coverage item. Follow
   the surrounding test file's idioms; never impose test-documentation formats
   onto code.

6. **Comments**: only where the case name can't carry the rationale — why this
   value (which boundary/partition/rule), never what the test does.

Reviewing existing tests uses the same table in reverse: map cases to coverage
items and report the gaps (missed partitions, untested boundaries, uncovered
decision-table columns, unexercised transitions).

## Routing for other tasks

| Task | Reference |
|---|---|
| Test plan content, entry/exit criteria, estimation (ratios, three-point, estimation poker), prioritization, test pyramid, quadrants, risk-based testing, metrics/reports, **defect reports** | [references/test-process-management.md](references/test-process-management.md) |
| User stories (3 C's, INVEST), acceptance criteria formats, ATDD/TDD/BDD | [references/specification-collaboration.md](references/specification-collaboration.md) |
| Terminology and concepts: testing principles, test levels/types, confirmation vs regression, static testing & review types, tools/automation, glossary | [references/terminology.md](references/terminology.md) |

When answering terminology questions, use the standard industry definition and
name the concept precisely (e.g., "all-valid-transitions coverage, also called
0-switch coverage").
