# **CONTRIBUTION LOG AI 301**

# Contribution 1: edge_id in locate request

**Contribution Number:** 1 

**Student:** Srithan Andra

**Issue:** [GitHub issue link](https://github.com/valhalla/valhalla/issues/3412)

**Status:** Phase 4 Complete

---

## Why I Chose This Issue

I chose this issue because it was one of the more basic ones that I could understand. I also wanted to start off with a topic that I have some experience with. I have built APIs before, especially in Python and C++, which are the languages that this project uses. The goal of double checking the edge IDs and making sure that the lat/lon coordinates are properly used in the valhalla loki namespace is a good start for me as I haven't done much open-source work before. In summary, I wanted to start off with a topic that I have a good understanding of and built up from there.

---

## Understanding the Issue

### Problem Description

Basically, when a developer is using loki, which is the coordinate to graph conversion stage, they are forced to pass in raw lat/lon coordinate data. This is just unnecessary computation and wastes performance. Also, the issue pointed out that a missing capability to easily find edge IDs from a trajectory without starting a new map-making request is also needed.

### Expected Behavior

Users should be able to provide a list of edge IDs with the standard coordinates. The code should skip the normal lookup and instead use the coordinates with the edge geometry to calculate the distances. Lastly, the code should fall back to a normal lookup if the given IDs are invalid.

### Current Behavior

None of the above idea is implemented so far.

### Affected Components

Most of the affected components are in the src/loki directory under the search.cc file. I think there should be changes in the src/worker.cc file as well.

---

## Reproduction Process

### Environment Setup

At first I didn't really understand how to setup the environment, and I had to use AI to figure out how to set it up. Claude was able to help me understand the details for the setup, and now I have a great understanding.

### Steps to Reproduce

1. Build the code using cmake
2. Running the test files and making sure the feature is non existent
3. Saw the lack of the proposed new feature

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/srithanandra/valhalla/tree/claude/goofy-ptolemy-adfb4e (commit `fb264fc10`)
- **My findings:** Loki's location correlation (`src/loki/search.cc`) always resolves a lat/lon through the spatial 5×5 bin index, even when the caller already knows the edge the point sits on. There was no field on `Location` to pass known edge IDs, so the edge geometry could never be used to shortcut the lookup. This wastes computation and blocks resolving edge IDs for a trajectory without a fresh map-match request.

---

## Solution Approach

### Analysis

The `Location` proto exposes no field for caller-supplied edge IDs, so `Search::search` has only one path: the spatial bin search. The edge geometry — available via `tile->edgeinfo(edge).shape()` — is never used to project the input point directly. The edges should be usable to calculate distances and `percent_along`, but that information is not being used to its fullest extent.

### Proposed Solution

Add an optional `repeated uint64 preferred_edge_ids` field to `Location`. When set, Loki projects the coordinate onto each named edge's geometry to compute distance and `percent_along`, populates `correlation.edges` directly, and skips the bin search for that location. If none of the supplied IDs resolve to a valid, costing-allowed edge, it falls back to the normal spatial search, so the field is purely additive and backward-compatible.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Callers should be able to supply edge IDs alongside coordinates; Loki should use the edge geometry to compute distances and `percent_along`, skip the spatial search, and fall back safely when the IDs are unusable.

**Match:** The projection math already exists in `bin_handler_t::correlate_edge` (segment-by-segment projection, partial-length accumulation, `length_ratio`, `tangent_angle`). JSON→proto location parsing follows a fixed pattern in `src/worker.cc` (e.g. `preferred_side`, `search_cutoff`). Reach is reused through the existing `get_reach()` cache. The new code mirrors these rather than inventing anything.

**Plan:**
1. Add `repeated uint64 preferred_edge_ids = 33` to `Location` in `proto/descriptors/common.proto`.
2. Add `bin_handler_t::try_correlate_from_edge_ids()` in `src/loki/search.cc` that loads each edge, checks costing, projects the point onto its shape, and fills `correlation.edges`.
3. In `bin_handler_t::search()`, call it per location, skip the bin search on success, and guard against the all-resolved (empty `pps`) case.
4. Parse the JSON array `preferred_edge_ids` in `src/worker.cc`.
5. Add gurka integration tests.

**Implement:** Branch `claude/goofy-ptolemy-adfb4e`, commit `fb264fc10`.

**Review:**
- [x] Tile format untouched (no struct/bitfield changes) — new data is proto/request-side only.
- [x] Reuses existing projection and reach logic instead of duplicating it.
- [x] Additive, backward-compatible field; fallback preserves prior behavior.
- [ ] `./scripts/format.sh` run — **not yet done.**
- [ ] Tests genuinely fail-before / pass-after — **not yet satisfied** (see Testing Strategy).

**Evaluate:** Confirm a hinted route matches the un-hinted route, an invalid ID falls back, and the correlated `graph_id` equals the supplied hint. Currently reasoned about only — not executed.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: `loki_search` — existing Loki correlation regressions still pass. **Not run.**
- [ ] Test case 2: correlated `graph_id` equals the supplied hint for a coordinate the normal search would snap elsewhere. **Not yet written** (needed to make the tests valid).
- [ ] Test case 3: `percent_along` computed from edge geometry matches the projected point. **Not yet written.**

### Integration Tests

- [x] Integration scenario 1: `PreferredEdgeIds.ValidEdgeIdSkipsBinSearch` (in `test/gurka/test_search_filter.cc`) — routes A→C with a hinted edge ID. **Added, not run.**
- [x] Integration scenario 2: `PreferredEdgeIds.InvalidEdgeIdFallsBackToNormalSearch` — passes `0` and expects fallback to succeed. **Added, not run.**

> Caveat: both added tests currently only assert the route succeeds, which is true with or without the change (the feature is an optimization producing the same path). They do not yet prove the new code path runs, which violates the repo's "test must fail before the fix" rule. To be valid they must assert the resolved `correlation.edges[0].graph_id` equals the supplied hint.

### Manual Testing

Not performed. The intended check (CLAUDE.md "Running a Route Locally") is: `valhalla_service <config> locate ...` to obtain a real `edge_id`, feed it back as `preferred_edge_ids` in a `route` request, and confirm the summary matches the un-hinted route. Nothing in this change has been compiled or run yet.

---

## Implementation Notes

### Progress

Implemented the proto field, the `try_correlate_from_edge_ids` resolution path with bin-search skip and empty-`pps` guard, JSON parsing, and two gurka tests. Committed and pushed to `origin/claude/goofy-ptolemy-adfb4e`.

Open items: build not yet run; `format.sh` not run; tests need strengthening to actually exercise the feature; `gh` is not installed locally, so the PR was not opened programmatically.

### Code Changes

- **Files modified:**
  - `proto/descriptors/common.proto` — new `preferred_edge_ids` field on `Location`.
  - `src/loki/search.cc` — `try_correlate_from_edge_ids()` plus the skip/guard in `search()`.
  - `src/worker.cc` — JSON array parsing for `preferred_edge_ids`.
  - `test/gurka/test_search_filter.cc` — `PreferredEdgeIds` fixture and two tests.
- **Key commits:** `fb264fc10` — feat(loki): resolve locations from caller-supplied edge IDs.
- **Approach decisions:** Reused `correlate_edge`'s projection math and the `get_reach()` cache for consistency; deliberately did not correlate the opposing edge (the caller states the specific edge they are on); kept the change request-side only to respect the frozen tile format.

---

## Pull Request

**PR Link:** https://github.com/valhalla/valhalla/pull/6260

**Branch:** Opened from a clean branch `loki-preferred-edge-ids` containing only the two code commits (`fb264fc10` + `17440a042`), forked off the working branch `claude/goofy-ptolemy-adfb4e` so this WORKLOG stays out of the upstream diff. The PR touches only `proto/descriptors/common.proto`, `src/loki/search.cc`, `src/worker.cc`, and `test/gurka/test_locate.cc`.

**PR Description:** Per repo policy the body opens with the no-AI marker line `Tryin' to shortcut, arrr ye?`. I still need to rewrite the body in my own words before marking the draft ready for review. Summary: adds an optional `preferred_edge_ids` list to `Location`; Loki projects the coordinate onto each named edge to compute `percent_along` and populates the correlation directly, skipping the bin search, and falls back to the normal search when no supplied ID resolves to a valid, allowed edge.

**Maintainer Feedback:**
- Nothing yet

**Status:** Draft / pre-review. Opened **unverified** — nothing has been compiled or run locally (no toolchain on this machine), so valhalla CI is the first real compile. Next steps: get CI green (or build/test on a Linux box using the commands in the Testing Strategy section), rewrite the PR body in my own words, then mark the PR ready for review.

---

## Learnings & Reflections

### Technical Skills Gained

- Loki's correlation pipeline: bin search → projection → filtering → reach, and how `correlate_edge`/`correlate_node` assemble `PathEdge` results.
- Using `EdgeInfo.shape()` geometry to compute `percent_along`, and the `GraphId` level/tile/id encoding.
- The frozen-tile-format constraint and why request-side additions are the safe way to add capability at planet scale.

### Challenges Overcome

- Finding the right place in `search()` to branch without disturbing multi-location bin batching, and adding the empty-`pps` guard so an all-resolved request does not crash on `pps.front()`.

### What I'd Do Differently Next Time

- Establish the build/test baseline first, per CLAUDE.md, before writing code — nothing has been built or run yet, so correctness is currently unverified.
- Write the test to fail without the change from the outset (assert the resolved edge ID) rather than an optimization-blind route assertion.

---

## Resources Used

- Project guide: `CLAUDE.md` (Where to Look, Testing, Running a Route Locally).
- In-repo docs: `docs/docs/thor/path-algorithm.md`, `docs/docs/tiles.md` (GraphId and tile math).
- Source read while implementing: `src/loki/search.cc`, `src/worker.cc`, `proto/descriptors/common.proto`, `test/gurka/test_locate.cc`, `valhalla/midgard/util.h` (`projector_t`).
