# Habit Tracker — OpenClaw Skill

> A neuroscience-backed personal habit tracker with a full REST API, integrated with OpenClaw as an agent skill. Built by Fabricio Acosta as a daily personal tool.

**Live app:** [tasks.fabri.lat](https://tasks.fabri.lat)  
**Repo:** [Fabricio333/v0-habit-tracker-app](https://github.com/Fabricio333/v0-habit-tracker-app)

---

## What It Is

A habit tracker and todo app built on neuroscience principles (Atomic Habits, implementation intentions, habit stacking, streak psychology). It exposes a REST API that an AI agent (Topolino) uses to:

- Log habit completions instantly when the user reports doing something
- Query streaks, stats, and completion rates
- Manage todos with priorities, deadlines, and urgency classification
- Run Pomodoro sessions with DB-persisted history and agent-managed notifications

---

## Stack

| Layer | Tech |
|-------|------|
| Framework | Next.js 16, React 19, TypeScript |
| UI | TailwindCSS 4, shadcn/ui, Radix UI |
| Database | Neon PostgreSQL — data stored as JSONB blob per user |
| Auth | API token (`hti_...`) or Nostr pubkey (NIP-07/NIP-46) |
| AI | Ollama (local LLM, llama3.2:1b) for habit insights |

---

## Neuroscience Foundations

1. **Implementation Intentions** — if-then planning, 2–3x success rate (300+ studies)
2. **Atomic Habits** — identity formation ("I am someone who...")
3. **Tiny Habits** (BJ Fogg) — start tiny, scale after automaticity
4. **Habit Stacking** — chain habits for momentum
5. **Streak Psychology** — visual progress creates dopamine loops

---

## OpenClaw Integration

The habit tracker exposes a `v1` REST API consumed by an OpenClaw skill (`habit-tracker`). The skill handles:

### Habits
- `GET /api/v1/habits` — list all habits with streaks + completion status
- `GET /api/v1/stats` — overall stats (completion rates, best streaks)
- `GET /api/v1/completions?days=30` — recent completion history
- `POST /api/v1/completions` — log a completion (`habitId` as string, always)

### Todos
- `GET /api/v1/todos?status=pending&sort=deadline` — fetch with urgency classification
- `POST /api/v1/todos` — create with title, dueDate, priority, canTopolinoHelp flag
- `PUT /api/v1/todos/:id` — update any fields
- `PATCH /api/v1/todos/:id/status` — quick status change
- `DELETE /api/v1/todos/:id` — delete

### Pomodoro 🍅
- `POST /api/v1/pomodoro/start` — start session (default 15 min, custom 1–120)
- `POST /api/v1/pomodoro/pause` — toggle pause/resume
- `GET /api/v1/pomodoro/active` — get active session with `remainingMs`
- `DELETE /api/v1/pomodoro/active` — cancel without logging
- `POST /api/v1/pomodoro/complete` — complete + log to history (optionally link a habit)
- `GET /api/v1/pomodoro/sessions?limit=20` — session history

### Auth Token Management
- `POST /api/v1/auth/token` — generate a new OpenClaw integration token
- `DELETE /api/v1/auth/token` — revoke token

---

## Agent Skill Flow

### Habit Logging
When user says *"I just did [habit]"*:
1. Skill checks local ID cache (avoids API roundtrip for known habits)
2. Falls back to `GET /api/v1/habits` search if not cached
3. `POST /api/v1/completions` with `habitId` as **string** (integer returns 400)
4. Confirms with streak count

### Pomodoro Notification Flow
1. User: *"start a pomodoro [N] min"*
2. Agent calls `POST /api/v1/pomodoro/start`
3. Agent schedules a one-shot cron job for `endsAt` time
4. Cron fires → agent messages user: *"⏱ Done. What did you work on?"*
5. User replies → agent calls `POST /api/v1/pomodoro/complete` with matched `habitId`
6. Session logged to DB history

### Todo Urgency Classification
The API computes `urgency` server-side:

| urgency | Meaning |
|---------|---------|
| `overdue` | Past due date |
| `today` | Due today |
| `tomorrow` | Due tomorrow |
| `this_week` | Within 7 days |
| `later` | Further out |
| `no_date` | No deadline |

The `canTopolinoHelp` boolean flag marks todos the agent can handle autonomously.

---

## Critical Rules for Agent Integration

1. **`habitId` MUST be a string** — integer causes 400
2. **Sequential writes only** — JSONB blob has race conditions on concurrent writes
3. **Log immediately** — don't ask for confirmation, don't delay
4. **Check responses** — never report success if API returned an error

---

## Files in This Directory

| File | Contents |
|------|----------|
| `README.md` | This overview |
| `SKILL.md` | Full OpenClaw skill spec with code patterns |
| `api.md` | Complete v1 API reference |

---

## Relation to Liberture

The habit tracker is a **real-world implementation** of Liberture's "Functions" pillar — an agent-accessible tool that directly supports human performance improvement (habits, focus sessions, task management). The skill design here is a reference implementation for how Liberture tools integrate with AI agents.
