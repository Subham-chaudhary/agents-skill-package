---
name: chalkdoor-new-project
description: Start a new Chalkdoor app from scratch. Covers the one-round question list, prerequisites, scaffolding Next.js with Bun next to chalkdoor_agent/, applying the standard config, the first commits, and the feature-by-feature build loop. Use when state.yaml says segment new, or when the user wants to start a fresh app.
---

# New project

Prerequisite: bootstrap is done. `chalkdoor_agent/` (with `scratch/`) exists and is git-ignored, and `github.mode` is known.

## 1. Ask once

Send all of these in one message, each with a default, and record the answers in `state.yaml`:

1. What does the app do, in one sentence? Who uses it?
2. Main screens or pages? (Default: a landing page plus a signed-in dashboard.)
3. Do users sign in? (Default: yes, email sign-in with Supabase Auth.)
4. Does it store data or files? (Default: Supabase.)
5. Does it take payments? (Default: no. If yes, Stripe Checkout.)
6. Do you have a design, template or brand? (Default: shadcn/ui.)
7. App name, used for the folder, the repo and `package.json`.

## 2. Prerequisites

Check each tool. If one is missing, give the user the install command:

- **Node.js 20.9+** (Next.js runs on it). Check: `node --version`
  - Windows: `winget install OpenJS.NodeJS.LTS`
  - macOS: `brew install node`
- **Bun.** Check: `bun --version`
  - Windows: `powershell -c "irm bun.sh/install.ps1|iex"`
  - macOS/Linux: `curl -fsSL https://bun.sh/install | bash`
- **git, and gh in `full` mode:** see `skills/github` § Setup.

## 3. Scaffold

`create-next-app` refuses a folder that already contains `chalkdoor_agent/`, so scaffold into `scratch/` and then move the files up:

```bash
bunx create-next-app@latest chalkdoor_agent/scratch/scaffold --ts --tailwind --eslint --app --src-dir --import-alias "@/*" --use-bun --skip-install --disable-git --yes
```

Move the files into the project root. Merge `.gitignore`, and keep any file that already exists (for example the user's README):

```bash
# bash / zsh
cat chalkdoor_agent/scratch/scaffold/.gitignore >> .gitignore && rm chalkdoor_agent/scratch/scaffold/.gitignore
(shopt -s dotglob; mv -n chalkdoor_agent/scratch/scaffold/* .) && rm -rf chalkdoor_agent/scratch/scaffold
```

```powershell
# PowerShell
Get-Content chalkdoor_agent/scratch/scaffold/.gitignore | Add-Content .gitignore; Remove-Item chalkdoor_agent/scratch/scaffold/.gitignore
Get-ChildItem -Force chalkdoor_agent/scratch/scaffold | Where-Object { -not (Test-Path $_.Name) } | Move-Item -Destination .
Remove-Item -Recurse -Force chalkdoor_agent/scratch/scaffold
```

Then:

1. Set `"name"` in `package.json` to the app name (the scaffold names it `scaffold`).
2. `bun install`
3. TypeScript: follow the setup in `skills/nextjs-standard/SKILL.md` § Versions.

## 4. Apply the standard

1. `.gitignore`, `.gitattributes`, `.env.example`: follow `skills/git-workflow` § Before the first commit. Remove duplicate lines after the merge.
2. `next.config.ts`: add `output: "standalone"` and keep the generated `cacheComponents` and `partialPrefetching` settings (`skills/nextjs-standard/references/scripts.md`).
3. Scripts: merge `templates/project/package.scripts.json`.
4. `tsconfig.json`: add `"chalkdoor_agent"` to `exclude`. `eslint.config.mjs`: add `"chalkdoor_agent/**"` to `globalIgnores`.
5. Add the health route (`skills/nextjs-standard/references/structure.md`). Create other folders only when the first file needs them.
6. Keep the generated `AGENTS.md` (the Next.js docs block) and append the Chalkdoor pointer below it (`skills/bootstrap` step 6). Add the README line.
7. Design system: `skills/ui-ux`. Integrations from the answers: `skills/integrations`.

## 5. Verify, then make the first commits

1. Run `bun run check`, then `bun run start` and open `/api/health`, which must return `{"status":"ok"}`. Report the results.
2. In `local` or `full` mode: `git init -b main` if needed, then commit:
   - `chore: scaffold Next.js app with Bun` (the scaffold and the standard config)
   - `chore: add Chalkdoor agent pointers` (`AGENTS.md`, `CLAUDE.md`, README line)
3. In `full` mode: create the private repo and push (`skills/github` § Setup, step 5).
4. Update `state.yaml`: `stack`, `conventions`, `checks`, `github.remote`. `segment` stays `new` until the first feature merges, then becomes `standard`.

## 6. Build loop (per feature)

1. Add it to `state.yaml` `features` (`planned`) and confirm the scope in one line with the user.
2. Create a branch `feat/<slug>`.
3. Build the smallest vertical slice: the screen, then the Server Action or query, then the data. Follow `skills/nextjs-standard` and `skills/ui-ux`.
4. Run the done check (`skills/nextjs-standard` § Done check), then commit per `skills/git-workflow`.
5. Merge (a PR in `full` mode, a local merge in `local` mode) and mark the feature `done`.
6. Independent features may run in parallel: see `skills/agents`.

Tell the user after each feature what now works, how to try it (`bun run dev`, then the URL), and the one or two best next steps.
