# Lean toolchain and beads setup review

**Date:** 2026-11-25T10:00
**Scope:** environment check plus the fork-only beads setup on `beads-setup` (`.beads/`, `CLAUDE.md`, `docs/session-summaries/2026-11-25-100500-lean-toolchain-and-beads-setup.md`). No Lean physics sources.
**Commits:** `eb1fa9fd` (parent `7910f51f`, `origin/master`), `37eca4e8` (branch tip, matches `origin/beads-setup` at review time)

---

## Critical

None.

## Warning

### 1. CLAUDE.md names a JSONL file that is not in the tree, and its triage jq paths return empty

**File:** `CLAUDE.md` (lines 8, 71, 83-84, 92)

The issues file on disk is `.beads/issues.jsonl`. There is no `.beads/beads.jsonl`. `CLAUDE.md` tells agents to read `.beads/beads.jsonl` and to treat `data_hash` as a fingerprint of that file.

`bv --robot-triage` on this checkout (bv 0.2.1) returns top-level keys `data_hash`, `generated_at`, `output_format`, `triage`, `usage_hints`, `version`. `quick_ref` and `recommendations` are inside `.triage`. These two examples therefore print `null`:

```
bv --robot-triage | jq '.quick_ref'
bv --robot-triage | jq '.recommendations[0]'
```

Use `jq '.triage.quick_ref'` and `jq '.triage.recommendations[0]'`.

Measured the same morning, so the other jq examples are not the same bug: `--robot-plan` exposes `plan` at the top level, `--robot-alerts` exposes `alerts` and `summary` at the top level, and `--robot-next` is a flat object. Triage is the command the file calls the entry point, and that entry point's examples are wrong. `status` is also absent from the triage top level, while line 72 says every robot payload includes it.

### 2. The session summary says doctor is clean

**File:** `docs/session-summaries/2026-11-25-100500-lean-toolchain-and-beads-setup.md` (line 15)

`br doctor` (br 0.2.18, re-run at review time) ends `HEALTH workspace: healthy`, with the database and `.beads/issues.jsonl` both at 2 records and in sync. It also prints warnings. "reports everything OK" is how a `tail` of the OK block reads, and it will send the next agent past the warnings below.

The same paragraph's PATH note ("shells that started before the install") is incomplete. The elan export is `~/.profile` line 49. `~/.zshenv` has no elan line. Non-login shells, including agent shells started after the install, do not read `~/.profile`, so they still need `PATH="$HOME/.elan/bin:$PATH"` or an export in `~/.zshenv`. That fix is outside this repository, so it is not a physlib issue.

## Suggestion

### 1. Doctor warnings that should not be re-filed as setup failures

Re-run at review time, after `37eca4e8`:

- `gitignore.beads_inner_present`: `.beads/.gitignore` lacks the literal pattern `.write.lock`. Line 8 `*.lock` already ignores `.beads/.write.lock` (`git check-ignore -v` confirms). Adding the explicit pattern would only silence doctor.
- `db.sidecars`: `beads.db-wal` exists without a SHM sidecar. Doctor calls this expected for frankensqlite.
- `base_jsonl.missing_post_flush`: `.beads/beads.base.jsonl` is missing even though `metadata.last_export_time` is `2026-11-25T17:56:49.395952967+00:00`. That path is a local merge anchor and is gitignored. A later `br sync` flush can create it. Do not commit it.
- `write_lock`: `.beads/.write.lock` was about 322 seconds old and doctor called it orphaned. No `br` process was in the process list. The lock was left in place.

`OK base_jsonl` and the missing-anchor warning both appear in the same run. The health line is real. The anchor file is still absent.

### 2. Close reason cites the commit that still had the issue open

`physlib-v9v` is closed. The close reason names `eb1fa9fd`. That commit still stored the issue as open. The closed record is in `37eca4e8`. The setup itself is on the branch. No need to reopen `physlib-v9v`.

### 3. physlib-omp already covers the stray master commits

Local `master` is still `eaa8acc6`, two commits ahead of `origin/master` (`7910f51f`): `ca04e9c1` (unfilled session-summary template) and `eaa8acc6` (`add project`, including an empty `.env` and a Node `package.json`). `physlib-omp` states the check, the command `git branch -f master origin/master`, and the done-when count of 0. Its body says the beads setup sits on `eb1fa9fd`; the branch tip is now `37eca4e8`. This review did not move `master`.

## Positive

- Build logs still end in the success lines the summary cites: `Build completed successfully (10503 jobs).` for the default build and `Build completed successfully (10758 jobs).` for PhyslibAlpha. A search of those logs for `error:` hit nothing else. No second `lake build` was started for this review.
- `beads-setup` is based on `origin/master` (`7910f51f`), not on the stray local `master` commits. `eb1fa9fd` adds `.beads/` and `CLAUDE.md` and leaves out `.env` and `package.json`. `37eca4e8` adds the session summary and records `physlib-omp`. Both commits are on `origin/beads-setup` (fork `superradiant-research/physlib`). `upstream` is `leanprover-community/physlib` and was not pushed.
- Commit messages on those two commits have no Co-authored-by trailer.
- `physlib-omp` is the right needs-human ticket for the master reset, with a check to run first and a done-when read-back. The two commits it names are the ones `origin/master..master` contains.
- Doctor's health line, the matching JSONL and database counts (2 records), and the rollback write probe are all consistent with a working tracker. Prefix `physlib` is in use on `physlib-v9v` and `physlib-omp`.
