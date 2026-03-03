---
name: habit-tracker
description: "Interact with the user's Habit Tracker and Todo app (habits, streaks, completions, todos, deadlines, pomodoro). Use when: asking about habits, streaks, completion rates, logging a habit, checking todos, adding/updating/deleting todos, getting pending tasks or upcoming deadlines, starting/pausing/cancelling a Pomodoro timer, or asking what Topolino can help with. Requires HABIT_TRACKER_URL and HABIT_TRACKER_TOKEN configured."
---

# Habit Tracker + Todo Skill

Connects Topolino to Fabri's personal task and habit tracker at `tasks.fabri.lat` via the v1 REST API.

## Config

Stored in `~/.openclaw/habit-tracker.conf`:
```ini
[habit-tracker]
url = https://tasks.fabri.lat
token = hti_...
```

```python
import configparser, os

def get_config():
    conf = configparser.ConfigParser()
    conf.read(os.path.expanduser("~/.openclaw/habit-tracker.conf"))
    url = conf.get("habit-tracker", "url", fallback=None)
    token = conf.get("habit-tracker", "token", fallback=None)
    return url, token
```

All requests: `Authorization: Bearer <token>`

Full API reference: `references/api.md`

---

## ⚠️ Critical Rules

1. **habitId MUST be a string** — sending as integer returns 400. Always `str(habit_id)` or use `"id"` directly from API response.
2. **Sequential writes only** — NEVER run multiple todo/completion writes concurrently. JSONB blob storage causes race conditions.
3. **Log immediately** — when Fabri says he did something (yoga, pushups, meditation, etc.), log it RIGHT AWAY without asking for confirmation. Don't wait. Don't forget.

---

## Habit ID Cache

Use these cached IDs to skip the `/api/v1/habits` fetch when logging common habits:

```python
HABIT_IDS = {
    "read": "2",
    "reading": "2",
    "meditation": "3",
    "mindfulness": "3",
    "macroeconomics": "1767872564800",
    "studying": "1767872564800",
    "yoga": "1767872688499",
    "night yoga": "1767872688499",
    "push ups": "1767873067162",
    "pushups": "1767873067162",
    "day planning": "1767874552107",
    "planning": "1767874552107",
    "syncthing": "1767874666379",
    "eating ritual": "1767874752562",  # ⭐ MOST IMPORTANT
    "eating": "1767874752562",
    "pistol squats": "1768079284739",
    "squats": "1768079284739",
    "journal": "1768167299560",
    "nighttime journal": "1768167299560",
    "cold approaching": "1769104658652",
    "cold approach": "1769104658652",
    "handstand": "1769375017058",
    "digital life": "1769518318510",
    "ordering": "1769518318510",
    "coffee": "1770075085990",
    "breakfast": "1770075110140",
    "dinner": "1770292307275",
    "no ig": "1770415781242",
    "instagram": "1770415781242",
    "nf": "1770460035823",
    "pull ups": "1770931788860",
    "pullups": "1770931788860",
    "running": "1771618546734",
    "piano": "1771637529552",
}

def find_habit_id(name: str) -> str | None:
    """Try cache first, fall back to API search."""
    key = name.lower().strip()
    # Direct cache hit
    for k, v in HABIT_IDS.items():
        if k in key or key in k:
            return v
    return None
```

---

## Common Patterns

### "I just did [habit]" — Log immediately
```python
import requests, time

def log_habit(habit_name: str):
    url, token = get_config()
    h = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}

    # Try cache first
    habit_id = find_habit_id(habit_name)

    # Fall back to API search
    if not habit_id:
        habits = requests.get(f"{url}/api/v1/habits", headers={"Authorization": f"Bearer {token}"}).json()
        match = next((x for x in habits if habit_name.lower() in x["name"].lower()), None)
        if not match:
            return f"❌ Habit '{habit_name}' not found"
        habit_id = str(match["id"])  # ⚠️ always string

    r = requests.post(f"{url}/api/v1/completions", headers=h, json={"habitId": habit_id})
    data = r.json()

    if not r.ok:
        return f"❌ Failed to log: {data}"
    if data.get("alreadyCompleted"):
        return f"✅ Already logged today"
    streak = data.get("currentStreak", 0)
    return f"✅ Logged! Streak: {streak} day{'s' if streak != 1 else ''}"
```

### "How are my habits?"
```python
url, token = get_config()
h = {"Authorization": f"Bearer {token}"}
stats = requests.get(f"{url}/api/v1/stats", headers=h).json()
habits = requests.get(f"{url}/api/v1/habits", headers=h).json()

# Show: overall completion rate, streaks, what's done today
completed_today = [hb for hb in habits if hb.get("completedToday")]
not_done = [hb for hb in habits if not hb.get("completedToday")]
print(f"Today: {len(completed_today)}/{len(habits)} habits done")
print(f"Best streak: {stats.get('bestStreak', 0)} days")
```

---

## ⚠️ Critical: Sequential Operations

**NEVER run multiple write operations concurrently.** Always sequential with optional delay:

```python
def api_call(method, path, **kwargs):
    url, token = get_config()
    h = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}
    r = getattr(requests, method)(f"{url}{path}", headers=h, **kwargs)
    if not r.ok:
        raise Exception(f"API error {r.status_code}: {r.text}")
    return r.json()

for op in operations:
    result = api_call(**op)
    time.sleep(0.3)  # safety gap between writes
```

---

## Todo Patterns

### Fetch pending todos
```python
from datetime import date

url, token = get_config()
h = {"Authorization": f"Bearer {token}"}
todos = requests.get(f"{url}/api/v1/todos?status=pending&sort=deadline", headers=h).json()

overdue   = [t for t in todos if t["urgency"] == "overdue"]
today     = [t for t in todos if t["urgency"] == "today"]
tomorrow  = [t for t in todos if t["urgency"] == "tomorrow"]
this_week = [t for t in todos if t["urgency"] == "this_week"]

lines = []
if overdue:
    lines.append(f"⚠️ OVERDUE ({len(overdue)}):")
    for t in overdue:
        flag = " ⛄" if t.get("canTopolinoHelp") else ""
        lines.append(f"  • [P{t['priority']}] {t['title']}{flag}")
if today:
    lines.append(f"📅 TODAY ({len(today)}):")
    for t in today:
        flag = " ⛄" if t.get("canTopolinoHelp") else ""
        time_str = f" at {t['dueTime']}" if t.get("dueTime") else ""
        lines.append(f"  • [P{t['priority']}] {t['title']}{time_str}{flag}")
if tomorrow:
    lines.append(f"📆 TOMORROW ({len(tomorrow)}):")
    for t in tomorrow:
        flag = " ⛄" if t.get("canTopolinoHelp") else ""
        lines.append(f"  • [P{t['priority']}] {t['title']}{flag}")
print("\n".join(lines) if lines else "✅ Nothing urgent")
```

### Add a todo
```python
todo = requests.post(f"{url}/api/v1/todos", headers={**h, "Content-Type": "application/json"}, json={
    "title": "Title here",
    "dueDate": "2026-03-05",   # optional, YYYY-MM-DD
    "priority": 4,              # 1-5
    "canTopolinoHelp": False,   # True if Topolino can handle it
    "description": "Optional notes"
}).json()
```

### Mark todo done
```python
todos = requests.get(f"{url}/api/v1/todos?status=pending", headers=h).json()
match = next((t for t in todos if keyword.lower() in t["title"].lower()), None)
if match:
    r = requests.patch(f"{url}/api/v1/todos/{match['id']}/status",
        headers={**h, "Content-Type": "application/json"},
        json={"status": "completed"})
    print(f"✅ Done: {match['title']}")
```

### Update / Delete todo
```python
# Update
requests.put(f"{url}/api/v1/todos/{todo_id}", headers={**h, "Content-Type": "application/json"},
    json={"priority": 5, "dueDate": "2026-03-01", "canTopolinoHelp": True})

# Delete
requests.delete(f"{url}/api/v1/todos/{todo_id}", headers=h)
```

### "What can Topolino help with?"
```python
todos = requests.get(f"{url}/api/v1/todos?status=pending", headers=h).json()
helpable = [t for t in todos if t.get("canTopolinoHelp")]
for t in helpable:
    print(f"⛄ [P{t['priority']}] {t['title']} ({t['urgency']})")
```

---

## Reference Tables

### Urgency
| urgency | Meaning |
|---------|---------|
| `overdue` | Past due date |
| `today` | Due today |
| `tomorrow` | Due tomorrow |
| `this_week` | Due within 7 days |
| `later` | Due further out |
| `no_date` | No deadline set |

### Priority
| value | Label |
|-------|-------|
| 1 | Low |
| 2 | Medium-Low |
| 3 | Medium |
| 4 | High |
| 5 | Critical |

### canTopolinoHelp
Flag `true` when a task is something Topolino can handle: coding, writing, research, planning. Show ⛄ badge in summaries.

---

## Error Handling

Always check responses. Never say "done" if something failed.

```python
def safe_api(method, path, label="", **kwargs):
    url, token = get_config()
    h = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}
    try:
        r = getattr(requests, method)(f"{url}{path}", headers=h, **kwargs)
        if r.status_code == 401:
            return None, "❌ Auth failed — check token in ~/.openclaw/habit-tracker.conf"
        if r.status_code == 404:
            return None, f"❌ Not found: {label}"
        if not r.ok:
            return None, f"❌ Error {r.status_code} on {label}: {r.text[:200]}"
        return r.json(), None
    except Exception as e:
        return None, f"❌ Request failed on {label}: {e}"
```

---

## Chart Script

```bash
python3 scripts/chart.py --output /tmp/habits.png --json '{"stats": {...}, "habits": [...]}'
```

After generating: use `message` tool with `media=/tmp/habits.png` to send as image.

---

## Heartbeat / Periodic Todo Fetch

During heartbeats:
- Fetch `?status=pending&sort=deadline`
- Alert if any `overdue` or `today` todos — once per item per 24h
- Track in `memory/heartbeat-state.json` under `"lastTodoAlert"`

---

## 🍅 Pomodoro

API base: `/api/v1/pomodoro/`
Default duration: **15 minutes**. Custom: 1–120 min.

### Start a Pomodoro
```python
def pomodoro_start(duration_min=15):
    url, token = get_config()
    h = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}
    r = requests.post(f"{url}/api/v1/pomodoro/start", headers=h, json={"duration": duration_min})
    data = r.json()
    if not r.ok:
        return None, f"❌ {data}"
    return data, None  # { id, duration, startedAt, endsAt }
```

After starting, **always schedule a cron notification**:
```python
# Use OpenClaw cron tool — one-shot "at" job
# endsAt is ISO string from the API response
# job payload: systemEvent text read like a reminder
```
→ Use the `cron` tool with `action=add`, schedule `kind=at`, `at=<endsAt>`, `payload.kind=systemEvent`,
  `payload.text="⏱ Pomodoro done! Fabri, what did you work on? (reply to log your habit)"`,
  `sessionTarget=main`.

### Pause / Resume
```python
def pomodoro_pause():
    url, token = get_config()
    h = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}
    r = requests.post(f"{url}/api/v1/pomodoro/pause", headers=h)
    return r.json()  # { paused: true/false, pausedAt, totalPausedMs }
```

### Cancel (no log)
```python
def pomodoro_cancel():
    url, token = get_config()
    h = {"Authorization": f"Bearer {token}"}
    r = requests.delete(f"{url}/api/v1/pomodoro/active", headers=h)
    return r.json()  # { cancelled: true }
    # Also cancel the pending cron notification if one exists
```

### Check active session
```python
def pomodoro_active():
    url, token = get_config()
    h = {"Authorization": f"Bearer {token}"}
    r = requests.get(f"{url}/api/v1/pomodoro/active", headers=h)
    data = r.json()
    if not data.get("active"):
        return "No active Pomodoro"
    remaining = data.get("remainingMs", 0) // 1000
    mins, secs = divmod(remaining, 60)
    paused = " (paused)" if data.get("paused") else ""
    return f"⏱ {mins}m {secs}s remaining{paused}"
```

### Complete + log habit (called AFTER asking Fabri what he worked on)
```python
def pomodoro_complete(habit_name=None, label=None):
    url, token = get_config()
    h = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}
    body = {}
    if habit_name:
        habit_id = find_habit_id(habit_name)
        if habit_id:
            body["habitId"] = habit_id
    if label:
        body["label"] = label
    r = requests.post(f"{url}/api/v1/pomodoro/complete", headers=h, json=body)
    return r.json()  # { logged: true, session }
```

### Session history
```python
def pomodoro_sessions(limit=10):
    url, token = get_config()
    h = {"Authorization": f"Bearer {token}"}
    r = requests.get(f"{url}/api/v1/pomodoro/sessions?limit={limit}", headers=h)
    return r.json()
```

### Full Pomodoro flow (Topolino-managed)

When Fabri says "start a pomodoro [N min]":
1. Call `pomodoro_start(N)` → get `endsAt`
2. Schedule cron: `action=add`, `kind=at`, `at=<endsAt>`, `sessionTarget=main`,
   `payload.kind=systemEvent`, `payload.text="⏱ Reminder: Pomodoro finished! Ask Fabri what he worked on and call pomodoro_complete() with his answer."`
3. Store cron `jobId` in `memory/pomodoro-state.json` so it can be cancelled if needed

When Fabri says "cancel pomodoro":
1. Call `pomodoro_cancel()`
2. Read `memory/pomodoro-state.json` → get cron `jobId` → `cron action=remove`
3. Delete `memory/pomodoro-state.json`

When the cron fires:
1. Message Fabri: *"⏱ Pomodoro done! What did you work on?"*
2. Wait for his reply
3. Call `pomodoro_complete(habit_name=reply)`
4. Confirm: *"✅ Logged! [habit] completion recorded."*
