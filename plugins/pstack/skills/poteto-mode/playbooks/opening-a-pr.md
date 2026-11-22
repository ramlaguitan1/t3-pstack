### Opening a PR

Invoked at the end of every other playbook.

Work from a git worktree off trunk, and keep unrelated work out of it. Rebase into small, landable commits. Run `/deslop` over the diff before commit. Before you commit, merge, or deploy from a worktree, stop every live agent that holds it with `task_cancel`, which also stops the tasks it delegated, and confirm each stop with `task_status`. A delegate's children do not inherit its brief, so a read-only instruction never reaches them. Then run `git status` and read the tree you are about to ship.

Title: `type(scope): subject`, naming a real symbol when one carries the change. Body: a one-minute briefing under 40 lines with these `##` headings in order, dropping empty ones: `## Why`, `## What changed`, `## Scope`, `## Tradeoffs` (only rejected alternatives a reviewer would otherwise ask about), `## Blast Radius`, `## Verification`.

Stack follow-ups by base branch. Open PRs ready, through the run's PR tool with `draft: false`. Without one, use `gh pr create` without `--draft`, or `origin pr create --status open` when Origin's CLI resolves the repository. Opening a PR does not start a babysit.

**Staging repositories (MEOS override):** If the repository uses a staging branch (per `~/.claude/skills/stage/references/staging-repos.md` or `Route: staging` in the launch message), follow the `stage` skill: open the Staging PR targeting `staging`, never target `main` directly, and merge into `staging` for staging verification before any promotion. Never add AI disclaimers or `Co-Authored-By` footers to commits, PRs, or comments.
