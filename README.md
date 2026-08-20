# test-design

A [Claude Code](https://code.claude.com) skill that makes Claude derive tests
systematically instead of improvising them. Whenever tests are created,
extended, or reviewed, Claude models the input space first and lets the model
dictate the cases:

- **Equivalence partitioning + boundary value analysis** for ordered input
  ranges (two-point or three-point boundaries)
- **Decision tables** for combinations of business rules, falling back to
  **pairwise** when the combinations explode
- **State transition testing** for lifecycles and state machines
- **Event-sequence & idempotency testing** for queue handlers and webhooks
  under at-least-once delivery — replays, out-of-order pairs, redelivery after
  a mid-handler failure
- **Each-choice coverage** for independent parameter sets
- **Branch coverage** checks and an **error-guessing** sweep as complements

Artifacts that aren't logic get their own disciplines instead of the techniques
above: data migrations and backfills, infrastructure-as-code, and rendered
output (golden-file/snapshot testing).

Each delivered test names its coverage item and asserts the behavior that item
names, and the suite reports the coverage achieved (partitions, boundaries,
rule columns, transitions). Rigor scales with risk — a trivial function gets a
case, not a model. The skill also answers testing-terminology questions and
helps with test plans, risk-based prioritization, acceptance criteria/ATDD, and
defect reports.

## Install

```sh
git clone https://github.com/joaohelio/test-design ~/.claude/skills/test-design
```

Restart Claude Code; the skill triggers automatically on test-writing tasks.

## Layout

- `SKILL.md` — the technique-selection workflow Claude follows
- `references/` — technique guides loaded on demand (input domains, logic &
  state, event-driven, migrations/infra/rendered output, structural coverage,
  assertion quality & heuristics, specifications, process, terminology)

## License

Apache-2.0. The test-design techniques taught here are industry standards that
long predate any certification scheme; each reference file cites canonical
sources (Myers, Beizer, Copeland, ISO/IEC/IEEE 29119, among others).

This project is not affiliated with or endorsed by ISTQB®. ISTQB is a
registered trademark of the International Software Testing Qualifications
Board.
