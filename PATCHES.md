# What fastdbsp changes in dbsp 0.2.0

fastdbsp is the crates.io release of dbsp 0.2.0 with two changes, both in operators that nested (recursive) circuits
use, and the dependency changes below, which let it build on current Rust. Each is one commit on top of the unpacked
release, so `git diff` against the first commit shows exactly what differs from upstream.

1. `src/trace/ord/{val,key}_batch.rs`, `recede_to`: after receding a batch to a frontier, set its upper bound to the
   frontier and meet its lower bound with it. Without this, every batch looks unreceded forever, so at the end of every
   outer step each `Z1Trace` walks and re-sorts every batch it holds: the cost of an update grows with the whole state.
2. `src/operator/join.rs`, `JoinTrace::eval`: add up a trace value's weights by output time before calling the join
   function. Updates that land at the same output time (+x from an earlier step, -x from this one) then cancel before
   any output tuple is built; without this, removing rules costs three times adding them.

Both are checked by dbsp's own library tests (`cargo test --release --lib`).

## Dependencies

dbsp 0.2.0 no longer compiles on current Rust: its lock has `time` 0.3.28, which fails on Rust 1.80 and later, and
`size-of` 0.1.5 implements `SizeOf` for calling conventions that Rust now rejects on targets without them. fastdbsp

- locks `time` 0.3.36;
- depends on `feldera-size-of` 0.1.7, the fork of `size-of` that later dbsp releases use, under the name `size-of`;
- builds rkyv in its 64-bit size mode (rkyv's `size_64`, ordered-float's `rkyv_64`, chrono's `rkyv-64`), because
  `feldera-size-of`'s chrono support selects it. Data archived with rkyv is therefore laid out differently from
  dbsp 0.2.0's.

The library and doc tests pass on Rust 1.79 and 1.95.
