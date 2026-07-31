# python-skill

A minimal [Agent Skill](https://agentskills.io) for Claude Code (and compatible runtimes) — just the essential best practices, no bloat. It sets an opinionated default stack for Python work: **uv, ruff, pyright, pydantic v2, httpx2, structlog, pytest** — plus naming rules, fail-loud error handling, and testing discipline.

> Might be a good skill that makes your life better. Might not. Hard to tell.

The skill triggers automatically whenever the agent writes or reviews Python code, starts a Python project, edits `pyproject.toml`, or configures linting/typing/testing. The stack applies only to greenfield projects — existing projects keep their current stack while still using the stack-agnostic naming, error-handling, async, and testing guidance.

## What's inside

- **Stack table** — one tool per concern, and a "don't use → use instead" mapping (pip → uv, black/flake8 → ruff ...)
- **Layout & commands** — `src/` layout, local vs CI command sets, ruff + pyright baseline config
- **Naming** — verb+noun functions, predicate booleans, banned vague words (`data`, `utils`, `manager`, …)
- **Rules** — type hints everywhere, pydantic at boundaries, no mutable defaults, exception chaining
- **Fail loud** — no swallowed exceptions, no fake fallback data, typed errors at entry points

Keeping it minimal is deliberate: skills work best when they stay small — best practices only, not an exhaustive manual.

## Install

Easiest — the [skills CLI](https://github.com/vercel-labs/skills) (works for Claude Code, Codex, Cursor and other agents):

globally, for all projects:

```bash
npx skills add akmalovaa/python-skill -g
```

into the current project:

```bash
npx skills add akmalovaa/python-skill
```

Or manually for Claude Code (personal skill, all projects):

```bash
git clone https://github.com/akmalovaa/python-skill ~/.claude/skills/python-skill
```

Or copy just the file:

```bash
mkdir -p ~/.claude/skills/python-skill
curl -fsSL -o ~/.claude/skills/python-skill/SKILL.md \
  https://raw.githubusercontent.com/akmalovaa/python-skill/main/SKILL.md
```

For a single project, put it in `<project>/.claude/skills/python-skill/` instead. Other runtimes that follow the Agent Skills spec (Codex, Copilot CLI, Gemini CLI) also read `~/.agents/skills/`.

## References

Used while creating this skill - [Anthropic — Skill Creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)

## Useful Python skills & projects

A collection of community skills and projects found while researching this one:

- [wondelai/skills](https://github.com/wondelai/skills) — `clean-code` / `code-craftsmanship` bundle: intention-revealing naming (classes are nouns, methods are verbs), function discipline, refactoring patterns, 0–10 code scoring. Language-agnostic; Python specifics live in the reference files.
- [wdm0006/python-skills](https://github.com/wdm0006/python-skills) — 14 focused Python skills (ruff/mypy config, CLI development, packaging, security audit, testing strategy) with clean progressive disclosure. Great fit for scripts, libraries, and SRE tooling.
- [manikosto/claude-code-python-stack](https://github.com/manikosto/claude-code-python-stack) — the densest backend coverage: `fastapi-patterns`, `sqlalchemy-patterns`, `pydantic-patterns`, `celery-patterns`, `docker-patterns` and 15 more. Cherry-pick the ones your stack uses; note it has no naming rules.
- [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) — 66 expert personas (`python-pro`, `fastapi-expert`, `sre-engineer`, `kubernetes-specialist`). Useful as a second opinion rather than a rule set.
- [obra/superpowers](https://github.com/obra/superpowers) — process skills (brainstorming, TDD, systematic debugging, skill testing with subagents). Covers *how to work*, not *what code to write* — complements stack skills like this one without overlap.
- [affaan-m/ECC](https://github.com/affaan-m/ECC) — 300+ skills in a full harness. Don't install wholesale (it rewrites your whole setup); extract individual skills. Its `coding-standards` is TypeScript-only.

Catalogs for finding more:

- [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) — the most active awesome-list
- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — broader than skills: hooks, statuslines, plugins
- [skills.sh](https://skills.sh) — skill search with install commands

## License

[MIT](LICENSE)
