---
name: chalkdoor-agents
description: How the main agent splits Chalkdoor work across subagents. Covers when to parallelize, ordered waves, one git worktree and branch per subagent, context isolation, token-efficient briefs, scratch docs, and the rule that only the main agent merges and resolves conflicts. Use before spawning any subagent or running work in parallel.
---

# Agents and parallel work

## When to use subagents

- **Use them** for independent tasks that touch different files: separate features, a broad read-only search, a review or test pass.
- **Do it yourself** when the task is small (a few files, one quick search), when tasks share the same files, or when the next step depends on this one.
- Run at most 3–4 subagents at a time. More parallel agents cost more tokens and produce more conflicts, not more speed.

## Work in ordered waves

| Wave | Who | What |
|---|---|---|
| 0. Foundations | Main agent, sequential | Shared contracts first: DB schema, shared types, env modules, auth, layout, design tokens. Commit them to `main`. |
| 1. Features | Subagents, in parallel | One feature each, in its own worktree and branch, with no overlapping files. |
| 2. Integrate | Main agent | Merge the branches one at a time, resolve conflicts, and run `bun run check` after each merge. |
| 3. Verify (optional) | Read-only subagents | Review, e2e tests and UX pass. They report findings; the main agent fixes them. |

Record waves and owners in `state.yaml` (`features`, `worktrees`).

## Worktrees (one per subagent branch)

```bash
git worktree add chalkdoor_agent/worktrees/<slug> -b feat/<slug> main
cp .env.local chalkdoor_agent/worktrees/<slug>/   # local copy only, never committed
cd chalkdoor_agent/worktrees/<slug> && bun install  # fast: Bun reuses its global cache
```

- `chalkdoor_agent/` is git-ignored and excluded from `tsconfig` and ESLint, so worktrees inside it do not leak into the main app's checks.
- A worktree has no `chalkdoor_agent/` of its own. Give subagents the absolute path to the main project's `chalkdoor_agent/package/` files they need.
- **Claude Code:** the Agent tool with `isolation: "worktree"` creates and removes the worktree for you. Keep its folder git-ignored (`.claude/worktrees/` is in the template).
- Clean up after merging: `git worktree remove chalkdoor_agent/worktrees/<slug>`, `git branch -d feat/<slug>`, `git worktree prune`.
- Windows: set `git config core.longpaths true` **before** you create worktrees. Otherwise the removal fails with "Filename too long" and leaves a half-deleted folder.

## Context isolation

- Each subagent knows only its own task. Do not give it the full conversation or other features' details.
- Exception: subagents that work on the **same** goal (for example research, then implementation) may share one notes file: `chalkdoor_agent/scratch/<task>/notes.md`.
- Subagent docs, notes and reports go in `chalkdoor_agent/scratch/<task>/` (git-ignored). Only the main agent decides whether anything from there gets committed.

## Brief template (keep it under about 15 lines)

```text
Goal: <one sentence, user-visible result>
Branch/worktree: feat/<slug> at <absolute path>
You own (may edit): <dirs/files>
Read-only: <shared files you may read but not change>
Read first: <absolute paths to the 1-2 Chalkdoor skill files needed>
Rules: commit on your branch per skills/git-workflow; never merge, rebase main, push main, or change shared files.
Done when: bun run lint && bun run typecheck pass; <feature check>
Return: <=10 lines: what changed, commits, checks run + results, open issues
```

## Token efficiency

- Pass **paths**, not file contents or skill text. The subagent reads only what it needs.
- Give a small `state.yaml` excerpt (stack and conventions), not the whole file.
- Ask for short, structured returns. Long logs go in `scratch/`, and the subagent returns the path.
- Send follow-ups to the same subagent (it keeps its context) instead of starting a new one.
- Use a smaller or faster model for read-only search and review when the agent tool lets you choose.
- Never have two subagents read the same large area. Do it once and share the summary.

## Merging and conflicts (main agent only)

1. Merge one branch at a time: `git switch main && git merge --no-ff feat/<slug>` (or `gh pr merge --merge` in `full` mode).
2. On a conflict, read both sides and keep both intents. Never resolve blindly with `-X ours` / `-X theirs`. If the intent is unclear, ask the user.
3. Run `bun run check` after each merge, and fix the problem before you merge the next branch.
4. Update `state.yaml` and clean up the worktree.
