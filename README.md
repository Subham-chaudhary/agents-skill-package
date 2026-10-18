# Chalkdoor agent

**v0.3.0. Agent skills that build new apps, or migrate existing vibe-coded apps, to the Chalkdoor development standard.**

Vibe-coded apps are rarely uniform. Framework, git habits, structure and third-party services differ from app to app. This package gives a coding agent one standard to work to:

- **Framework:** one Next.js server (App Router, TypeScript), Bun, Turbopack.
- **Version control:** git and GitHub, private repos by default, small clear commits.
- **Integrations:** Supabase by default, plus Neon, Stripe, Clerk and Better Auth.
- **UI/UX:** community design systems, and the user's own design always comes first.
- **Agents:** subagents in parallel on git worktrees, with a small token footprint.

It covers two cases. **New** means the vibe coder starts from scratch. **Migration** means they already have an app and want to bring it up to the standard.

## Use it in a project

Paste this to your coding agent, from inside your project folder:

> Set up the Chalkdoor agent standards in this project. First add `chalkdoor_agent/` to `.gitignore`. Then run `git clone --depth 1 https://github.com/Subham-chaudhary/agents-skill-package.git chalkdoor_agent/package`. Then read `chalkdoor_agent/package/SKILL.md` and follow it.

The agent then:

1. keeps everything it needs in `chalkdoor_agent/` (git-ignored, local only),
2. asks how you want GitHub handled: by you, by the agent locally, or fully by the agent,
3. detects whether the project is new or a migration, and asks the remaining questions in one go,
4. adds a short pointer to `CLAUDE.md` / `AGENTS.md` and one line to your README.

To update the package later: `git -C chalkdoor_agent/package pull --ff-only`.

## Layout

```text
SKILL.md                  # router: always read first, kept small
skills/
  bootstrap/              # install into a project, state files, pointers
  github/                 # account, gh, control modes, visibility rules
  git-workflow/           # commits, branches, .gitignore, secrets, assets
  new-project/            # build from scratch
  migration/              # bring an existing app to the standard
  nextjs-standard/        # framework, Bun, rendering, structure, backend
  ui-ux/                  # design systems and UX rules
  integrations/           # Supabase, Neon, Stripe, Clerk, Better Auth
  agents/                 # subagents, worktrees, token efficiency
templates/
  chalkdoor_agent/        # state files copied into the project
  project/                # .gitignore, .gitattributes, scripts, pointers
  migration/              # assessment, plan and validation reports
research/                 # background research (not loaded by agents)
```

Each skill is one file. The agent loads only the one it needs, and opens reference files only when needed.

## Limitation

A skill guides an agent. It does not enforce anything at runtime. The Chalkdoor platform must still enforce isolation, secret injection, resource limits and deployment permissions.
