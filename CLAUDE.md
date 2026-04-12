# cctop, Claude Code Sessions Dashboard

A live terminal dashboard for monitoring all your Claude Code sessions at a glance. Like `htop`, but for Claude Code.

## Why

Power users run multiple Claude Code sessions simultaneously, one refactoring a module, another writing tests, a third researching an API. You end up tab-switching between terminals just to check "is it done yet?" or "is it stuck waiting for me?" There's no central place to see what's happening across sessions.

## What It Does

Installs a lightweight hook into Claude Code that tracks session activity in real time. A companion TUI dashboard (`cctop`) displays all active sessions in a single live-updating table:

- **Status**, see at a glance whether each session is idle (waiting for you), thinking, editing files, running commands, searching the web, or spawning subagents
- **Project & branch**, know which codebase and branch each session is working in
- **Context usage**, monitor how much of the context window has been consumed, so you can wrap up or start fresh before hitting limits
- **Tool count**, track how many tool calls a session has made
- **Model**, which Claude model each session is using
- **Last messages**, peek at the most recent user prompt and Claude response without switching terminals

Sessions that go quiet for 1+ hour are marked stale. Sessions that end clean up after themselves automatically.

## Who It's For

Anyone running more than one Claude Code session at a time, or anyone who wants a quick overview of what's happening without context-switching into each terminal.

## Project Structure

- `plugin/`, distribution files (only this directory gets installed)
  - `plugin/scripts/cctop-hook.sh`, hook handler, writes `~/.cctop/<session-id>.json`
  - `plugin/scripts/cctop_dashboard.py`, Textual TUI app (run with `uv run --script`)
  - `plugin/scripts/cctop-poller.py`, background transcript poller
  - `plugin/scripts/launch-cctop.sh`, convenience launcher
  - `plugin/hooks/hooks.json`, registers the hook for 7 events
  - `plugin/.claude-plugin/plugin.json`, plugin manifest
- `.claude-plugin/marketplace.json`, local marketplace manifest (points to `./plugin/`)
- `tests/test_cctop_dashboard.py`, TUI tests
- `install.sh`, reinstalls plugin into Claude's cache
- `plans/`, gitignored, PRDs and design docs (never commit these)
- `BACKLOG.md`, numbered feature backlog with completion tracking

## Reference Docs

The `reference/` directory contains Claude Code internals documentation, split by topic. **Read these on-demand**, don't load them all upfront, just read the one relevant to your current task:

| File | When to read |
|---|---|
| `reference/hooks-api.md` | Writing or debugging hooks, events, stdin fields, output format |
| `reference/transcript-format.md` | Parsing JSONL transcripts, entry types, field shapes, path encoding |
| `reference/sessions-index.md` | Reading the sessions index, schema, customTitle timing |
| `reference/plugin-system.md` | Plugin install/dev workflow, manifests, cache, gotchas |
| `reference/session-data-files.md` | Tool counts and session-status JSON files |

## Installing After Changes

The plugin runs from a **copy** in `~/.claude/plugins/cache/`, not from this directory.
After editing any file under `plugin/`, you **must** reinstall:

```bash
./install.sh --dev
```

**Always run `./install.sh --dev` after modifying any plugin file** (hooks, scripts, manifests). New Claude sessions will pick up the changes; existing sessions keep the old version.

## Releasing

Use `release.sh` for version bumps and tagging. The script handles the mechanical parts, you write the changelog.

1. `./release.sh bump <version>` — updates `plugin.json`, prints git log since last tag
2. Read the git log output and write a human-readable `CHANGELOG.md` entry. Summarize what changed from the user's perspective (new features, bug fixes, improvements). Prepend the new entry to the file. Use format: `## vX.Y.Z — YYYY-MM-DD`
3. `git add plugin/.claude-plugin/plugin.json CHANGELOG.md`
4. `./release.sh tag` — commits, tags, pushes

## Writing Style

- Use commas instead of emdashes (—) in prose

## Security

Before committing, run a basic security audit on staged changes:
- No hardcoded secrets, API keys, tokens, or passwords
- No personal information (real names, private emails, internal hostnames, private IPs)
- No SentinelOne internal references (GHE URLs, internal tooling, team names)
- TruffleHog runs as a pre-commit hook, but also manually sanity-check diffs for anything it might miss

## Branching

- **ALWAYS use a worktree when starting work on a new branch.** Use the `EnterWorktree` tool to create an isolated worktree before making any changes. Do NOT just create a branch with `git checkout -b` or `git switch -c` in the main working directory.
- This keeps the main working directory clean on `main` and avoids conflicts with other sessions.
- When the work is done and merged, exit the worktree with `ExitWorktree`.

## GitHub CLI Gotchas

- This repo's remote is `github.com`, but `GH_HOST` may be set to GHE. Always prefix `gh` commands with `GH_HOST=github.com` (e.g. `GH_HOST=github.com gh pr create ...`).
- When merging PRs from a worktree, do NOT use `--delete-branch` on `gh pr merge`, it tries to checkout main locally which fails because main is already checked out in the main worktree. Just `gh pr merge N --squash`, then exit the worktree normally with `ExitWorktree` which handles branch cleanup.

## PR Groups Workflow (MANDATORY)

This project uses a structured PR-groups workflow defined in `plans/pr-groups.md`. **This workflow is not optional.** When asked to work on a PR group or backlog items:

1. Read `plans/pr-groups.md` first to understand the grouping and dependencies
2. Follow the workflow steps exactly as written in that file (branch → plan → implement → test → push & PR)
3. Use `EnterWorktree` to create the worktree, do not skip this step
4. Enter plan mode before implementing, get user approval before writing code
5. Make granular commits (one per logical change)
6. After merge, update both `BACKLOG.md` (mark items done) and `plans/pr-groups.md` (check off the PR group)

## Commits

- Split uncommitted changes into logical, self-contained commits (e.g. separate feature code, tests, docs, backlog updates)
- When moving or renaming files, update all references in other files (BACKLOG.md links, CLAUDE.md structure, README, etc.) in the same or immediately following commit

## Docs Hygiene

When making changes that affect user-visible behavior (new features, changed columns, new keybindings, install steps, usage), always check that `README.md`, `BACKLOG.md`, and `CONTRIBUTING.md` are updated to match.


<!-- KARPATHY_DISCIPLINE_BEGIN v1.0 -->
## Behavioral Discipline (Karpathy Guidelines)

> Adapted from [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) · MIT License · ~14k★ · Karpathy-starred.
> Adopted by Everest Capital 2026-04-12. This section is **complementary** to the existing HONESTY PROTOCOL, PAIRING RULE, COST DISCIPLINE, and CLI-ANYTHING mandates above — it does not replace them.

**Tradeoff posture:** These guidelines bias toward caution over speed. For trivial tasks (typo fix, one-line config), use judgment and skip the ceremony.

### K1. Think Before Coding *(reinforces HONESTY PROTOCOL)*

Don't assume. Don't hide confusion. Surface tradeoffs.

- State assumptions explicitly. If uncertain, label as `INFERRED` per HONESTY PROTOCOL.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

**Everest delta:** when an assumption is surfaced, it must carry a `VERIFIED / UNTESTED / INFERRED` tag. Wrong `VERIFIED` = 3× penalty to honesty_violations table.

### K2. Simplicity First *(complements XGBoost efficiency cap)*

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and 50 would do, rewrite.

Ask: "Would a senior engineer call this overcomplicated?" If yes, simplify.

**Everest delta:** this is per-diff. XGBoost efficiency (90 min/chat, max 3 chats/task) is per-session. Both apply.

### K3. Surgical Changes *(NEW — closes AUTOLOOP evolver bloat gap)*

Touch only what you must. Clean up only your own mess.

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, **mention it — don't delete it.**

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless explicitly asked.

**The test:** every changed line must trace directly to the user's request.

**Everest delta — AUTOLOOP V2 evolver constraint:** prompt/rule updates produced by the evolver must be **minimal and surgical**. Diffs that exceed 20% line growth or touch sections unrelated to the failing case must be rejected by the evolver's self-check and re-attempted with a narrower edit. This closes the bloat failure mode flagged by Dylan Cleppe's extraction-funnel analysis (2026-04-12) and by Karpathy directly.

### K4. Goal-Driven Execution *(complements EG14 gate)*

Define success criteria. Loop until verified.

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

**Everest delta:** for SUMMIT dispatches touching production (zonewise-web, dify-zonewise, nexus), the EG14 14-point enterprise gate is the canonical success criteria. Goal-driven execution at the sub-task level must compose up to an EG14 verdict, not replace it.

### Working indicators

These guidelines are working if:
- Fewer unnecessary changes appear in diffs.
- Fewer rewrites happen due to overcomplication.
- Clarifying questions arrive *before* implementation, not after mistakes.
- AUTOLOOP evolver prompt diffs stay small and targeted.

### Attribution

Source: https://github.com/forrestchang/andrej-karpathy-skills (MIT)
Upstream quote from Karpathy: *"LLMs are exceptionally good at looping until they meet specific goals. Don't tell it what to do, give it success criteria and watch it go."*
<!-- KARPATHY_DISCIPLINE_END v1.0 -->
