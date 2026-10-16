---
name: chalkdoor-git-workflow
description: Chalkdoor git rules for commits, commit messages, branches, .gitignore, .gitattributes, secrets, media and assets, history and repo hygiene. Use whenever you are about to commit, branch, merge or change ignore rules, or when the user asks a git question.
---

# Git workflow

These rules apply in every `github.mode`. In `self` mode, you suggest the commands and the user runs them.

## Before the first commit

1. **`.gitignore`:** copy or merge `templates/project/gitignore.txt` into the project `.gitignore`. Keep existing project lines. Order matters: `!.env.example` must come **after** `.env*`. Check with `git check-ignore -q .env.example || echo tracked`.
2. **`.gitattributes`:** copy `templates/project/gitattributes.txt`. It sets LF line endings, which avoids Windows/macOS diff noise, and marks binary files. In an existing repo, follow it with `git add --renormalize .` and commit the result as `chore: normalize line endings`.
3. **`.env.example`:** copy `templates/project/env.example`. It lists variable names only.
4. **Already-tracked files that should be ignored:** check `git ls-files -ci --exclude-standard`. Remove them from the index with `git rm --cached <path>`, which keeps the local file, and tell the user.

Set `conventions.gitignore` and `conventions.gitattributes` in `state.yaml`.

## Commit frequency

- Commit when one logical unit is done and the app still builds. Examples: a feature slice, a config change, a dependency change, one migration step.
- Group changes by context. If a session touched unrelated things, split them into one commit per logical unit (deps / config / feature / docs).
- Too often: a commit per file or per small edit. Too rarely: one commit per day or per session. A good rhythm is one commit per finished sub-task.

## Commit messages

Agents and developers both read the log, so keep the format strict:

```text
<type>(<optional scope>): <title>

<optional body: 1-3 short lines on why, not what>
```

- Types: `feat` `fix` `refactor` `style` `perf` `test` `docs` `build` `ci` `chore` `migrate`.
- Title: imperative, lower case, no period, 72 characters at most. Example: `feat(auth): add email sign-in with Supabase`.
- Add a body only when the reason is not obvious.

## Pre-commit check

Run this before every commit:

1. Check `git status --short` and `git diff --cached --stat`. Stage explicit paths. Use `git add -A` only after you have read the status.
2. Secret scan of the staged diff (see § Secrets).
3. Size check: no staged file over 1 MB unless it is approved (see § Assets).
4. Code changed → run `bun run lint && bun run typecheck`. Run `bun run build` before merging into `main`.

## Branches

- `main` must always build. Do feature work on branches.
- Names are lower-case kebab with a type prefix: `feat/<slug>`, `fix/<slug>`, `chore/<slug>`, `migrate/<slug>`.
- One branch per feature or task. Delete it after the merge. Record branches in `state.yaml` `features`.
- Merge conflicts are resolved by the main agent only (`skills/agents`).

## Secrets

- `.env*` is ignored. Only `.env.example` is committed, and it holds names only.
- Staged-diff scan: `git diff --cached -U0 | grep -nE '(sk_live_|sk_test_|rk_live_|whsec_|sb_secret_|service_role|ghp_|github_pat_|AKIA[0-9A-Z]{16}|-----BEGIN [A-Z ]*PRIVATE KEY|postgres(ql)?://[^ ]*:[^ ]*@)'`. If anything matches, stop and do not commit.
- Full-history scan (before a repo goes public): `git log -p --all | grep -cE '<same pattern>'`. Report the count only, never the matched values.
- If a secret was committed: stop and tell the user. They must rotate the secret at the provider first. Then remove it from the code. Rewrite history only with approval (§ History). Rotating is the real fix, because a pushed secret is already exposed.

## Assets and repo size

- Keep the repo small. Before committing images, compress them and prefer SVG, WebP or AVIF. Fonts go through `next/font`.
- Do not commit videos, audio, archives, design source files (`.psd`, `.fig`) or datasets. Put them in storage (for example Supabase Storage) and reference them by URL.
- Files over 1 MB need the user's approval. GitHub rejects files over 100 MB. Use Git LFS only if the user asks for it.
- Never commit build output (`.next/`, `out/`), `node_modules/`, coverage or logs.

## History

- Never rewrite commits that are already pushed unless the user asks. Add new commits on top.
- If the user does ask: create `backup/<branch>-<yyyymmdd>` first, confirm the exact plan, and push with `--force-with-lease`. For the default branch, also follow the double confirmation in `skills/github`.
- Never run `git reset --hard`, `git clean -fd` or `git checkout -- .` on work the user has not committed.

## Other repo rules

- One lockfile: `bun.lock`. Delete `package-lock.json`, `yarn.lock` and `pnpm-lock.yaml` in a `build:` commit.
- Keep the README short and current. Add a LICENSE only if the user chooses one.
- `chalkdoor_agent/` and agent scratch docs are never committed.
- Tag releases `vMAJOR.MINOR.PATCH` when the user ships a version.

## Answering git questions

Use plain words, then the command, then what it changes. Do not lecture. Offer the next step.
