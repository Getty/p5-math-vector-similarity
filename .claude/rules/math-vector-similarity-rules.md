# Math::Vector::Similarity House Rules

The shared house rules come from skilletor — bundle `getty-cpan-dist@getty`: engineering
discipline, the delegation lock, the publish gate, karr coordination, CPAN release. They load
as `.claude/rules/.local.*.md`; if they are missing, run `skilletor sync` and publish nothing
until they load. This file adds what is true only here, and wins where it differs.

## What this repo is

Cosine similarity, euclidean distance and other vector comparison functions — a tiny
zero-dependency function library whose whole value is exact, documented numeric behavior.

## Agent team

| Task | Agent |
|---|---|
| Implement / refactor / debug behavior-relevant code | `math-vector-similarity-worker` (default) |
| Commits, `Changes`, card → done, pre-release audit | `math-vector-similarity-release-manager` |
| Read-only research, audit, gap analysis | `math-vector-similarity-worker` ("read-only, change nothing") |

Behavior-relevant = anything in `lib/`, the tests, the exported function set, the
dimension/zero-vector contracts and numeric tolerance. `README.md` and `Changes` wording are not.

## Release

This distribution is an **upstream**: **Langertha** depends on it. A release here can leave that
pin stale — that is a ticket on Langertha's board, never an edit made from here.

## Hazards — the contracts are the product, don't regress them silently

- **The pinned contracts:** the dimension-mismatch croak, the exact zero-vector guards, the
  integer-exact `dot_product` and the `1e-9` float tolerance are pinned by `t/10_functions.t` —
  a change that needs a looser epsilon to pass lost precision: treat that as a finding, not a
  fix. Full contract table: skill `math-vector-similarity-core`.
- **Zero dependencies.** A new `requires` needs a real justification.
- **Layout:** `=func` POD goes after the `sub`, matching the existing code.

## Perl specifics — reference, don't restate

Module loading, cpanfile pinning for Getty-authored dependencies, POD conventions and house
style live in skills `getty-perl-core` and `getty-perl-release-author-getty` (force-loaded per
lane via `briefing.skills`); the function set, export contract and numeric invariants live in
skill `math-vector-similarity-core`.
