# Math::Vector::Similarity House Rules

Apply to every task in this repository unless explicitly overridden. Bias: caution over
speed on non-trivial work; use judgment on trivial tasks. Loaded automatically at launch
(same priority as `CLAUDE.md`). Subagents get their conventions from the skills
force-loaded via `briefing.skills` — this file is for the orchestrating agent.

## Engineering discipline

1. **Think before coding** — State assumptions. When uncertain, ask rather than guess.
   Push back when a simpler approach exists. Stop when confused; name what's unclear.
2. **Simplicity first** — Minimum code that solves the problem. Nothing speculative. This
   is a zero-dependency function library; a new `requires` needs a real justification.
3. **Surgical changes** — Touch only what you must. Don't "improve" adjacent code,
   comments or formatting. Match the existing `=func`-after-the-`sub` layout.
4. **Tests verify intent, not just behavior** — Reproduce a bug before fixing it; leave a
   regression test behind. The numeric tolerances and zero-vector contracts encode why
   the results matter — a test that can't fail when they change is not a test.
5. **Match the codebase's conventions, even if you disagree** — Conformance > taste.
   Surface a harmful convention; don't fork silently.
6. **Fail loud** — "Done" is wrong if anything was skipped silently. "Tests pass" is
   wrong if any were skipped. Surface uncertainty, don't hide it.
7. **A red test is a claim before it is a failure** — Before changing code to turn a test
   green, say what it asserts and whether your fix keeps that claim or replaces it. If the
   claim is wrong, fix the claim and say so; don't quietly make the code match it.

## Delegation

Depends on whether the Agent/Task tool is available to you.

- **You can spawn subagents** (orchestrating main agent): Do NOT touch behavior-relevant
  code yourself — delegate. Your lane: coordinate, inspect, plan, review diffs, run tests,
  edit non-behavioral docs. Why: only the `math-vector-similarity-*` agents
  get their skills force-loaded via `briefing.skills`; you get no briefing and would touch
  the numeric internals with too little context.

  | Task | Agent |
  |---|---|
  | Implement / refactor / debug behavior-relevant code | `math-vector-similarity-worker` (default) |
  | Commits, `Changes`, card → done, pre-release audit | `math-vector-similarity-release-manager` |

- **You cannot spawn subagents** (you ARE a `math-vector-similarity-*` agent): The
  delegation lock does not apply — implement, refactor, debug and test per these rules.

Behavior-relevant = anything in `lib/`, the tests, the exported function set, the
dimension/zero-vector contracts, and numeric tolerance. `README.md` and `Changes` wording
are not.

**Only `math-vector-similarity-release-manager` commits.** A worker leaves a commit-ready tree and hands its card
to `review`; you then dispatch `math-vector-similarity-release-manager` to cut the commit and close the card.

## The contracts are the product — don't regress them silently

This is a tiny module whose whole value is exact, documented numeric behavior. The
dimension-mismatch croak (`qr/same dimensions/`), the exact zero-vector guards
(`normalize` returns its input ref; `cosine_similarity` returns integer `0`), the
integer-exact `dot_product`, and the `1e-9` float tolerance are all pinned by
`t/10_functions.t`. A change that needs a looser epsilon to pass lost precision — treat
that as a finding, not a fix. Full contract table: skill `math-vector-similarity-core`.

## Coordination — karr board (always in scope)

Ticket coordination is the orchestrating agent's job, so `karr` is always in scope —
don't invoke the skill first, just use it. Board state lives in `refs/karr/*`.

- `karr list --compact` / `karr board` — open work · `karr show ID` — detail
- `karr create "Title" --priority high --tags a,b --body '…'` — new ticket
- `karr move ID in-progress --claim NAME` — start · `karr handoff ID --claim NAME --note "…"` — to review

Serialize board mutations when fanning out: keep implementation parallel, then loop the
`karr move`/`handoff`/`sync` calls sequentially. Full command surface: skill
`kanban-issues-karr-coordination`.

## Release — never without permission

`dzil build` / `dzil test` / `prove -lr t/` are fine anytime. `dzil release` and any CPAN
upload are STRICTLY forbidden without the maintainer's explicit go-ahead — even if a plan
lists "release" as the next step. Stop and ask.

This distribution is an **upstream**: **Langertha** depends on it. A release here can
leave that pin stale — that is a ticket on Langertha's board, never an edit made from
here.

## Public issues — never act without instruction

`karr` is the internal agent board, churned freely. Any public tracker (GitHub/Codeberg)
carries real people's reports, written under the maintainer's account. Never act on a
public issue on your own initiative — not even to read it. No listing, viewing,
commenting, closing or creating unless the user explicitly says to handle a specific one.

## Perl specifics — reference, don't restate

Module loading, cpanfile pinning for Getty-authored dependencies, POD conventions and
house style live in skills `getty-perl-core` and `getty-perl-release-author-getty`
(force-loaded for `math-vector-similarity-*` agents). The function set, export contract
and numeric invariants live in skill `math-vector-similarity-core`. Do not duplicate them
here.
