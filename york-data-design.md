# york-data Design — MCP Server

## Stack
- Node.js MCP server (called via `mcporter call york-data.<function>`)
- `better-sqlite3` — synchronous, WAL mode, single file DB
- No ORM, no query builder. Raw SQL with parameterized queries.

## Project Structure
```
york-data/
  package.json
  src/
    index.js              # MCP server entry, registers all domains
    db.js                 # Database init, migration runner
    domains/
      consumption.js      # Food/drink logging
      daily.js            # Daily metrics (weight, sleep, etc.)
      workouts.js         # Workout sessions + exercises
      cannabis.js         # Cannabis session tracking
      observations.js     # Bede's findings
  migrations/
    001-initial.sql       # All MVP tables
```

### Modularity Pattern
Each domain file exports an array of tool definitions:
```js
// domains/consumption.js
module.exports = [
  {
    name: 'log_food',
    description: 'Log a food/drink item',
    parameters: { ... },
    handler: (db, params) => { ... }
  },
  ...
]
```

`index.js` imports all domain files and registers them with the MCP server.

**Adding a new domain = add a file in `domains/`, import it in `index.js`.** That's it.

**Adding a new migration = add a numbered SQL file in `migrations/`.** The runner applies any unapplied migrations on startup.

---

## Database Schema (MVP)

### consumption
| Column | Type | Notes |
|--------|------|-------|
| id | INTEGER PRIMARY KEY | auto |
| date | TEXT NOT NULL | YYYY-MM-DD |
| time | TEXT | HH:MM or null |
| item | TEXT NOT NULL | what was consumed |
| quantity | TEXT | "2 slices", "1 bowl", etc. |
| calories | INTEGER | estimated or known |
| protein | INTEGER | grams |
| notes | TEXT | optional |
| created_at | TEXT | ISO timestamp, auto |

### daily_metrics
| Column | Type | Notes |
|--------|------|-------|
| date | TEXT PRIMARY KEY | YYYY-MM-DD, one row per day |
| weight | REAL | lbs, nullable |
| calories_total | INTEGER | daily total, nullable |
| protein_total | INTEGER | daily total grams, nullable |
| kitchen_closed | TEXT | time or null |
| wake_up | TEXT | time or null |
| bedtime | TEXT | time or null |
| sleep_hrs | REAL | nullable |
| energy | INTEGER | 1-5, nullable |
| updated_at | TEXT | ISO timestamp, auto-updated |

### workouts
| Column | Type | Notes |
|--------|------|-------|
| id | INTEGER PRIMARY KEY | auto |
| date | TEXT NOT NULL | YYYY-MM-DD |
| type | TEXT | "gym", "bodyweight", "swimming", etc. |
| location | TEXT | "home", "gym name", etc. |
| duration_min | INTEGER | nullable |
| notes | TEXT | optional |
| created_at | TEXT | ISO timestamp |

### exercises
| Column | Type | Notes |
|--------|------|-------|
| id | INTEGER PRIMARY KEY | auto |
| workout_id | INTEGER | FK → workouts.id |
| exercise | TEXT NOT NULL | exercise name |
| sets | INTEGER | |
| reps | INTEGER | |
| weight | REAL | lbs, nullable for bodyweight |
| notes | TEXT | optional |

### cannabis_sessions
| Column | Type | Notes |
|--------|------|-------|
| id | INTEGER PRIMARY KEY | auto |
| date | TEXT NOT NULL | YYYY-MM-DD |
| time | TEXT | HH:MM |
| session_number | INTEGER | nth session of the day |
| created_at | TEXT | ISO timestamp |

### observations
| Column | Type | Notes |
|--------|------|-------|
| id | INTEGER PRIMARY KEY | auto |
| domain | TEXT NOT NULL | "consumption", "routing", "conversation", etc. |
| type | TEXT NOT NULL | "correction", "error", "frustration", "pattern" |
| summary | TEXT NOT NULL | one-line finding |
| evidence | TEXT | supporting detail |
| severity | TEXT | "low", "medium", "high" |
| suggested_action | TEXT | what Bede thinks should happen |
| status | TEXT DEFAULT 'open' | "open", "addressed", "dismissed", "in-progress" |
| resolution | TEXT | what actually happened |
| created_at | TEXT | ISO timestamp |
| updated_at | TEXT | ISO timestamp |

---

## MCP Tools (MVP)

### Consumption Domain
```
log_food(date, item, quantity?, calories?, protein?, time?, notes?)
  → { id }

get_consumption(date)
  → { items: [...] }

get_consumption_range(start_date, end_date)
  → { items: [...] }

update_consumption(id, fields...)
  → { updated: true }

delete_consumption(id)
  → { deleted: true }
```

### Daily Metrics Domain
```
log_daily(date, weight?, calories_total?, protein_total?, kitchen_closed?, wake_up?, bedtime?, sleep_hrs?, energy?)
  → { upserted: true }
  # INSERT OR REPLACE — partial updates merge with existing row

get_daily(date)
  → { metrics: {...} }

get_daily_range(start_date, end_date)
  → { metrics: [...] }

get_weight_trend(days)
  → { weights: [...], moving_avg_7d: [...] }
```

### Workouts Domain
```
log_workout(date, type?, location?, duration_min?, notes?)
  → { id }

log_exercises(workout_id, exercises: [{exercise, sets, reps, weight?, notes?}])
  → { count }

get_workouts(start_date?, end_date?)
  → { workouts: [...] }  # includes exercises

get_last_workout()
  → { workout: {...} }
```

### Cannabis Domain
```
log_cannabis(date, time?, session_number?)
  → { id }

get_cannabis_sessions(date?)
  → { sessions: [...] }

get_cannabis_history(days)
  → { days: [...], avg_per_day, total }
```

### Observations Domain
```
log_observation(domain, type, summary, evidence?, severity?, suggested_action?)
  → { id }

get_observations(domain?, type?, status?, since?)
  → { observations: [...] }

update_observation(id, status?, resolution?)
  → { updated: true }
```

### Cross-Domain
```
get_day_summary(date)
  → { consumption: [...], metrics: {...}, workouts: [...], cannabis: [...] }

get_trends(days)
  → { weights: [...], calories: [...], protein: [...], cannabis: [...], workouts: [...] }
```

---

## Validation Rules (enforced by the server, not the LLM)

- **Dates:** Must be YYYY-MM-DD format. Reject anything else.
- **Calories:** 0-8000 range. Reject unreasonable values.
- **Protein:** 0-500g range.
- **Weight:** 100-400 lbs range.
- **Energy:** 1-5 integer.
- **Sleep:** 0-24 hours.
- **Session number:** 1-20 range.
- **Date bounds:** No future dates for consumption/workouts. Daily metrics allows today only for forward fields (bedtime).

## What's NOT in MVP

- Google Sheets sync/export (future: write-only mirror for James to view)
- Task/chore storage (lives in Google Tasks)
- Home project tracking (deferred)
- Flags from agents (the "Flag for Bede" global skill — add when needed)
- Backup automation (manual for now: copy the .db file)
- Auth/permissions (all agents get all functions for now)
- Schema versioning UI (migrations handle it)

## Migration Strategy

Data currently lives in Google Sheets. Migration approach:
1. Build york-data with empty tables
2. Write a one-time migration script that reads Sheets via mcporter and inserts into york-data
3. Run it once
4. Switch agents to york-data
5. Google Sheets becomes read-only archive
