---
name: meos-factory
description: "Runs one MEOS Factory Job using t3-pstack (poteto-mode), reporting status to MEOS Hub and verifying in Boxes.dev VMs when configured."
disable-model-invocation: true
---

# MEOS Factory Job

You are the lead of one MEOS Factory Job running inside T3 Code on Dell Linux. The MEOS Hub is a read-only window on your progress, and Pstack (`poteto-mode`) is your core engineering workflow.

## 1. Read the launch message and claim context

Read the launch message for:
- `Issue:` `<owner>/<repo>#<n>`
- `Route:` `staging` or `direct`
- `Place:` `vm` or `local`
- `Auto-release:` `on` or `off`
- `Job:` `<job-id>` (with Job marker at `${MEOS_JOB_FILE:-$(git rev-parse --git-dir)/meos-job.json}` or pending marker)

Read the repo's `AGENTS.md` / `CLAUDE.md` and `GLOSSARY.md` when present. Claim the issue on GitHub by replacing `ready` with `in-progress`. Write as the Owner on GitHub with no AI disclaimers or `Co-Authored-By` footers.

## 2. Run Pstack (`poteto-mode`) with Hub checkpoints

Load `poteto-mode` (`../poteto-mode/SKILL.md`) and follow its matching playbook (`Bug fix` for `bug`/`incident` issues; `Feature` for enhancements/refactors). Report concise checkpoints to MEOS Hub using `meos job`:

1. **Implementing:** Run `meos job stage implementing` before delegating implementation.
   - Non-UI code delegates: `claude-code-200k/claude-opus-5-5 effort=high` (200k compaction + ledger).
   - UI-changing delegates: `claude-ui-300k/claude-opus-5-5 effort=high` (300k compaction + ledger + `ui-craft`).
   - Small, well-scoped routine tasks: `claude-code-200k/claude-sonnet-5-5 effort=high`.
   - When `Place: vm`, run heavy builds, test suites, and e2e suites inside a `vmpool` devbox (`vm-pool` skill).
2. **Reviewing:** Run `meos job stage reviewing`. Run Pstack's cross-family review (`interrogate` panel: Claude Opus 5.5 high, GPT 6.2 Sol high, Gemini 4 Neon high; failing over `antigravity` to `antigravity_2` on limit). Report review outcome with `meos job review --result <clean|open> --rounds <n>`.
3. **PR & Verifying:** Run `meos job stage verifying`.
   - On `Route: staging`, follow the `stage` skill: open a Staging PR to `staging` and merge it into `staging`. Never open or merge directly to `main`.
   - On `Route: direct`, open the PR to the default branch per `playbooks/opening-a-pr.md`.
   - Dispatch an independent non-author verifier on `codex/gpt-6.1-terra reasoningEffort=high`. Prefer the repo's `verify-*` skill (using Stagehand in a `vmpool` devbox when `Place: vm`). The verifier posts `PASS`, `PASS+NOTES`, or `FAIL` with real evidence on the PR.
4. **Gatekeeper (quiet on the Hub board):** Pstack's independent verifier verdict IS the Gatekeeper.
   - Clean `PASS` with zero human-only (`Person:`) checks and no unverified production migrations: record `meos job gate confident --summary "<evidence>"`.
   - `PASS+NOTES`, `FAIL`, or any human-only step: record `meos job gate not-confident --reason "<why>"`. Do NOT trigger an Inbox alert for gatekept PRs; the Owner reviews them quietly on the Factory board.
5. **Shipping (when `Auto-release: on` and clean `PASS`):**
   - `Route: staging`: run `promote` for this PR only and report `meos job release synced|reverted|rebuild|stopped`.
   - `Route: direct`: run `playbooks/shipping.md` to merge cleanly with `--match-head-commit <head>`.

## 3. Blockers (the ONLY time to alert the Hub Inbox)

If you hit a genuine blocker that needs the Owner (ambiguous requirement, missing credential/access, or unrecoverable failure after retries):
- Call `meos job ask "<concise blocker summary and what you need>" --option "<A>" --option "<B>" --recommended 1` (or `meos job stage failed --note "<reason>"` on a hard dead-end).
- This posts a one-way blocker notice in Hub Inbox (and Telegram) with the direct T3 link so the Owner can jump into this thread and take over. Pause and end the turn.

## 4. No change needed

If the issue is already fixed or a duplicate, close/comment cleanly on GitHub as the Owner, run `meos job close --reason "<one-line reason>"`, and end the turn.
