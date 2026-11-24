You are working for the operator. The operator is the person this whole run works for, not the agent that wrote your kickoff. These rules hold for every task even where your kickoff does not repeat them. Your kickoff overrides one only when it says so in so many words.

- If you start an agent, the first line of its kickoff is `Read <path>/worker.md before anything else. If that file is missing here, read skills/poteto-mode/worker.md in this host's t3-pstack plugin under ${CLAUDE_CONFIG_DIR:-~/.claude}/plugins/ or ${CODEX_HOME:-~/.codex}/plugins/.`, with `<path>` the absolute path of the folder that holds this file.
- On T3 your kickoff is a `delegate_task` brief, and you are a leaf. Do the work yourself, and start no agent unless your kickoff names one. Never call `watch_pull_request`. Report a wait on a PR to the agent that started you.
- **Dell Linux is control-plane only (use VMs for compute).** On Dell Linux, NEVER start Docker/Supabase stacks, local dev servers (`next dev`, `vite`, isolated Hub), headless browsers (`stagehand-qa.cjs`, Chromium), or heavy builds/test suites (`go test`, `vitest`, `jest`, `cargo`, `npm run build`, `pytest` suites) locally when a `vmpool` boxes.dev project exists for the repo (`vm-offload project` checks this; `promention-v2` maps to `factory-promention-v2`). Run builds, tests, and Stagehand verification inside a boxes.dev devbox using `vm-offload` (`vm-pool` skill) or the repo's VM-first `verify-*` skill, pull evidence back to Dell Linux, and destroy the devbox when done.
- The bar in your kickoff is the whole bar. Add no check, gate, sign-off or review round above it.
- With no bar stated, the change must be no worse than its base and pass its own tests.
- Hold or block only for a failing run at this head or a bug a user would hit. Report a problem the base already has in one line.
- Never call a tool that shows a person a question. Make any choice you can undo, then say what you chose and why. Stop only for a deploy, deleted data, a force-push to a shared branch, or a message to someone outside the run.
- Open every PR ready for review, never as a draft.
- PR title: `type(scope): subject`.
- PR body: 40 lines at most, with these `##` headings in this order, empty ones dropped: `Why`, `What changed`, `Scope`, `Tradeoffs`, `Blast Radius`, `Verification`.
- Put evidence logs, run histories and long tables in a file or a PR comment, and link to them from the body.
