---
name: steady-context
description: Pull the user's current Steady (MCP) work context to inform the approach and flag goal alignment. DEFAULT TO PULLING before substantive work on the user's job or product (coding, planning, writing/reviewing work content, prioritizing). Skip only if you can say in one line why the task is trivial, one-off, or unrelated — "it's just code", "it's a refactor", and "context won't change this" do NOT count. Skip follow-ups where context was already pulled this session. If the user signals to skip (e.g. "skip steady", "skip context"), honor it even when in scope.
---

# Pull Steady context

Pull the user's current Steady work context and distill it to what bears on the task.

**Run it in a subagent** that makes all calls below and returns ONLY the distilled summary (see "Return"). Give it: the user's task in a sentence, today's date, and these instructions. If your environment lets you pick the subagent's model, prefer a fast, lightweight one — this is mostly mechanical (scoped calls, short summary) and needs no heavy reasoning.

## What the subagent does

Ensure the Steady MCP tools are loaded (lazy environments: load the `Steady` tools first), then pull in **waves** — calls within a wave run in parallel.

Windows capture each artifact's _latest_ instance: **~7 days** for fast/bulky ones (check-ins, activities, echo answers — newest-first, so a short reach suffices); **~30 days** for slow goals, to span the update cadence.

**Wave 1 — `get_me`.** Returns the `person_id` and `team_ids` the rest need.

**Wave 2 (parallel), scoped with Wave 1 IDs:**

- `get_goals(people_ids: [me], team_ids: [...])` — goals the user owns, has a role on, or that a team of theirs is on.
- `get_check_ins(team_ids: [...], date_start: ~7d)` — recent intentions and blockers. Intentions are PLANS, not progress — never report them as done.
- `get_activities(team_ids: [...], date_start: ~7d)` — recent team activity; pass `kinds` if the task points to specific ones.
- `get_echo_questions` — index of the user's Echoes (titles, schedule, IDs); no answer bodies.

**Wave 3 (parallel, dependent):**

- `get_goal_updates(goal_ids: [Wave 2 goals], date_start: ~30d)` — recent updates, whoever wrote them.
- `get_echo_answers(question_id, date_start: ~7d)` for each Echo that bears on the task. A purpose-built Echo (e.g. "coding agent context") is high-signal; "personal insights"/"just for fun" are noise. Daily/weekly answers land inside ~7 days; a monthly one may not surface — fine for current context.

Scale to the task — a goal-focused one may need only goals + updates + the one relevant Echo. Pull more if warranted (`get_people`, `get_insights`, `get_absences`).

## Return

Return a **concise** summary that:

1. **Informs** — the context that bears on the task: relevant goals, recent decisions/plans, people, deadlines, relevant Echo content. Pull forward what's useful, not just what's misaligned.
2. **Signals alignment** — open with a 🟢/🟡/🔴 read:
   - 🟢 **Green** — well aligned with current goals and plans.
   - 🟡 **Yellow** — mostly aligned, but a priority/timing/scope nuance is worth weighing.
   - 🔴 **Red** — conflicts with goals/priorities, or something should change the approach.

Keep it short and specific. Then proceed, informed by it.

Pull **once per task** — don't re-pull each message. If later work needs an omitted detail, query the same subagent again rather than re-pulling everything.
