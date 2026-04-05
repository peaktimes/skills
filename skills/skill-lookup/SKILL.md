---
name: skill-lookup
description: Activates when the user asks about Agent Skills, reusable prompts, or extending the assistant with capabilities from prompts.chat. Use for discovering, retrieving, and saving content via the prompts.chat MCP (search_prompts, get_prompt). Applies to Cursor skills and Claude-style skill folders when converting or installing retrieved material.
---

When the user wants reusable instructions, agent-style prompts, or Cursor **Agent Skills**, use the **prompts.chat** MCP server (`prompts.chat` in Cursor MCP settings). That service is the **[prompts.chat](https://prompts.chat)** library ([open-source project](https://github.com/f/prompts.chat)): community **prompts** you can search and fetch. Treat highly structured results as candidates to turn into a local **`SKILL.md`** if the user wants a persistent skill.

## When to Use This Skill

Activate when the user:

- Asks to find prompts or skills (“code review”, “testing”, “documentation”, etc.)
- Wants to search what is available on prompts.chat
- Wants the full text of a specific item by ID
- Wants to **save** a new prompt (requires authentication — see below)
- Wants to install or adapt retrieved content as a **Cursor skill** (folder + `SKILL.md`)

## Available Tools (prompts.chat MCP)

Call only tools that the connected MCP actually exposes. The standard **`@fkadev/prompts.chat-mcp`** / hosted API surface uses:

| Tool | Purpose |
|------|--------|
| **`search_prompts`** | Search public prompts by keyword; filter by type, category, tag |
| **`get_prompt`** | Fetch one prompt by ID (body text, metadata, template variables like `${var}`) |
| **`save_prompt`** | Save a prompt to the user’s account (**requires `PROMPTS_API_KEY`** in local MCP env — not used for casual search/fetch) |

**Deprecated names (do not use):** `search_skills`, `get_skill` — they are not registered on this server.

## How to Search

Call **`search_prompts`** with:

- **`query`**: Keywords from the user’s request
- **`limit`**: Optional cap on results (use server defaults if unsure)
- **Filters** as supported by the tool (e.g. type, category, tag)

Present results clearly:

- Title and description
- Author (if shown)
- Category / tags (if shown)
- Link or ID so the user can say “get that one”

Note: The catalog is **prompt-centric**. Items may be single prompt texts, not multi-file skill bundles. If the user needs a **Cursor skill**, offer to turn the best match into a **`SKILL.md`** with proper YAML frontmatter (`name`, `description`).

## How to Retrieve Full Content

Call **`get_prompt`** with:

- **`id`**: The prompt ID from search results

Use the returned text as the basis for answers, summarization, or for writing a local skill file.

## How to Install as a Cursor Agent Skill

When the user asks to “install” or keep something as a skill:

1. Use **`get_prompt`** (or the chosen search hit) to get the full text.
2. Choose a **folder name** (lowercase, hyphens) matching the skill `name` in frontmatter.
3. Write:

   **User-wide:** `~/.cursor/skills/{skill-name}/SKILL.md`  
   **Project-only:** `{workspace}/.cursor/skills/{skill-name}/SKILL.md`

4. Ensure **`SKILL.md`** starts with:

   ```yaml
   ---
   name: skill-name
   description: When the agent should use this skill (what + when).
   ---
   ```

   Put the retrieved instructions (edited for clarity) in the markdown body.

5. If the source included extra files (rare for plain prompts), mirror the same paths under that folder.

## Claude Code path (optional)

If the user uses **Claude Code** instead: skill folders are often **`./.claude/skills/{skill-name}/`** with the same `SKILL.md` layout.

## Guidelines

- Prefer **`search_prompts`** before inventing a skill from scratch.
- After **`get_prompt`**, summarize what the prompt is for and when to use it.
- For **`save_prompt`**, only attempt if the user has configured API access; otherwise say saving needs auth.
- After installing a skill, remind the user to **restart Cursor** (or reload window) if the new skill does not appear in `@` yet.
