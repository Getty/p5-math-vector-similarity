# Math::Vector::Similarity

Lightweight pure-Perl, zero-dependency functions for comparing numeric vectors
(`cosine_similarity`, `cosine_distance`, `euclidean_distance`, `dot_product`,
`normalize`) — a flat `Exporter` function library, not an OO class. Released to CPAN as
`Math-Vector-Similarity`, built with `Dist::Zilla` via `[@Author::GETTY]`.

The function set, export contract and the numeric/zero-vector invariants live in skill
`math-vector-similarity-core` — they are not repeated here.

## Delegation

Delegate behavior-relevant code to the right agent instead of touching it yourself — the
principle and the lane boundaries are in `.claude/rules/math-vector-similarity-rules.md`.

| Task | Agent |
|---|---|
| Implement / refactor / debug behavior-relevant code | `math-vector-similarity-worker` (default) |
| Pre-release audit | `math-vector-similarity-release-checker` |

The agents carry their skills via `briefing.skills` (see `.claude/agents/`); the main
agent delegates rather than loading them. Skill sources live under `.claude/skills/` —
`getty-perl-core` and `kanban-issues-karr-cli` are hardlinked from `~/dev/skills/perl/`
and `~/dev/karr/`, `getty-perl-release-author-getty` and `perl-release-dist-ini` from the
shared library; `math-vector-similarity-core` is owned by this repo.

Work is coordinated on the repo's `karr` board (`karr board`).

## Downstream

**Langertha** depends on `Math::Vector::Similarity`. A release here can leave that pin
stale — file that as a ticket on Langertha's board, never as an edit from here.
