---
name: chalkdoor-github
description: Set up and manage GitHub for a Chalkdoor project. Covers asking the user how version control should be handled, GitHub account and gh CLI setup, git identity, private-by-default repo creation, pull requests, and strict rules for visibility and destructive operations. Use before the first push, when creating a repo, or when the user asks about GitHub.
---

# GitHub

## Control modes

Ask once, in the first question round, and record the answer in `state.yaml` as `github.mode`. The user can switch modes at any time.

> How do you want version control handled?
> 1. **Full control (recommended).** I set up GitHub, create a private repo, commit, push and open pull requests.
> 2. **Local only.** I commit and manage branches on your computer. You push and manage GitHub yourself.
> 3. **Self-managed.** You run git yourself. I tell you when to commit and suggest the message.

Then tell the user in 2–4 lines what you will do and what you will skip in that mode:

| | `full` | `local` | `self` |
|---|---|---|---|
| `.gitignore`, `.gitattributes`, secret checks | do | do | do, then tell the user |
| `git init`, branches, commits | do | do | suggest only |
| Account, `gh` install and auth | guide the user | skip | skip |
| Create repo, push, PRs | do | tell the user the command | skip |
| Read-only git (`status`, `diff`, `log`) | do | do | do |

In `self` mode, at each checkpoint say: "Good point to commit." Then give the files and the message from `skills/git-workflow`.

## Setup (full mode)

Check what exists first and skip the steps that are done. Record `github.account_ready` and `github.gh_authenticated`.

1. **Account.** If the user has none, explain the steps. The agent cannot create the account.
   - Sign up at https://github.com/signup and verify the email.
   - Turn on two-factor authentication (Settings → Password and authentication).
   - Optional: Settings → Emails → "Keep my email addresses private".
2. **Install git and gh.** Check with `git --version` and `gh --version`.
   - Windows: `winget install --id Git.Git -e` and `winget install --id GitHub.cli -e`
   - macOS: `brew install git gh`
   - Linux: the distro's git package, and gh from https://github.com/cli/cli#installation
3. **Log in.** The login is interactive, so the user runs it. In Claude Code they type `! gh auth login --web --git-protocol https`. Then run `gh auth setup-git` and check with `gh auth status`.
4. **Git identity.** Tell the user before you change global config.
   - `git config --global user.name "<name>"`
   - Private email: `gh api user --jq '"\(.id)+\(.login)@users.noreply.github.com"'`, then `git config --global user.email "<that>"`
   - `git config --global init.defaultBranch main`
   - Windows only: `git config --global core.longpaths true`. Without it, git fails with "Filename too long" on deep `node_modules` paths, for example when removing a worktree.
5. **Repo.**
   - New: `gh repo create <name> --private --source . --remote origin --push`
   - Existing remote: check that it belongs to the user with `gh repo view --json owner,visibility`.
   - Record `github.remote` and `github.visibility`.

## Daily flow (full mode)

- Do feature work on a branch (naming rules in `skills/git-workflow`).
- Push with `git push -u origin <branch>`. Open a PR with `gh pr create --fill`.
- Merge after the checks pass: `gh pr merge --merge --delete-branch`. This keeps the commit history. Then `git switch main && git pull --ff-only`.
- Small docs or config fixes may go straight to `main` if the build still passes.

## Strict rules (all modes)

1. **Private by default.** Every repo is created with `--private`.
2. **Public only with a double confirmation.** The user must ask for it explicitly, and then confirm twice in two separate messages:
   - First: "Making `<repo>` public means anyone can see all code and the full commit history. Make it public?"
   - Before the second confirmation, scan the full history for secrets (`skills/git-workflow` § Secrets) and report the result.
   - Second: "Final check: reply `make <repo> public` to confirm."
   - Then run `gh repo edit <owner>/<repo> --visibility public --accept-visibility-change-consequences`. The same rule applies to any visibility change and to forks of private code.
3. **Never** delete or transfer a repo, change collaborators, branch protection, Actions secrets or repo settings, or delete releases or tags, unless the user asks for that exact action.
4. **Never force-push the default branch.** On an agent-owned feature branch, use `--force-with-lease` only. Rewriting history: see `skills/git-workflow` § History.
5. **Never print or store tokens.** Do not run `gh auth token`. Do not write tokens to files, remotes or logs.
6. **Only push to repos the user owns.** For other repos, fork and open a PR, and only when asked.
7. **Never skip hooks** (`--no-verify`) or signing unless the user asks.

## Answering GitHub questions

Use plain words first, then the command, then one line on what it changes. For example: "A branch is a separate copy of the work where you can try things safely. `git switch -c feat/login` creates one and moves you onto it."
