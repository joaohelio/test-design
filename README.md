# test-design

A [Claude Code](https://code.claude.com) skill that makes Claude derive tests
systematically instead of improvising them. Whenever tests are created,
extended, or reviewed, Claude models the input space first and lets the model
dictate the cases:

- **Equivalence partitioning + boundary value analysis** for ordered input
  ranges (two-point or three-point boundaries)
- **Decision tables** for combinations of business rules
- **State transition testing** for lifecycles and state machines
- **Each-choice coverage** for independent parameter sets
- **Branch coverage** checks and an **error-guessing** sweep as complements

Each delivered test names its coverage item, and the suite reports the coverage
achieved (partitions, boundaries, rule columns, transitions). The skill also
answers testing-terminology questions and helps with test plans, risk-based
prioritization, acceptance criteria/ATDD, and defect reports.

## Install

```sh
git clone https://github.com/joaohelio/test-design ~/.claude/skills/test-design
```

Restart Claude Code; the skill triggers automatically on test-writing tasks.

## Layout

- `SKILL.md` — the technique-selection workflow Claude follows
- `references/` — technique guides loaded on demand (input domains, logic &
  state, structural coverage & heuristics, specifications, process, terminology)

## License

Apache-2.0. The test-design techniques taught here are industry standards that
long predate any certification scheme; each reference file cites canonical
sources (Myers, Beizer, Copeland, ISO/IEC/IEEE 29119, among others).

This project is not affiliated with or endorsed by ISTQB®. ISTQB is a
registered trademark of the International Software Testing Qualifications
Board.
