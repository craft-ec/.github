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
   - design check: matches `ARCHITECTURE.md`, or the PR updates it
   - anything measured: the number, the command, the machine
6. **Squash-merge**, delete the branch. `main` stays linear and always green.

## Labels

| Group | Labels |
|---|---|
| type | `feature` · `bug` · `measure` · `docs` · `chore` |
| track | `substrate` · `sdk` · `builder` |
| phase | `phase-0` … `phase-13` |
| state | `needs-decision` (blocked on a design call) · `blocked` |

## Milestones

One per build phase (see the architecture's build order). A phase closes when its
"done when" holds and its numbers are recorded.

## Design changes

The architecture is the source of truth. A PR that changes behaviour the
architecture describes must change the architecture in the same PR series. A
decision that is not obvious gets a `needs-decision` issue first.
