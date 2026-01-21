# Findings & Decisions
<!-- 
  WHAT: Your knowledge base for the task. Stores everything you discover and decide.
  WHY: Context windows are limited. This file is your "external memory" - persistent and unlimited.
  WHEN: Update after ANY discovery, especially after 2 view/browser/search operations (2-Action Rule).
-->

## Requirements
<!-- 
  WHAT: What the user asked for, broken down into specific requirements.
  WHY: Keeps requirements visible so you don't forget what you're building.
  WHEN: Fill this in during Phase 1 (Requirements & Discovery).
  EXAMPLE:
    - Command-line interface
    - Add tasks
    - List all tasks
    - Delete tasks
    - Python implementation
-->
<!-- Captured from user request -->
- MCP server must be stdio/command-based (no HTTP, no ports, no endpoints).
- CLI: `uvx yourskill-mcp` with repeated `--repo` and `--path` arguments.
- Scan repos/paths for `*/SKILL.md`; each skill becomes an MCP resource (unique name).
- Local cache allowed but rebuildable; no backend, no DB, no user system.
- Python + FastMCP + argparse/typer; stdio transport only.
- README must teach usage and explicitly state what it is NOT.
- SaaS mode is primary: read `YOURSKILL_API_URL` and `YOURSKILL_TOKEN`; pull subscribed skills.
- `--repo` / `--path` remain as advanced fallback for local skills.

## Research Findings
<!-- 
  WHAT: Key discoveries from web searches, documentation reading, or exploration.
  WHY: Multimodal content (images, browser results) doesn't persist. Write it down immediately.
  WHEN: After EVERY 2 view/browser/search operations, update this section (2-Action Rule).
  EXAMPLE:
    - Python's argparse module supports subcommands for clean CLI design
    - JSON module handles file persistence easily
    - Standard pattern: python script.py <command> [args]
-->
<!-- Key discoveries during exploration -->
- Planning skill templates located at `/home/wuy6/.codex/skills/planning-with-files/skills/planning-with-files/templates`.
- SaaS API v1: `GET /v1/skills` lists subscribed skills (slug, id, revision, files list).
- SaaS file fetch: `GET /v1/skills/{skill_id}/files/{path}` returns content, supports ETag/If-None-Match.
- Auth: `Authorization: Bearer <YOURSKILL_TOKEN>` header for all requests.
- Optional `GET /v1/me` for token validation (non‑MVP).

## Technical Decisions
<!-- 
  WHAT: Architecture and implementation choices you've made, with reasoning.
  WHY: You'll forget why you chose a technology or approach. This table preserves that knowledge.
  WHEN: Update whenever you make a significant technical choice.
  EXAMPLE:
    | Use JSON for storage | Simple, human-readable, built-in Python support |
    | argparse with subcommands | Clean CLI: python todo.py add "task" |
-->
<!-- Decisions made with rationale -->
| Decision | Rationale |
|----------|-----------|
| Use planning files in repo root | Required by planning-with-files skill |
| Use Pydantic v2 for config models | User requirement; typed validation for config |
| SaaS mode is primary via env vars YOURSKILL_API_URL/TOKEN; repo/path are fallback | User updated requirement |

## Issues Encountered
<!-- 
  WHAT: Problems you ran into and how you solved them.
  WHY: Similar to errors in task_plan.md, but focused on broader issues (not just code errors).
  WHEN: Document when you encounter blockers or unexpected challenges.
  EXAMPLE:
    | Empty file causes JSONDecodeError | Added explicit empty file check before json.load() |
-->
<!-- Errors and how they were resolved -->
| Issue | Resolution |
|-------|------------|
| session-catchup script missing due to `CLAUDE_PLUGIN_ROOT` unset | Used explicit template path; proceed without catchup |

## Resources
<!-- 
  WHAT: URLs, file paths, API references, documentation links you've found useful.
  WHY: Easy reference for later. Don't lose important links in context.
  WHEN: Add as you discover useful resources.
  EXAMPLE:
    - Python argparse docs: https://docs.python.org/3/library/argparse.html
    - Project structure: src/main.py, src/utils.py
-->
<!-- URLs, file paths, API references -->
- FastMCP skills provider doc referenced by user (https://gofastmcp.com/servers/providers/skills#template-mode-default)
- Planning templates: `/home/wuy6/.codex/skills/planning-with-files/skills/planning-with-files/templates`

## Visual/Browser Findings
<!-- 
  WHAT: Information you learned from viewing images, PDFs, or browser results.
  WHY: CRITICAL - Visual/multimodal content doesn't persist in context. Must be captured as text.
  WHEN: IMMEDIATELY after viewing images or browser results. Don't wait!
  EXAMPLE:
    - Screenshot shows login form has email and password fields
    - Browser shows API returns JSON with "status" and "data" keys
-->
<!-- CRITICAL: Update after every 2 view/browser operations -->
<!-- Multimodal content must be captured as text immediately -->
-

---
<!-- 
  REMINDER: The 2-Action Rule
  After every 2 view/browser/search operations, you MUST update this file.
  This prevents visual information from being lost when context resets.
-->
*Update this file after every 2 view/browser/search operations*
*This prevents visual information from being lost*

## Update (2026-01-21)
- Repo contains src/, pyproject.toml, main.py, skills/, and empty README.md.
- src/yourskill_mcp contains __init__.py, config.py, util/, server.py, providers/
- server.py currently empty.
- config.py and providers/__init__.py are empty placeholders.
- util/__init__.py and package __init__.py are empty.
- main.py just prints a hello message; no CLI wiring yet.
- pyproject.toml defines fastmcp dependency only; no scripts/entrypoints.
- skills/ currently contains only planning-with-files; demo skill directory missing.

## FastMCP Skills Provider Notes (2026-01-21)
- SkillsDirectoryProvider scans roots for directories containing SKILL.md; each becomes a skill resource.
- Resource URIs use skill://<skill>/SKILL.md and skill://<skill>/_manifest; supporting files can be templated.
- Default supporting_files="template" hides supporting files from list_resources; manifest is used to discover them.
- FastMCP SkillsDirectoryProvider exposes each SKILL.md as a skill:// resource; template mode hides supporting files from list_resources.

- Decision: default cache directory = ~/.cache/yourskill-mcp (user choice).
- Decision: repo handling = cache then git pull on subsequent runs.
- Decision: skill resource naming uses <source>@<short-hash>/<relative-path> to avoid collisions.
- Decision: config will read SaaS env vars in addition to CLI arguments (supersedes CLI-only).
- Decision: config inputs are CLI-only (no env or config file).

- New user requirement (conflicts with original): add SaaS mode via env vars YOURSKILL_API_URL and YOURSKILL_TOKEN to fetch subscribed skills; SaaS to be primary path; repo/path fallback.
- Decision: merge SaaS + local skills when both provided; local takes precedence on name collisions.
- Decision: all MCP resource URIs use skill:// scheme (SaaS + local).
