# How we work

One rule: **nothing lands on `main` without an issue and a pull request.**

## The loop

1. **Issue first.** Every change starts as an issue that says what and why, and
   how we will know it is done. Bugs include what was measured, not what was
   guessed.
2. **Branch from `main`:** `<issue-number>-<short-slug>`, e.g. `12-node-format`.
3. **Commit** in small, imperative, self-explaining commits. No attribution
   trailers.
4. **Pull request** titled like the change, with `Closes #<issue>` in the body.
   Give it the **same labels and milestone as the issue** — GitHub does not
   copy them, and an unlabelled PR cannot be found by type, track or phase.
   One logical change per PR. If it spans repos, open one PR per repo and link
   them; merge in dependency order.
5. **Gates before merge** — all stated in the PR, with output where there is any:
   - `cargo fmt --check`, `cargo clippy -D warnings`, `cargo test`
   - contracts: reproducible wasm build, wasm size and hash recorded
   - formats (anything that fixes bytes or hashes): frozen vectors pass on native
     **and** on wasm32 (`./check-wasm.sh` where the repo has one); a check that
     cannot run — missing tool, missing target — fails, it never skips
   - a test that would fail if the thing were broken (negative control for any
     "X prevents Y" claim)
   - the same for what the code **asserts to its caller**, not only what it
     returns: a proof or evidence it accepts from a third party, the blocks it
     says to fetch, a continuation it hands back, "these bytes are the value".
     Returned data can be right while an assertion is wrong, and nothing
     notices — run a mutant on each acceptance/assertion path before merging
   - **a restored mutant must rebuild.** Restoring a mutated file by moving a
     backup back (`mv f.bak f`, `cp -p`) gives it its OLD mtime, older than the
     mutant's build, so cargo (and any mtime-based build) keeps running the
     MUTANT binary: a later green run can certify the mutant, and a later
     failure looks like a real defect (sdk#235: a new test "failed" on a clean
     tree because the engine was still the previous mutant). Restore with a
     write that bumps the mtime (`git checkout -- f`, or `touch f` after the
     move), and after every mutant batch run the suite green ONCE and say so —
     that run is the only evidence the restore took
   - **every public door to the same check gets the same tests.** When one check
     is reachable through several entry points (bytes and decoded, a wrapper with
     its own pre-gate, native and wasm), honest inputs and the hostile list run
     through each of them and the answers are asserted equal. A fix verified
     through the inner function proves the inner function: the outer door — the
     one callers use — may refuse or accept for a reason of its own
   - design check: matches `ARCHITECTURE.md`, or the PR updates it
   - anything measured: the number, the command, the machine
6. **Gate the merged tree.** A gate certifies the tree it ran on, and for a merge
   that tree does not exist until the merge does: two green PRs can break `main`
   together with no git conflict (one adds a call to what the other renamed). If
   `main` has moved since the branch was cut, run the gates on the branch merged
   with `main` before merging, and build `main` again afterwards. "Merges
   cleanly" is not a gate.
7. **Squash-merge**, delete the branch. `main` stays linear and always green.

## Labels

| Group | Labels |
|---|---|
| type | `feature` · `bug` · `measure` · `docs` · `chore` |
| track | `substrate` · `sdk` · `builder` |
| phase | `phase-0` … `phase-15` |
| state | `needs-decision` (blocked on a design call) · `blocked` |

## Milestones

One per build phase (see the architecture's build order). A phase closes when its
"done when" holds and its numbers are recorded.

## Design changes

The architecture is the source of truth. A PR that changes behaviour the
architecture describes must change the architecture in the same PR series. A
decision that is not obvious gets a `needs-decision` issue first.
