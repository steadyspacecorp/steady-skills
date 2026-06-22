---
name: steady-context
description: Pull the user’s current Steady (MCP) work context — live work memory — to inform the approach and flag alignment with current goals. DEFAULT TO PULLING before any substantive work on the user’s job or product — coding on their codebase, planning, writing or reviewing work content, or prioritizing. At the start of such a task, USE this skill unless you can state in one line why the task is genuinely trivial, one-off, or unrelated to the user’s job. “It’s just code”, “it’s a refactor”, and “context won’t change this” are NOT valid reasons to skip. Also skip follow-ups where context was already pulled this session. User override — if the user signals to skip (e.g. “skip steady”, “no steady context”, “skip context”), do NOT pull — honor it even when the task is in scope.
---

# Pull Steady context

Do this entirely **in this thread** — making the calls AND reading/summarizing what comes back. Never spawn a subagent for any part of it; that adds a slow round-trip and is what we're avoiding.

`get_goals` depends on `get_me` (you need the person ID to scope goals to the user), so run two waves rather than one batch:

**Wave 1 — `get_me` first.** It's small and fast, and returns the `people`/`team` IDs the next calls need.

1. `get_me` — the authenticated user.

**Wave 2 — after `get_me` returns, fire these two in parallel** (don't wait on the digest before starting goals):

2. `get_goals` — pass the user's person ID as `people_ids` so you get the goals they're actually involved in, not every active goal.
3. `get_digest` — recent activity across the user's teams; highest-signal summary. There's no recency limit, only a `category` filter — when the task is narrow (e.g. goal-focused), pass the matching `category` (`goals`, `check-ins`, etc.) to cut volume; otherwise read the full digest inline (no subagent).

If the task warrants it, pull more (e.g. `get_check_ins`, `get_activities`).

Then, before doing the work, give a **concise** summary that does two things:

1. **Informs** — surface the context that actually bears on this task: relevant goals, recent decisions or plans, people, deadlines, or anything that should shape the approach. This is the memory part — pull forward what's useful, not just what's misaligned.
2. **Signals alignment** — open with a green/yellow/red read:
   - 🟢 **Green** — well aligned with current goals and stated plans. Say so plainly.
   - 🟡 **Yellow** — mostly aligned, but a priority, timing, or scope nuance is worth weighing.
   - 🔴 **Red** — conflicts with current goals/priorities, or something in the context should change the approach.

Keep it short and specific to what actually matters for this task.

Pull **once per task**. Don't re-pull on every message in the same thread.
