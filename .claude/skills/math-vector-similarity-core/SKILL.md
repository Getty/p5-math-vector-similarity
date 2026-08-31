---
name: math-vector-similarity-core
description: "Load before editing Math::Vector::Similarity — the pure-Perl, zero-dependency vector comparison module: it is a flat function library exported via Exporter (not OO, no Moo), its five functions, the dimension-mismatch and zero-vector contracts, and the numeric tolerances its tests pin."
---

# Math::Vector::Similarity — core

One module, `lib/Math/Vector/Similarity.pm` (~164 lines). Pure-Perl functions that
compare numeric vectors given as plain `ArrayRef`s of any dimensionality. **Zero runtime
dependencies** — only `Carp` and `Exporter`, both core. The intended use is comparing
embedding vectors from LLM APIs, but nothing in the code assumes that.

There is no object here. Do not introduce `Moo`/`Moose`, a class, or `$self` — this is a
flat function library and callers depend on the functional call form.

## Export contract

```perl
use Exporter 'import';
our @EXPORT_OK  = qw( cosine_similarity cosine_distance euclidean_distance
                      dot_product normalize );
our %EXPORT_TAGS = ( all => \@EXPORT_OK );
```

- **Nothing is exported by default.** Callers name functions explicitly or use `:all`.
- A new public function is added to **both** `@EXPORT_OK` and, transitively, `:all`
  (which is just a ref to `@EXPORT_OK`, so adding to the array is enough).
- POD for each function is an `=func NAME` block placed immediately **after** the `sub`,
  with a one-line usage example — match the existing layout, don't move to a POD section.

## The five functions

| Function | Args | Returns |
|---|---|---|
| `dot_product($a, $b)` | two vecs | scalar inner product |
| `normalize($vec)` | one vec | **new** L2-unit `ArrayRef` |
| `cosine_similarity($a, $b)` | two vecs | scalar in `[-1, 1]` |
| `cosine_distance($a, $b)` | two vecs | `1 - cosine_similarity`, in `[0, 2]` |
| `euclidean_distance($a, $b)` | two vecs | scalar L2 distance |

Inputs are never mutated. `normalize` returns a fresh array (except the zero case below);
the two-arg functions return scalars. `cosine_distance` is defined purely in terms of
`cosine_similarity` — it does no arithmetic of its own and inherits that function's
guards, so fix numeric behavior in one place.

## Contracts the tests pin — do not regress these

1. **Dimension mismatch croaks.** Every two-arg function that iterates by index
   (`dot_product`, `cosine_similarity`, `euclidean_distance`) does
   `croak "vectors must have same dimensions" unless @$a == @$b;`. The message is matched
   by tests as `qr/same dimensions/` — keep that substring. `cosine_distance` inherits the
   check through `cosine_similarity`; `normalize` is single-arg and has no such check.

2. **Zero-vector guards, exact.** These are deliberate, not edge-case sloppiness:
   - `normalize` on a zero-magnitude vector returns the **original ref unchanged** (the
     test asserts identity `is $zero, [0,0,0]`), never a division by zero.
   - `cosine_similarity` returns integer `0` when either magnitude is zero
     (`return 0 if $denom == 0`) — the test asserts an exact `0`, so do not turn this
     into `0.0`-with-epsilon or let it produce `NaN`.

3. **Exact integer results.** `dot_product` on integer inputs is asserted with exact
   equality (`is dot_product([2,3],[4,5]), 23`), no epsilon. The accumulation must stay
   integer-exact for integer inputs — plain `+=` of products does this; don't reroute
   through a float-normalizing path.

4. **Float tolerance is `1e-9`.** Every non-exact numeric assertion uses
   `abs($got - $expected) < 1e-9`. That is the house tolerance for this module: results
   are computed in native double precision (`sqrt`, division), and a change that needs a
   looser epsilon to pass is a change that lost precision — investigate rather than widen
   the bound.

5. **Scale invariance & range.** `cosine_similarity` is scale-invariant (`[2,4,6]` vs
   `[1,2,3]` → `1`) and stays within `[-1, 1]` even at embedding dimensionality (a
   768-dim case is in the suite). Preserve both.

## Tests

`t/00_load.t` (loads the module) and `t/10_functions.t` (all behavior, `Test2::V0`).
Run with `dzil test` or `prove -lr t/`. `Test2::V0` is the `on test` dep in `cpanfile`;
it is the only non-core thing the suite needs.
