# Event B - N-gram Containment in `cal_similarity_ratio`

## Pre-Event-B state (frozen)

- Week 9 decision: [patchtrack-class-responsibilities.md](patchtrack-class-responsibilities.md)
  - `PatchTrack` recognised as a God class; refactor **deferred** until a
    concrete change or defect points at a specific boundary.
- Git tag: `concrete-design-problem` (`f6b2860`).
- Test baseline: **241 passed, 1 skipped**.
- Published sample result ([results.md](../../../results.md)): PA 42 / NE 26 /
  PN 12, median adoption **48.52%**, Q3 61%.

## Evolution pressure

A follow-up requirement depends on an assumption that was not explicit in the
original M4 AI prompt ("fixing PA", commit `133065d`). The M4 rewrite of
`classifier.cal_similarity_ratio` matches a source n-gram if it shares **any
single token** with the added lines:

```python
if added_tokens.intersection(ngram.split())
```

This equals the paper's Jaccard containment only when `NGRAM_SIZE == 1`. The
assumption is invisible: `common.ngram_size` is a mutable global reassigned in
`main._process_patch_pair`, `classifier.process_patch` and inside
`SourceLoader._build_bloom`.

## Predictions (recorded before any code change or test run)

1. The pressure lands on `classifier.cal_similarity_ratio` and the mutable
   `common.ngram_size` global, **not** on the `PatchTrack` God class deferred
   in Week 9.
2. Classification results will **not** move. Production runs at n = 1, where
   the corrected metric is mathematically identical; PA/NE/PN and the median
   adoption rate should be unchanged.
3. A new n = 2 test fails before the fix and passes after it.
4. Blast radius is at most 3 files: `analyzer/classifier.py`, its call site in
   `analyzer/main.py`, and `tests/test_classifier.py`.
5. Reading `common.ngram_size` at call time is unreliable, so `n` must be
   passed explicitly rather than only swapping the formula.

## Options considered

| Option | Description | Main trade-off |
|---|---|---|
| A | No structural change; document the n = 1 limitation only | Zero regression risk, but the metric silently misreports if `NGRAM_SIZE` is ever changed |
| B (chosen) | Compare n-grams to n-grams, with `n` passed explicitly | Correct at any n; forces a decision on line boundaries for added n-grams |
| C | Delete the knob: hard-code n = 1 and remove `NGRAM_SIZE` | Simplest, but removes a published extension point and diverges from the upstream tool |
| D (rejected) | Decompose the `PatchTrack` God class | Large blast radius (API, filesystem, reporting) with no demonstrated benefit for this pressure |

---

*Everything below was written after the predictions above were recorded.*

## Method

- Input data: upstream `dataprep/patches.zip` (200 projects), extracted outside
  the repo. Our original local `data/` folder is gitignored and was not
  committed.
- Harness runs the same path as `notebooks/run_experiment.ipynb`:
  `build_pr_project_pairs()` -> `classify()` -> `create_dataframes()`, plus the
  notebook's `median_integration_rate`. Slices: `first` = `pairs[:80]` (Week 7),
  `last` = `pairs[-80:]` (Week 8).
- **Pre** = analyzer exported from `HEAD` (`concrete-design-problem`).
  **Post** = working tree after the change. `n = 2` runs set
  `constant.NGRAM_SIZE = 2` to break the hidden assumption on purpose.
- Every `Similarity (%)` value was compared row by row between pre and post.

## Change made (Option B)

`analyzer/classifier.py::cal_similarity_ratio` now builds n-grams from each
added line and intersects them with the source n-grams, instead of asking
whether a source n-gram shares any single token. `n` is **derived from each
source n-gram** rather than read from `common.ngram_size`, and 0-length
n-grams never match.

Six tests were added to `tests/test_classifier.py`: whole-n-gram match at
n = 2, containment at n = 2, no matching across added-line boundaries,
independence from the global, mixed n-gram lengths, and empty n-grams.

## Results

### Tests

| State | Result |
|---|---|
| Pre (baseline) | 241 passed, 1 skipped |
| New tests against pre code | 4 of 5 failed (e.g. `assert 50.0 == 0.0`) |
| After first fix attempt | 246 passed, 1 skipped |
| Empty-n-gram test against first fix | 1 failed |
| Post (final) | **247 passed, 1 skipped** |

### Option A: no change (pre code), measured

| Run | PA / NE / PN | PA mean similarity | PA median similarity |
|---|---|---:|---:|
| last 80, n = 1 | 36 / 34 / 10 | 22.33% | 16.67% |
| last 80, n = 2 | 31 / 34 / 15 | **63.02%** | **66.67%** |

With the assumption broken (n = 2), the unchanged metric **nearly triples**
mean PA similarity. Each bigram counts as a match if it shares any single
token, so the ratio inflates instead of becoming stricter. Nothing warns
about this: the run succeeds with 0 errors.

### Option B: chosen change, measured

| Run | PA / NE / PN | Median adoption | PA mean sim. (pre -> post) | Rows changed |
|---|---|---:|---:|---:|
| first 80, n = 1 | 42 / 26 / 12 (unchanged) | 59.47% (unchanged) | 31.45% -> 31.45% | **0 / 5364** |
| last 80, n = 1 | 36 / 34 / 10 (unchanged) | 59.31% (unchanged) | 22.33% -> 22.33% | **0 / 1924** |
| last 80, n = 2 | 31 / 34 / 15 (unchanged) | 64.76% (unchanged) | 63.02% -> **14.05%** | 623 / 1924, all down |

- At n = 1 the change is behaviour-preserving: 0 of 7,288 rows changed.
- At n = 2 it corrects 623 rows, all downward, as expected when moving from
  "shares a token" to "shares the whole n-gram".

### Option C: hard-code n = 1, assessed but not implemented

`NGRAM_SIZE` / `ngram_size` is referenced in 13 files, including the
`-n/--ngram` CLI flag in `PatchTrack.py`, `patchLoader`, `sourceLoader`,
`common.PatchInfo` and 5 test modules. Removing the knob would spread changes
across all of them and diverge from the published tool. That cost is too high
for a defect that Option B fixes in one function.

### Option D: God-class decomposition, rejected

The final diff does not touch `analyzer/main.py`:

```
PatchTrack/analyzer/classifier.py   | 19 +++++++++++++------
PatchTrack/tests/test_classifier.py | 35 +++++++++++++++++++++++++++++++++++
2 files changed, 48 insertions(+), 6 deletions(-)
```

## Prediction check

| # | Prediction | Outcome |
|---|---|---|
| 1 | Pressure lands on `cal_similarity_ratio` / `common.ngram_size`, not the God class | **Confirmed.** Diff is confined to `classifier.py` + its tests |
| 2 | Results will not move at n = 1 | **Confirmed for the final change**: 0 / 7,288 rows. The first fix attempt **did** move 536 rows at n = 1 (all `0.0` -> up to `100.0`), which exposed a second hidden assumption (see below) |
| 3 | n = 2 test fails before, passes after | **Confirmed** (4 failing -> passing) |
| 4 | Blast radius <= 3 files | **Better than predicted**: 2 files. No `main.py` call-site change was needed |
| 5 | `n` must be passed explicitly | **Partly falsified.** The global is unreliable, as predicted, but an explicit parameter would be unreliable too. `patchLoader` shrinks the global per hunk, so one `source_hashes` list can mix n-gram lengths. Deriving `n` from each n-gram is the only correct option |

## Second hidden assumption found

`PatchLoader._add_patch_from_diff` sets `common.ngram_size = len(diff_norm_lines)`
for short hunks. For a **deletion-only hunk** that is **0**, so `SourceLoader`
then emits empty-string "n-grams". The M4 code never matched `''` by accident:
it iterates over `''.split() == []`. Our first version built 0-length added
n-grams (`''`) that matched them, which flipped 536 rows from 0% to non-zero
at n = 1. This was caught only by the row-level comparison, because
classification counts and the median did not change. Fixed by never building
0-length added n-grams, with a dedicated test. `''` stays in the denominator
to keep exact n = 1 parity.

## Other observation (not caused by this change)

The notebook's committed output reports a last-80 median adoption of
**33.33%** (and `results_wk8.md` reports 33%). Using the **same 627 PA file rows
from the committed cache** with the notebook's own formula on the upstream
dataset gives **59.31%**. Per-PR values agree for some PRs (e.g.
`WordPress/gutenberg#54172` = 3.68% in both) and differ for others. This
suggests the group's local `data/` differed from the upstream dataset. The
median is therefore **not reproducible from the repository alone**: the input
data is gitignored. `similarityRatio` does not feed classification or the
median, so this is independent of Event B.

## Conclusion

- **Week 9 decision: partly supported.** Deferring the God-class refactor was
  right: the pressure was resolved with a 2-file change at a module boundary,
  and `main.py` was untouched. However, the actual weakness was not class
  size. It was **mutable global configuration (`common.ngram_size`) written by
  three modules and read implicitly by a fourth**. Week 9 did not identify
  this coupling.
- **Is the M4 AI change robust at system level?** No. It was locally correct
  only under two unstated assumptions: n = 1 and "empty n-grams can't match".
  The second assumption held only by accident of `''.split()`.
- **Accepted trade-off:** added n-grams are built per line, so at n > 1 a
  sequence split across two added lines is not matched. This can only
  under-count, never inflate.
- **Remaining anti-pattern risk:** `common.ngram_size` is still mutable global
  state (set in `main._process_patch_pair`, `classifier.process_patch`,
  `PatchLoader._add_patch_from_diff`, `SourceLoader._build_bloom`).
  `cal_similarity_ratio` is now immune to it, but the other readers are not.
  Also, `_process_patch_pair`'s blanket `except Exception` would hide a
  contract break as a silent `ERROR` row.
- **Evidence:** the tests above, the row-level pre/post comparison
  (0 / 7,288 at n = 1; 623 corrected at n = 2), and the 2-file diff.
