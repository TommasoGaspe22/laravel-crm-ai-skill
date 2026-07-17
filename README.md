# laravel-crm-ai-skill

An **AI agent skill** (for Claude Code, or any other AI coding assistant) that is a complete, reverse-engineered build framework for adding an **enterprise-style CRM sales flow** (Lead → Account → Contact → Opportunity, with stages, saved views, conversion, activities) to **any Laravel app**.

It's not a package or a starter kit you `composer require`. It's a **detailed specification** — data model, UX anatomy, component catalog, TDD test plan, phased roadmap — meant to be fed to an AI coding assistant so it can implement the CRM in your own codebase without reinventing the key product/architecture decisions.

Validated end-to-end: following this framework in a fresh Laravel 13 app (service extraction, additive migrations, list views, saved views, quick actions, and a transactional lead-conversion flow) produced 27 passing tests and a working UI on the first real implementation pass.

## What's inside

- **`SKILL.md`** — entry point: overview, guardrails, reuse map, phased build order.
- **`references/`** — ~45 detail files: UX anatomy captured from a real enterprise CRM reference org, full object catalog (Account/Contact/Opportunity/Activities), data model & migrations, component catalog (Blade + Alpine + Tailwind), permissions, testing strategy, V1/V2/V3 roadmap.
- **`HOW-TO-USE-THIS-SKILL.md`** — practical usage guide.

## Using it with Claude Code

Drop this folder into your project's `.claude/skills/` directory (or your global `~/.claude/skills/`):

```bash
git clone https://github.com/TommasoGaspe22/laravel-crm-ai-skill.git .claude/skills/laravel-crm-ai-skill
```

Claude Code auto-discovers it via the `name`/`description` frontmatter in `SKILL.md` and invokes it automatically when you ask for CRM-related work (list views, opportunity pipeline, lead conversion, etc.), or explicitly with `/laravel-crm-ai-skill`.

## Using it with any other AI assistant

The `SKILL.md` frontmatter is a Claude Code convention for auto-invocation — other tools won't discover it on their own. But every file in here is plain Markdown with no Claude-specific instructions inside the content itself, so any AI coding assistant (ChatGPT/Codex, Cursor, GitHub Copilot Chat, Gemini, Windsurf, a raw API call, ...) can use it just as well:

1. Clone or download this repo.
2. Point your assistant at the folder, or paste the relevant files into its context — start with `SKILL.md`, then `references/README.md` for the full file index and reading order.
3. Ask it to follow the build order in `SKILL.md` §"Build order" for your own Laravel app.

Nothing in here calls a Claude-only tool or API; it's a specification document, not code that executes.

## Prerequisites

Read `SKILL.md` §"Prerequisites (Day 0)" first. This is a build framework, not a scaffolder: it assumes your app already has an operational lead/pipeline table, a dashboard shell with auth, and basic roles. If you're starting from a blank Laravel app, build (or stub) that layer first.

## License

MIT — see `LICENSE`. No proprietary data, branding, or assets from any real product are included; only patterns, UX anatomy, and architecture were captured and re-described from scratch.
