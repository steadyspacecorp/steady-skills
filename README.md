# steady-skills

Skills for working with [Steady](https://runsteady.com) from AI-powered tools.

Each skill is a self-contained folder with a `SKILL.md` written to the [Agent Skills](https://agentskills.io) open standard — the same format Claude Code, the Claude apps, and OpenAI Codex all read. Drop a skill into your assistant's skills directory and it loads the right Steady knowledge automatically when a task calls for it (or on demand when you invoke it by name).

## Available skills

| Skill | What it does | Use it when |
|-------|--------------|-------------|
| [`steady-api`](skills/steady-api/SKILL.md) | Write correct, well-behaved code against Steady's v2 REST API — auth with `steady_pat_` tokens, the OpenAPI spec as source of truth, rate-limit handling, and the shapes for common tasks (check-ins, goal updates, activity, people). | You're writing scripts, clients, or integrations that hit `service.steady.space/api/v2`. |
| [`steady-cli`](skills/steady-cli/SKILL.md) | Drive Steady from the terminal with the `steady` binary — install, OAuth + `STEADY_TOKEN` for CI, discovering commands, JSON/`jq` output, filters, and create/update input. | You want to read or update check-ins, goals, activities, etc. from a shell. |
| [`steady-updates`](skills/steady-updates/SKILL.md) | Write clear, useful check-ins and goal updates — adding context beyond captured activity, right length, unambiguous people references, scannable formatting. This is the *what to write* skill; the API/CLI skills cover *how to submit*. | You're drafting or polishing a check-in or goal update (as a person, or as an agent reporting progress). |

> **Note on the [Steady MCP server](https://runsteady.com/docs/article/143-mcp-server/):** it lets an assistant read and write Steady directly as tool calls in a conversation — no API or CLI code involved. It's the easiest *transport* for that case, and pairs naturally with `steady-updates` (which covers *what* to write). Use `steady-api` / `steady-cli` when you'd rather move the data through code, a script, or a shell.

## Install

The same skill folders (under [`skills/`](skills/)) work across every tool below. The fastest paths are one command each; manual copy/upload still works where there's no CLI.

### Claude Code

Add this repo as a plugin marketplace, then install the bundled `steady-skills` plugin (all three skills):

```
/plugin marketplace add steadyspacecorp/steady-skills
/plugin install steady-skills
```

Claude loads each skill when relevant, or invoke one directly with `/steady-api`, `/steady-cli`, `/steady-updates`. Update later with `/plugin update steady-skills`.

→ Full reference: [Extend Claude with skills](https://code.claude.com/docs/en/skills)

### Agent Skills hosts (Codex, Cursor, Copilot, Gemini CLI, Zed, …)

Any host that reads the open [Agent Skills](https://agentskills.io) format can install with the [`skills`](https://github.com/vercel-labs/skills) CLI — it discovers all three skills under `skills/` and writes them to the right place for your agent:

```sh
# Personal (all repos) — drop -g for project-local (./<agent>/skills/)
npx skills add steadyspacecorp/steady-skills -g

# — or — install a single skill by name
npx skills add steadyspacecorp/steady-skills/steady-api -g
```

→ Full reference: [Agent Skills — Codex](https://developers.openai.com/codex/skills)

### Claude apps (web & desktop)

The apps install skills by **upload**, not the filesystem. First enable **Code Execution** and **File Creation** under Settings → Capabilities (skills require them; available on Pro, Max, Team, and Enterprise plans). Then upload each skill's `SKILL.md` under Settings → Features → Skills.

→ Step-by-step: [Use skills in Claude](https://support.claude.com/en/articles/12512180-use-skills-in-claude)

### Manual copy

Prefer the filesystem? Clone and copy the folders into any agent's skills directory:

```sh
git clone https://github.com/steadyspacecorp/steady-skills.git

# Claude Code — personal or project
cp -R steady-skills/skills/steady-* ~/.claude/skills/

# Codex and other .agents hosts
cp -R steady-skills/skills/steady-* ~/.agents/skills/
```

### ChatGPT (consumer app)

The consumer ChatGPT app does **not** have a native Agent Skills feature — skills are a Codex capability (CLI/IDE/app). To use these with ChatGPT directly, either use **Codex** (above), or paste a skill's `SKILL.md` contents into a [Project's custom instructions](https://help.openai.com/en/articles/10169521-projects-in-chatgpt) or a Custom GPT. The content is portable even where the auto-discovery format isn't.
