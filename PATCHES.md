# What fastdbsp changes in dbsp 0.2.0

fastdbsp is the crates.io release of dbsp 0.2.0 with two changes, both in operators that nested (recursive) circuits
use. Each is one commit on top of the unpacked release, so `git diff` against the first commit shows exactly what
differs from upstream.

1. `src/trace/ord/{val,key}_batch.rs`, `recede_to`: after receding a batch to a frontier, set its upper bound to the
   frontier and meet its lower bound with it. Without this, every batch looks unreceded forever, so at the end of every
   outer step each `Z1Trace` walks and re-sorts every batch it holds: the cost of an update grows with the whole state.
2. `src/operator/join.rs`, `JoinTrace::eval`: add up a trace value's weights by output time before calling the join
   function. Updates that land at the same output time (+x from an earlier step, -x from this one) then cancel before
   any output tuple is built; without this, removing rules costs three times adding them.

Both are checked by dbsp's own library tests (`cargo test --release --lib`).
