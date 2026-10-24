# Last Report — full-project workflow run

Scope: the `donmemedo/donmemedo` GitHub **profile README** repo. The only tracked product is `README.md`.
Paths below are repo-relative on purpose. Nothing here was measured on a network or a running service.

## Outcome in one paragraph

There is no application in this repo, so none was built. The workflow's reviews, agent consultation, static checks and security review ran against the profile README and the untracked Claude Code bootstrap (`.claude/`, `CLAUDE.md`, `docs/`, `scripts/`, `setup-claude-project.sh`). No secret was found. `README.md` was **not modified**. The real risks are about *what gets published* from this repo, not about code defects.

## Chronology

1. Session resumed after a usage limit. The bootstrap (`docs/CLAUDE_SETUP_REPORT.md`, 2026-10-23) was complete; no `HANDOFF.md`, `docs/LAST_Report.md` or graph existed, and nothing was committed.
2. `/graphify .` ran: 56 files (5 code, 51 docs), 3 extraction subagents, **264 nodes / 647 edges / 27 communities**. Health check warned: 4 dangling-endpoint edges, 3 undirected edge collapses. Extraction used 437,269 subagent tokens (the harness reports one combined total; graphify records it as 437,269 input / 0 output). `.claude/agent-memory/build/MEMORY.md` is on disk but absent from the corpus; cause not verified.
3. `/ponytail:ponytail` loaded (plugin v5.3.2). `/ponytail-review` loaded and the review was performed on the hook, both scripts and `setup-claude-project.sh` lines 1–460. Lines 460–795 (launch prompt and embedded payload) were not read line by line by the orchestrator; the payload was decoded and scanned by `security-auditor`.
4. `triage` returned `{"route":"build","risk":"medium","specialist":null}`.
5. 19 agents were consulted (list below). 11 were interrupted by an API rate limit and resumed from their saved transcripts; all 11 then reported.
6. Findings below; two deliverables (this file, `HANDOFF.md`) written; `verifier` run last (see ledger).

## Commands, skills, agents — actual status

| Item | Status | Evidence |
|---|---|---|
| `/graphify .` | Done | `graph.json`, `graph.html`, `GRAPH_REPORT.md`, manifest written under `graphify-out/` |
| `/ponytail:ponytail` | Loaded | skill text returned; alias registered, no `/ponytail` fallback needed |
| `/ponytail-review` | Loaded, review done | findings F4–F11 below |
| Agents consulted (19) | Reported | triage, product-owner, tech-lead, prototyper, grower, maintainer, sweeper, security-auditor, blue-team, blue-team-lead, red-team, red-team-lead, qa-qc, qa-qc-lead, build, quick, builder, deep, escalate |
| `verifier` | First pass: FAIL on four factual defects in the draft (stub count, token wording, publication tense, untracked-file count) — all corrected | Re-checked by the orchestrator with commands, **not** by a second verifier run |
| Specialist skills (26) | Present on disk | `scripts/claude-skill-audit.sh`: 27 AVAILABLE (26 + graphify). **Whether each agent actually loaded its preloaded skills was not verified.** |

## Checks that actually ran

| Check | Result |
|---|---|
| `bash -n` on `setup-claude-project.sh`, `scripts/validate-universal-bootstrap.sh`, `scripts/claude-skill-audit.sh` | exit 0 ×3 |
| `shellcheck` | **Not run** — not installed |
| AST parse of `.claude/hooks/user-prompt-submit.py` (no bytecode written) | exit 0 |
| JSON parse of `.claude/settings.json`, `graphify-out/graph.json` | exit 0 ×2 |
| `scripts/validate-universal-bootstrap.sh` | exit 0 — 20 agents, 26 skills assigned |
| `scripts/claude-skill-audit.sh` | exit 0 — but it **never exits non-zero for a missing skill**, so exit 0 proves little |
| Secret / PII / local-path scan (regex + source review, payload decoded and rescanned, not executed) | **No true secrets.** Local `/home/<user>` paths found (F2) |
| Tar extraction test (GNU tar 1.35, crafted archive with symlink, `..`, absolute and hardlink members) | Nothing written outside the extraction dir; tar aborted the script (rc=2) |
| `setup-claude-project.sh --dry-run` with hostile `--project` names (`$(…)`, backticks, `;`, quotes, newline) on scratchpad copies | No files created, no injection |
| Reference-integrity script over all backticked paths/commands in docs | See F6 |

Reported by agents and **not re-verified** by the orchestrator or verifier: the tar-extraction test, the `--dry-run` injection tests, the secret-scan details, and the 4-dangling/3-collapsed graph health counts (that one is graphify's own output).

**Not run:** any real run of `setup-claude-project.sh`; link/widget liveness (no network by rule); DAST, SAST scanners, container scans (no app, no Dockerfile); performance measurement (nothing to measure — no service); GitHub rendering of the README; `/plugin` listing inside Claude Code.
No penetration test was performed. The red-team work above is a local static/dry-run review.

## Findings

**Publishing risks (act before anything is committed to this public repo)**
- **F1 — work email in public history.** The global git identity is a work address; a plain commit would publish it, and deleting a branch does not unpublish fetched SHAs. Use the GitHub noreply address for any commit here.
- **F2 — local path exposure.** `docs/CLAUDE_SETUP_REPORT.md` (line 4, lines 67–97) and `graphify-out/.graphify_root`, `graphify-out/.graphify_python` contain `/home/<user>` paths, which reveals the username.
- **F3 — no `.gitignore`.** `git add -A` would stage over 150 untracked files (156 when last counted), including graph caches and generated bytecode. Stage by explicit path only.

**Bootstrap configuration (local tooling; fix at the owner's discretion)**
- **F4 — "read-only" verifier is not read-only.** `.claude/agents/verifier.md:27` says "You are a read-only verifier. Do not edit files…", but its frontmatter `tools:` grants `Write, Edit`. The restriction is advisory only. Fix: drop `Write, Edit` from `verifier.md`.
- **F5 — nothing is pinned.** Plugin marketplace `.claude/settings.json:37-47` (no ref/SHA); `setup-claude-project.sh:286,291,295-301` (`npx --yes skills add` from 7 repos); `:310,:314` (`graphifyy` unversioned); `:329-330` (plugin install). Failure case: an upstream skill rename makes `install_group` fail, `|| true` swallows it, and the validator still prints PASS because it only checks names in agent frontmatter.
- **F6 — dangling reference.** `~/bin/claude-project-init.sh` is cited at `docs/AGENT_AND_SKILL_POLICY.md:19`, `setup-claude-project.sh:26,45,46`, `docs/UNIVERSAL_CLAUDE_SETUP.md:4`; it does not exist, so the documented commands fail from the repo root.
- **F7 — lists that must change together.** The 26-skill list is copied in `setup:295-301`, `setup:339`, `validate:19`, `audit:6-13` (plus agent frontmatter); the agent count `20` is hard-coded at `validate:10` and `setup:101,366,422`.
- **F8 — audit script exit code.** `scripts/claude-skill-audit.sh` exits 0 even when a skill is MISSING.
- **F9 — setup script edge cases.** `CLAUDE.md` is rewritten with no backup (`setup:202-232`; content between the markers is replaced); the marker rule at `:169` replaces any user-edited managed file that still contains the marker (a backup is made first); a relative `--project` starting with `-` fails at `:80` (use `cd --`). Low severity.
- **F10 — symlink following (from reading only, untested).** `mkdir -p`/`cp -a`/`rm -rf` at `setup:164,171,178` follow symlinks inside the project dir; a committed symlink such as `.claude -> …` in a cloned repo could redirect writes.
- **F11 — hooks.** `graphify hook-guard` fails open; with `graphify` missing it returns rc=127, not the blocking rc=2 (tested via `env PATH=… sh -c`, **not tested inside Claude Code**). `CLAUDE_PROJECT_DIR` is echoed into model context (`user-prompt-submit.py:26`). No `permissions` allow/deny rules exist in project or user settings. Anyone who opens this folder and trusts it runs the project hooks and is offered the third-party plugin.
- **F12 — clutter.** `.claude/settings.json.graphify-bak` is a stale older copy; `.claude/CLAUDE.md` repeats the root graphify block; four docs restate `docs/FULL_PROJECT_WORKFLOW.md`; the 20 `MEMORY.md` files were created as near-empty stubs (11 now hold a lesson line that agents appended during this run). The validator's `py_compile` creates `.claude/hooks/__pycache__/` (observed created, then deleted by `qa-qc`).

**README.md (product) — verified, not changed**
- Line 12: two U+200C characters immediately before the "&". Line 129: empty `<div align="center"></div>`. These two are unambiguous defects.
- Owner decisions, not defects: empty `## Recent Blog Posts` heading (line 113); Linux (lines 43, 57) and Bash (45, 59) icons listed in both Backend and DevOps; whether Keras/PyTorch/OpenCV/MongoDB/MySQL belong under DevOps; the "since 2010" claim.
- Accessibility/rendering (read, not rendered): no alt text on lines 2 and 123; no h1; `style=`/`target=` attributes are likely stripped by GitHub (relying on known sanitizer behaviour, untested); line 108 hardcodes `theme=dark`.
- Privacy: `komarev.com` view counter (line 123) and the stats card (line 108) are third-party requests; the counter sees visitor IP/user-agent. Five third-party image hosts; all URLs are `https`.
- Persian UX vocabulary and RTL guidance do not apply to this English README.

## Decisions taken

- **Not built:** FastAPI, Next.js, PWA, localisation, CI, tests. The repo has no code and the owner did not request that stack for it (`product-owner`, `builder`, `prototyper` agree).
- **Not committed:** `.claude/`, `CLAUDE.md`, `scripts/`, `setup-claude-project.sh`, `graphify-out/`, and every other `docs/` file (F1–F3, `tech-lead`, `escalate`).
- **Publication:** authorised by the owner's launch flag. The planned scope is only the two deliverables (`HANDOFF.md`, `docs/LAST_Report.md`), on a feature branch, as a draft PR, committed with the GitHub noreply identity and never pushed to `main`. Whether the branch and PR actually exist is stated in the PR itself and in the session summary, not in this file.

## Remaining work (owner)

1. Decide whether these two reports should live in the public profile repo at all; closing the draft PR and deleting its branch is the clean way to say no (SHAs stay fetchable).
2. Optionally apply the README fixes (line 12, line 129; then the owner-decision items).
3. Optionally: remove `Write, Edit` from `verifier.md`, pin the plugin/skill sources, replace `~/bin/claude-project-init.sh` references, add a `.gitignore` or `.git/info/exclude` entry for `.claude/` and `graphify-out/`.
4. Install `shellcheck` and run it on the three scripts.
