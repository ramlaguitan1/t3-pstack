---
name: meos-factory
description: "Runs one MEOS Factory Job using t3-pstack (poteto-mode), reporting status to MEOS Hub and verifying in Boxes.dev VMs when configured."
disable-model-invocation: true
---

# MEOS Factory Job

You are the lead of one MEOS Factory Job running inside T3 Code on Dell Linux. The MEOS Hub is a thin status window on your progress, and Pstack (`poteto-mode`) is your engineering workflow.

## 1. Read the launch message and claim context

Read the launch message for:
- The target `<owner>/<repo>#<n>: <title>` and GitHub issue URL
- The Factory Job name, attempt number, and branch (`This is a Factory Job (<name>), attempt <n>, on the branch <branch>.`)
- `Lane:` `bug`, `feature`, or `none yet` (when `none yet`, pick `bug` or `feature` after reading the issue and record it once with `meos job lane bug|feature`)
- `UI:` `yes` or `no`
- `Place:` `vm (boxes.dev project \`<uuid>\`)` or `local`
- The Job marker at `${MEOS_JOB_FILE:-$(git rev-parse --git-dir)/meos-job.json}` (read automatically by `meos job`)

Read the repo's `AGENTS.md` / `CLAUDE.md` and `GLOSSARY.md` when present. Determine whether the repo uses the **staging route** (repo has a `staging` branch / covered by the `stage` skill) or the **direct route** (PRs target the default branch). Claim the issue on GitHub by replacing `ready` with `in-progress`. Write as the Owner on GitHub with no AI disclaimers or `Co-Authored-By` footers.

## 2. Run Pstack (`poteto-mode`) with Hub checkpoints

Load `poteto-mode` (`../poteto-mode/SKILL.md`) and follow its matching playbook (`Bug fix` for bug lane; `Feature` for feature lane). Every delegated subagent, reviewer, and verifier MUST be dispatched through `delegate_task` (`mode: "async"`, `runtimeMode: "full-access"`) so it nests cleanly under this thread — **NEVER call `create_threads` or `t3_thread_launch`** (those create top-level sidebar threads). On Antigravity/Gemini leads where `delegate_task` is absent from the direct MCP tools and `T3_ACP_MCP_NODE` is set, ALWAYS invoke `delegate_task` through the ACP bridge: `ELECTRON_RUN_AS_NODE=1 "$T3_ACP_MCP_NODE" ${T3_ACP_MCP_ENTRYPOINT:+"$T3_ACP_MCP_ENTRYPOINT"} acp-mcp-call delegate_task '<json>'`. Report concise checkpoints to MEOS Hub using `meos job`:

1. **Implementing:** Run `meos job stage implementing` before delegating implementation.
   - Non-UI code delegates: `claude-code-200k/claude-opus-5-5 effort=high` (200k compaction + ledger).
   - UI-changing delegates (`UI: yes` or UI files touched): `claude-ui-300k/claude-opus-5-5 effort=high` (300k compaction + ledger + `ui-craft`).
   - Small, well-scoped routine tasks: `claude-code-200k/claude-sonnet-5-5 effort=high`.
   - **Dell Linux is control-plane only:** For every repo with a `vmpool` boxes.dev project (`vm-offload project`), offload builds, test suites, Docker/Supabase stacks, dev servers, and Stagehand v3 browser verification to a `vmpool` devbox via `vm-offload` (`vm-pool` skill) or the repo's VM-first `verify-*` skill, and destroy the temporary devbox when finished.
2. **Reviewing:** Run `meos job stage reviewing`. Run Pstack's cross-family review (`interrogate` panel: Claude Opus 5.5 high, GPT 6.2 Sol high, Gemini 4 Neon high). When a reviewer hits a usage limit, follow poteto-mode's **Usage-limit fallback to Claude**: the other family that is not at limit, then `claudeAgent/claude-fable-5-2 effort=high`, then `claude-code-200k/claude-opus-5-5 effort=high`. Retire the limited reviewer's run as that rule says. A Hub reviewer override (`Cross-family review model override`) is a preference: when its provider is at limit, the same fallback applies. Report the review outcome, with the models that really ran:
   - `meos job review --result clean --rounds <n> --model "<models>"`
   - or `meos job review --result open --rounds <n> --finding "<finding>" --model "<models>"`
3. **PR & Verifying:** Run `meos job stage verifying`.
   - **Direct route:** Open the pull request to the default branch per `playbooks/opening-a-pr.md` BEFORE verification so evidence attaches to the PR head. **Never merge on the direct route** — the Runner merges after Gatekeeper/Owner approval.
   - **Staging route:** Follow the `stage` skill: open a Staging PR to `staging` and merge ONLY that Staging PR into `staging`. **Never merge to `main` and never run `promote`** until a `Release:` message arrives in this thread.
   - Dispatch an independent non-author verifier on `codex/gpt-6.1-terra reasoningEffort=high` (`runtimeMode: "full-access"`). When Codex is at its usage limit, or the Verifier's task fails on one, run the Verifier on `claude-code-200k/claude-sonnet-5-5 effort=high` at once, per poteto-mode's **Usage-limit fallback to Claude**. The Owner allows this switch. Never wait for the reset and never schedule a retry for it. Retire the limited Verifier's run (cancel, queue, archive) as that rule says, and name the model that ran in the evidence and the gate reason. Resolve the repo's verification skill in order: `<repo>/.agents/skills/verify-*/SKILL.md`, `<repo>/.claude/skills/verify-*/SKILL.md`, then `~/.claude/skills/verify-<repo-name>/SKILL.md`. For browser/UI checks, use **Stagehand v3** (`/home/ram-laguitan/.local/share/meos-stagehand/stagehand-qa.cjs`, `deepseek-v4.1-flash`, never Playwright), always offloading Docker, dev servers, Chromium, and heavy test suites to a `vmpool` devbox via `vm-offload` whenever the repo has a boxes.dev project. Record evidence with `meos job evidence --head <sha> --file <evidence.json>` when available, and post `PASS`, `PASS+NOTES`, or `FAIL` with real evidence on the PR.
4. **Gatekeeper (quiet on the Hub board):** Pstack's independent verifier verdict IS the Gatekeeper.
   - Clean `PASS` with zero human-only (`Person:`) checks and no unverified production migrations:
     `meos job gate --result confident --reason "<concise evidence summary>"`
   - `PASS+NOTES`, `FAIL`, open P1, or any human-only step:
     `meos job gate --result not-confident --reason "<why>" [--check "<human check>"] [--p1-blocks]`
   - Recording `meos job gate` moves the Job to `PR` quietly on the Factory Board (no Inbox/Telegram noise). Settle this thread with `t3_thread_organize` (`action: "settle"`, no `threadId`) and end your turn.
5. **Release turn (staging route ONLY, when a `Release:` message arrives later):**
   - Run the `promote` skill's "Release from a Job's thread" section for the single Staging PR named in the `Release:` message.
   - Report the outcome:
     - `meos job release synced`
     - `meos job release reverted --reason "<why>"`
     - `meos job release rebuild --reason "<why>"`
     - `meos job release stopped --reason "<why>"`

## 3. Blockers (the ONLY time to alert the Hub Inbox)

If you hit a genuine blocker that needs the Owner (ambiguous requirement, missing credential/access, or unrecoverable failure after retries):
- Ask via `meos job ask "<concise blocker summary and what you need>" --option "<A>" --option "<B>" --recommended 1` (never use `AskUserQuestion` in an unattended Factory thread), OR on a hard dead-end run `meos job stage failed --note "<reason>"`.
- This posts a one-way blocker alert in Hub Inbox and Telegram with a direct `Open in T3` link. Stop and end the turn.

## 4. No change needed

If the issue is already fixed or a duplicate before any PR is opened, comment/close cleanly on GitHub as the Owner, run `meos job close --reason "<one-line reason>"`, and end the turn.
