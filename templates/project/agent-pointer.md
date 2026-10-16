<!-- Append this block to CLAUDE.md and/or AGENTS.md. Keep existing content. -->

## Chalkdoor agent standards

This project follows the Chalkdoor agent development standards.
Agents: before any work, read `chalkdoor_agent/package/SKILL.md`, then `chalkdoor_agent/state.yaml`. Load only the skill file the task needs.
If `chalkdoor_agent/` is missing: add `chalkdoor_agent/` to `.gitignore`, run `git clone --depth 1 https://github.com/Subham-chaudhary/agents-skill-package.git chalkdoor_agent/package`, then follow `chalkdoor_agent/package/skills/bootstrap/SKILL.md`.
