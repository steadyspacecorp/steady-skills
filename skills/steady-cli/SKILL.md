---
name: steady-cli
description: Use when working with the Steady command-line tool (the `steady` binary) — running commands like `steady check-ins`, `steady goals`, `steady me`, or `steady auth login` to read/write Steady data from the terminal. Covers install, auth (OAuth + `STEADY_TOKEN` for CI), discovering commands, JSON/jq output, filters, and create/update input. Trigger when the user runs or asks about the `steady` CLI, mentions `steady auth`, wants to script Steady from a shell, or wants to pull/update check-ins, goals, goal updates, echoes, activities, or people from the command line.
---

# Steady CLI

The `steady` binary reads and writes Steady data from the terminal. Use it when the user wants to *run commands*; for writing code against the API use the `steady-api` skill, and for Claude reading/writing data itself use the Steady MCP server (`https://runsteady.com/mcp`).

Every command prints JSON to stdout (pipe to `jq`). Commands mirror the Steady v2 API resources. `steady version` / `steady upgrade` manage the binary.

## Install

```sh
curl -fsSL https://cli.runsteady.com/install.sh | bash   # macOS / Linux / WSL2 / Git Bash
irm https://cli.runsteady.com/install.ps1 | iex          # Windows PowerShell
```

If `steady` isn't found after, it's not on `PATH` — check `command -v steady`.

## Auth

OAuth browser flow, credential managed for you:

```sh
steady auth login    # authorize in browser
steady auth status   # logged-in state + token expiry
steady auth logout
steady auth token    # print the access token (reuse in a Bearer header)
```

Tokens expire — an auth error on an otherwise-valid command means run `steady auth login` again; check `steady auth status` first. For CI/non-interactive use, skip the browser and set `STEADY_TOKEN` to a `steady_pat_` personal access token (created at `https://app.steady.space/my/integrations/edit`), sourced from a secret store, not hardcoded:

```sh
STEADY_TOKEN="steady_pat_…" steady check-ins
```

## Discovering commands

The list drifts as the CLI grows, so discover it live rather than memorizing it:

```sh
steady --help                       # all command groups
steady <command> --help             # a command's operations + flags
steady <command> <operation> --help # an operation's flags + input
```

Groups mirror the API: `check-ins`, `goals`, `goal-updates`, `echo`, `activities`, `digest`, `teams`, `people`, `me`, plus `auth`, `version`, `upgrade`.

## Reading

List commands print a JSON array; filter flags (see each command's `--help`):

- `--team-ids` / `--people-ids` / `--goal-ids` — comma-separated or repeated.
- `--since` / `--until` — inclusive. Dates (`YYYY-MM-DD`) for check-ins/goals; ISO 8601 (`2026-05-06T00:00:00Z`) for activities.
- `--page` (1-indexed) / `--per-page`.
- `--blocked` (check-ins) · `--kinds` (activities).

```sh
steady check-ins 2026-06-09                                       # a check-in by date (yours, that day)
steady goals 7a1d2e3f-4b5c-4d6e-8f90-1234567890ab                 # a goal by ID
steady activities --since 2026-04-01 --kinds github_pr,jira
steady check-ins --since 2026-06-08 --until 2026-06-14 --blocked | jq -r '.[].person.name'
steady me | jq -r '.id'                                           # your person ID; no "me"/"self" shorthand in filters
steady teams | jq -r '.[] | select(.name=="Product") | .id'      # a team UUID for --team-ids
```

## Writing: `<json>|@<file>`

Create/update take one positional arg — inline JSON or `@file.json`:

```sh
steady goals create '{"title":"Reduce p95 API latency","end_date":"2026-06-30"}'
steady check-ins 2026-06-15 update @checkin.json   # cleaner for long Markdown bodies
```

Field names match the API schemas (the CLI sends your JSON straight through); for exact shapes fetch `https://app.steady.space/openapi.yml`.

Quirks worth knowing:

- **Check-ins are update-only** — no `create`. Steady pre-generates them from team schedule + absences, so "filling out a check-in" is `check-ins <key> update`, where `<key>` is a date (your check-in that day) or UUID. Body: `previous`, `intentions`, `blockers`, `mood`, `previous_completed`, optional `team_ids`. Errors if no check-in exists for the date (off-schedule or absent).
- **Goal updates: nested on create/list, flat on access** — create/list via `goals <id> goal-updates …`; get/update/delete via `goal-updates <id> …`. On create, `progress`/`confidence` default to the prior update (omit for prose-only); `confidence` snaps to 30/60/90 (off track / at risk / on track).
- **`mood`** is a fixed enum (`focused`, `happy`, `tired`, …) — see the spec.
- **Markdown fields** (`previous`, `intentions`, `blockers`, comment `body`, …) take Markdown, incl. `@username` mentions.

## Troubleshooting

- Auth error → `steady auth status`, then `steady auth login` (or set `STEADY_TOKEN`).
- `command not found` → not on `PATH`; check `command -v steady`.
- Empty `[]` → filters too narrow, or no records; loosen to confirm.
- Write rejected → field/value off; compare against the OpenAPI spec.
- CLI doesn't expose what you need → `steady auth token` + hit the HTTP API.
