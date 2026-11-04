# FalkorDB branch `falkordb/i64`

This branch of [FalkorDB/feral](https://github.com/FalkorDB/feral), a fork of
[jkitchin/feral](https://github.com/jkitchin/feral), carries the changes FalkorDB needs in
three ordering crates: `feral-ordering-core`, `feral-amd` and `feral-metis`. FalkorDB's CCH
index (`graph/src/index/falkordb/cch`, cargo feature `index-falkordb`) orders road-network
graph skeletons with `feral_metis::metis_order` and depends on this branch by git revision.

It starts from upstream `main` at d3f7eb6 (feral 0.22.0), where the three crates are the
crates.io releases feral-ordering-core 0.2.2, feral-amd 0.2.1 and feral-metis 0.3.0 (their
`src/` is byte-identical to the published packages). Versions are not bumped.

## Commits

1. **64-bit indices.** `feral_ordering_core::Idx = i64`; every `i32` index, count and
   weight in the three crates is an `Idx`. Permutation unchanged.
2. **Linear component split** (`node_nd.rs`, `split_and_recurse`): one counting sort
   instead of one scan of all vertices per component (132 s -> linear on a road
   network with 294k isolated vertices). Permutation unchanged.
3. **Recursion by value** (`node_nd.rs`, `recurse`): a level's graph is dropped before its
   halves are ordered (peak memory -38% on California). Permutation unchanged.
4. **Linear 2-hop matching** (`coarsen.rs`, `two_hop_pass`): a 200k-leaf star goes from
   11 s to 22 ms. Permutation unchanged.
5. **Node FM `mindiff`** (`fm_refine.rs`): of equally light separators keep the
   better-balanced one, as METIS's `sfm.c`. Changes the permutation.
6. **Best of five bisections** (`node_nd.rs`, `best_of_bisections`) for subgraphs of
   5,000 vertices or more, as METIS's `MlevelNodeBisectionL2`. Changes the permutation.

With 5 and 6, California's chordal hierarchy goes from 5,932,138 arcs / 59.6M triangles
to 5,838,003 / 53.9M (C METIS: 5,527,199 / 48.6M). Each commit message has the details.

## Scope

Only the three crates' libraries, their tests and `feral-amd`'s binaries are converted to
`i64`. `feral-amf`, `feral-scotch`, `feral-kahip` and the root `feral` crate still use
`i32` against these APIs, so they, and a whole-workspace build, do not compile on this
branch. Build and test the three crates by name:

```bash
cargo test -p feral-ordering-core -p feral-amd -p feral-metis
cargo clippy -p feral-ordering-core -p feral-amd -p feral-metis --all-targets -- -D warnings
```
