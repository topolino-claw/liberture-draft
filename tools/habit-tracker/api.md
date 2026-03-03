# Habit Tracker v1 API

Base URL: `HABIT_TRACKER_URL` (e.g., `https://tasks.fabri.lat`)
Auth: `Authorization: Bearer <hti_token>`

## Endpoints

### GET /api/v1/habits

Returns all active habits with computed stats.

**Response:**
```json
[
  {
    "id": "uuid",
    "name": "Morning Run",
    "color": "#8B5CF6",
    "schedule": { "type": "daily" },
    "timeOfDay": "morning",
    "category": "exercise",
    "tags": ["exercise"],
    "currentStreak": 5,
    "longestStreak": 12,
    "completionRate7d": 0.86,
    "completionRate30d": 0.73,
    "completedToday": false,
    "lastCompletedAt": "2026-02-25"
  }
]
```

### GET /api/v1/stats

Returns overall statistics for the last 30 days.

**Response:**
```json
{
  "totalHabits": 8,
  "activeHabits": 6,
  "totalCompletions": 142,
  "completionRateToday": 0.75,
  "completionRate7d": 0.68,
  "bestCurrentStreak": 14,
  "bestHabitName": "Morning Run",
  "totalStreakDays": 89,
  "completionsByDay": [
    { "date": "2026-01-28", "count": 5 }
  ]
}
```

### GET /api/v1/completions?days=30

Returns recent completions with habit name. `days` max: 365.

**Response:**
```json
[
  {
    "habitId": "uuid",
    "habitName": "Morning Run",
    "date": "2026-02-25",
    "completedAt": "2026-02-25T08:30:00.000Z"
  }
]
```

### POST /api/v1/completions

Log a habit completion.

**Body:**
```json
{ "habitId": "uuid", "date": "2026-02-26" }
```
`date` is optional — defaults to today (UTC).

**Response:**
```json
{ "success": true, "alreadyCompleted": false, "currentStreak": 6 }
```

---

## Todo Endpoints

### GET /api/v1/todos

Returns todos with optional filtering and sorting.

**Query params:**
- `status`: `pending` (incomplete + in_progress), `completed`, `all` (default: `all`)
- `sort`: `deadline` (dueDate ASC, nulls last), `priority` (DESC), `created` (DESC, default)
- `overdue`: `true` to return only past-due items

**Response:** Array of Todo objects with an extra `urgency` field:
```json
[
  {
    "id": "uuid",
    "title": "Write blog post",
    "description": "About Bitcoin halving",
    "dueDate": "2026-03-01",
    "dueTime": "18:00",
    "priority": 4,
    "status": "incomplete",
    "canTopolinoHelp": true,
    "createdAt": "2026-02-27T00:00:00.000Z",
    "urgency": "this_week"
  }
]
```

`urgency` values: `"overdue"` | `"today"` | `"tomorrow"` | `"this_week"` | `"later"` | `"no_date"`

### POST /api/v1/todos

Create a new todo.

**Body:**
```json
{
  "title": "Write blog post",
  "description": "Optional details",
  "dueDate": "2026-03-01",
  "dueTime": "18:00",
  "priority": 4,
  "status": "incomplete",
  "canTopolinoHelp": true
}
```

- `priority`: 1–5, defaults to 3
- `status`: defaults to `"incomplete"`
- `canTopolinoHelp`: boolean — marks whether Topolino can assist (default false)

**Response:** The created todo object (with `id`, `createdAt`, `urgency`).

### PUT /api/v1/todos/:id

Update any fields of a todo.

**Body:** Any subset of `{ title, description, dueDate, dueTime, priority, status, canTopolinoHelp }`

- Setting `status: "completed"` auto-sets `completedAt`
- Setting `status: "incomplete"` or `"in_progress"` clears `completedAt`

**Response:** Updated todo object.

### PATCH /api/v1/todos/:id/status

Quick status change only.

**Body:**
```json
{ "status": "in_progress" }
```

**Response:** Updated todo object.

### DELETE /api/v1/todos/:id

Delete a todo.

**Response:** `{ "success": true }`

---

## Todo Data Model

```typescript
interface Todo {
  id: string
  title: string
  description?: string
  dueDate?: string        // YYYY-MM-DD
  dueTime?: string        // HH:MM
  priority: 1 | 2 | 3 | 4 | 5
  status: "incomplete" | "in_progress" | "completed"
  canTopolinoHelp?: boolean  // Can Topolino assist with this?
  completedAt?: string    // ISO timestamp (set when completed)
  createdAt: string       // ISO timestamp
  urgency?: string        // Computed: overdue/today/tomorrow/this_week/later/no_date
}
```

Priority labels: 1=Low · 2=Medium-Low · 3=Medium · 4=High · 5=Critical

---

## Error Codes

| Status | Meaning |
|--------|---------|
| 401 | Invalid or missing token |
| 404 | Todo/Habit not found |
| 409 | Token already exists (revoke first) |
| 500 | Server error (check DB connection) |

## Notes

- No rate limits currently
- All dates in `YYYY-MM-DD` format
- All timestamps in ISO 8601 UTC
- All data stored in JSONB blob per user
