---
name: webnovel-codex
description: Codex compatibility layer for the Webnovel Writer plugin. Use whenever the user asks for webnovel-writer, Webnovel Writer, webnovel init/plan/write/review/query/learn/dashboard/doctor, or uses Claude-style commands such as /webnovel-init, /webnovel-plan, /webnovel-write, /webnovel-review, /webnovel-query, /webnovel-learn, /webnovel-dashboard, or /webnovel-doctor in Codex.
---

# Webnovel Writer For Codex

This plugin was originally authored for Claude Code. In Codex, preserve the original creative workflow, but translate Claude-specific assumptions before acting.

## Command Mapping

Treat these Claude slash commands as Codex natural-language intents:

| User intent | Codex skill/workflow |
| --- | --- |
| `/webnovel-init` | Use `webnovel-init` to initialize a new book project. |
| `/webnovel-plan N` | Use `webnovel-plan` to plan volume `N`. |
| `/webnovel-write N` | Use `webnovel-write` to write chapter `N`. |
| `/webnovel-review N` or `N-M` | Use `webnovel-review` to review the chapter or range. |
| `/webnovel-query ...` | Use `webnovel-query` for read-only story-state queries. |
| `/webnovel-learn ...` | Use `webnovel-learn` to store reusable writing lessons. |
| `/webnovel-dashboard` | Use `webnovel-dashboard` to start the read-only dashboard. |
| `/webnovel-doctor` | Use `webnovel-doctor` for read-only diagnostics. |

## Codex Environment Mapping

When a referenced Webnovel Writer skill says:

- `CLAUDE_PLUGIN_ROOT`, use the installed plugin root, meaning the `plugins/webnovel-writer` directory after installation.
- `CLAUDE_PROJECT_DIR`, use the current Codex workspace root or the explicitly named book project root.
- `SCRIPTS_DIR`, use `<plugin-root>\scripts`.
- `SKILL_ROOT`, use `<plugin-root>\skills\<skill-name>`.
- `AskUserQuestion`, ask the user directly in chat when the decision is required. In default Codex mode, make a reasonable assumption for non-risky details.
- `Agent tool`, use Codex's available subagent or task-decomposition tools if available; otherwise perform the step directly while preserving the required output shape.
- `WebSearch` or `WebFetch`, use browsing only when the user asks for current market/platform information or when a time-sensitive fact must be verified.

Use PowerShell-compatible commands on Windows. A reliable command shape from a cloned repository is:

```powershell
$env:WEBNOVEL_CODEX_PLUGIN_ROOT = (Resolve-Path ".\plugins\webnovel-writer").Path
$env:SCRIPTS_DIR = "$env:WEBNOVEL_CODEX_PLUGIN_ROOT\scripts"
python -X utf8 "$env:SCRIPTS_DIR\webnovel.py" --project-root "<PROJECT_ROOT>" <subcommand>
```

The Python runtime still understands `.claude/.webnovel-current-project` pointers and the legacy `CLAUDE_*` environment variables. Do not create new project files under the plugin root.

## Operating Rules

- Before using a specific workflow, read that workflow's `SKILL.md` completely and follow its reference-loading rules.
- Keep all author-facing outputs in Chinese unless the user asks otherwise.
- Prefer the plugin's `scripts/webnovel.py` CLI for project initialization, diagnostics, state updates, reviews, commits, memory, and dashboard startup.
- Keep `dashboard/`, `templates/`, `references/`, `agents/`, and `scripts/` as plugin resources, not user project content.
- For commands that write story canon, confirm required creative choices before writing.
- For read-only commands such as query, doctor, and dashboard, do not mutate project files except for harmless logs/pointers created by the plugin itself.

## Quick Diagnostics

Use these when checking the installation:

```powershell
$plugin = (Resolve-Path ".\plugins\webnovel-writer").Path
python -X utf8 "$plugin\scripts\webnovel.py" --help
python -X utf8 "$plugin\scripts\webnovel.py" --project-root "<PROJECT_ROOT>" doctor --format text
```
