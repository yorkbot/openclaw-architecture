<template>
  <div>
    <div class="card">
      <h2>york-data Service Layer</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Custom MCP server sitting between agents and data. Agents call typed functions with named parameters.
        The service validates, enforces schema, and handles all DB operations. LLMs never see raw queries or column indices.
      </p>
      <p style="color: #c9d1d9; margin-bottom: 0; line-height: 1.6; font-size: 0.9rem; border-left: 3px solid #d29922; padding-left: 1rem;">
        <strong style="color: #d29922;">Stack:</strong> Node.js MCP server + <code style="color: #d29922;">better-sqlite3</code> (synchronous, WAL mode, single file DB).
        No ORM, no query builder. Raw SQL with parameterized queries.
        Called via <code style="color: #d29922;">mcporter call york-data.&lt;function&gt;(...)</code>.
      </p>
    </div>

    <!-- Architecture -->
    <div class="card">
      <h3>Project Structure</h3>
      <div class="structure">
        <div class="struct-item" v-for="f in structure" :key="f.path">
          <code class="struct-path">{{ f.path }}</code>
          <span class="struct-desc">{{ f.desc }}</span>
        </div>
      </div>
      <p style="color: #3fb950; font-size: 0.85rem; margin-top: 1rem; font-style: italic;">
        Adding a new domain = add a file in domains/, import it in index.js. That's it.
      </p>
    </div>

    <!-- Database Schema -->
    <div class="card border-yellow">
      <h3>Database Schema (MVP)</h3>
      <div class="table-list">
        <div class="table-card" v-for="t in tables" :key="t.name">
          <div class="table-name">{{ t.name }}</div>
          <div class="table-desc">{{ t.desc }}</div>
          <div class="col-list">
            <div class="col-item" v-for="c in t.columns" :key="c.name">
              <code class="col-name">{{ c.name }}</code>
              <span class="col-type">{{ c.type }}</span>
              <span class="col-note" v-if="c.note">{{ c.note }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Function Inventory by Domain -->
    <div class="card" v-for="domain in domains" :key="domain.name" :class="'border-' + domain.color">
      <h3>{{ domain.icon }} {{ domain.name }}</h3>
      <p style="color: #8b949e; margin-bottom: 1rem; font-size: 0.85rem;">{{ domain.desc }}</p>

      <div class="func-list">
        <div class="func-item" v-for="fn in domain.functions" :key="fn.name">
          <div class="func-sig">
            <span class="func-name">{{ fn.name }}</span>
            <span class="func-params">({{ fn.params }})</span>
            <span class="func-returns" v-if="fn.returns">→ {{ fn.returns }}</span>
          </div>
          <div class="func-desc">{{ fn.desc }}</div>
        </div>
      </div>
    </div>

    <!-- Validation Rules -->
    <div class="card border-green">
      <h3>Validation Rules (server-side)</h3>
      <p style="color: #8b949e; margin-bottom: 1rem; font-size: 0.85rem;">
        Enforced by york-data, not the LLM. Bad data gets rejected with a clear error message.
      </p>
      <div class="validation-list">
        <div class="validation-item" v-for="v in validations" :key="v.field">
          <span class="validation-field">{{ v.field }}</span>
          <span class="validation-rule">{{ v.rule }}</span>
        </div>
      </div>
    </div>

    <!-- Gotchas Eliminated -->
    <div class="card border-green">
      <h3>Problems This Eliminates</h3>
      <div class="gotcha-list">
        <div class="gotcha" v-for="g in gotchas" :key="g">
          <span class="gotcha-icon">✕</span> {{ g }}
        </div>
      </div>
    </div>

    <!-- What's NOT in MVP -->
    <div class="card">
      <h3>What's NOT in MVP</h3>
      <div class="not-mvp-list">
        <div class="not-mvp" v-for="n in notMvp" :key="n">→ {{ n }}</div>
      </div>
    </div>

    <!-- Migration Strategy -->
    <div class="card border-yellow">
      <h3>Migration from Google Sheets</h3>
      <div class="migration-steps">
        <div class="migration-step" v-for="(s, i) in migration" :key="i">
          <span class="step-num">{{ i + 1 }}</span>
          <span class="step-text">{{ s }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const structure = [
  { path: 'york-data/', desc: '' },
  { path: '  package.json', desc: '' },
  { path: '  src/index.js', desc: 'MCP server entry, registers all domains' },
  { path: '  src/db.js', desc: 'Database init, migration runner' },
  { path: '  src/domains/consumption.js', desc: 'Food/drink logging' },
  { path: '  src/domains/daily.js', desc: 'Daily metrics (weight, sleep, etc.)' },
  { path: '  src/domains/workouts.js', desc: 'Workout sessions + exercises' },
  { path: '  src/domains/cannabis.js', desc: 'Cannabis session tracking' },
  { path: '  src/domains/observations.js', desc: 'Bede\'s findings' },
  { path: '  src/domains/cross-domain.js', desc: 'Aggregation queries' },
  { path: '  migrations/001-initial.sql', desc: 'All MVP tables' },
]

const tables = [
  {
    name: 'consumption',
    desc: 'Food/drink items. Highest frequency, most error-prone domain.',
    columns: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'date', type: 'TEXT NOT NULL', note: 'YYYY-MM-DD' },
      { name: 'time', type: 'TEXT', note: 'HH:MM or null' },
      { name: 'item', type: 'TEXT NOT NULL', note: 'what was consumed' },
      { name: 'quantity', type: 'TEXT', note: '"2 slices", "1 bowl"' },
      { name: 'calories', type: 'INTEGER', note: 'estimated or known' },
      { name: 'protein', type: 'INTEGER', note: 'grams' },
      { name: 'notes', type: 'TEXT', note: '' },
      { name: 'created_at', type: 'TEXT', note: 'ISO timestamp, auto' },
    ],
  },
  {
    name: 'daily_metrics',
    desc: 'One row per day. Upsert pattern — partial updates merge.',
    columns: [
      { name: 'date', type: 'TEXT PK', note: 'YYYY-MM-DD, one row per day' },
      { name: 'weight', type: 'REAL', note: 'lbs' },
      { name: 'calories_total', type: 'INTEGER', note: 'daily total' },
      { name: 'protein_total', type: 'INTEGER', note: 'daily total grams' },
      { name: 'kitchen_closed', type: 'TEXT', note: 'time or null' },
      { name: 'wake_up', type: 'TEXT', note: 'time or null' },
      { name: 'bedtime', type: 'TEXT', note: 'time or null' },
      { name: 'sleep_hrs', type: 'REAL', note: '' },
      { name: 'energy', type: 'INTEGER', note: '1-5' },
      { name: 'updated_at', type: 'TEXT', note: 'auto-updated' },
    ],
  },
  {
    name: 'workouts',
    desc: 'Workout session summaries.',
    columns: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'date', type: 'TEXT NOT NULL', note: 'YYYY-MM-DD' },
      { name: 'type', type: 'TEXT', note: 'gym, bodyweight, swimming' },
      { name: 'location', type: 'TEXT', note: 'home, gym name' },
      { name: 'duration_min', type: 'INTEGER', note: '' },
      { name: 'notes', type: 'TEXT', note: '' },
      { name: 'created_at', type: 'TEXT', note: '' },
    ],
  },
  {
    name: 'exercises',
    desc: 'Individual exercises within a workout.',
    columns: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'workout_id', type: 'INTEGER FK', note: '→ workouts.id' },
      { name: 'exercise', type: 'TEXT NOT NULL', note: 'exercise name' },
      { name: 'sets', type: 'INTEGER', note: '' },
      { name: 'reps', type: 'INTEGER', note: '' },
      { name: 'weight', type: 'REAL', note: 'lbs, null for bodyweight' },
      { name: 'notes', type: 'TEXT', note: '' },
    ],
  },
  {
    name: 'cannabis_sessions',
    desc: 'Cannabis usage tracking.',
    columns: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'date', type: 'TEXT NOT NULL', note: 'YYYY-MM-DD' },
      { name: 'time', type: 'TEXT', note: 'HH:MM' },
      { name: 'session_number', type: 'INTEGER', note: 'nth of the day' },
      { name: 'created_at', type: 'TEXT', note: '' },
    ],
  },
  {
    name: 'observations',
    desc: 'Bede\'s structured findings from analysis.',
    columns: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'domain', type: 'TEXT NOT NULL', note: 'consumption, routing, etc.' },
      { name: 'type', type: 'TEXT NOT NULL', note: 'correction, error, frustration, pattern' },
      { name: 'summary', type: 'TEXT NOT NULL', note: 'one-line finding' },
      { name: 'evidence', type: 'TEXT', note: 'supporting detail' },
      { name: 'severity', type: 'TEXT', note: 'low, medium, high' },
      { name: 'suggested_action', type: 'TEXT', note: '' },
      { name: 'status', type: 'TEXT DEFAULT \'open\'', note: 'open, addressed, dismissed, in-progress' },
      { name: 'resolution', type: 'TEXT', note: 'what actually happened' },
      { name: 'created_at', type: 'TEXT', note: '' },
      { name: 'updated_at', type: 'TEXT', note: '' },
    ],
  },
]

const domains = [
  {
    name: 'Consumption',
    icon: '🍎',
    color: 'green',
    desc: 'Primary consumer: Wynn. Highest frequency, most error-prone domain.',
    functions: [
      { name: 'log_food', params: 'date, item, quantity?, calories?, protein?, time?, notes?', desc: 'Append a food/drink item' },
      { name: 'get_consumption', params: 'date', returns: 'items[]', desc: 'All items for a date' },
      { name: 'get_consumption_range', params: 'start_date, end_date', returns: 'items[]', desc: 'Items across date range' },
      { name: 'update_consumption', params: 'id, fields...', desc: 'Fix a specific entry' },
      { name: 'delete_consumption', params: 'id', desc: 'Remove a mis-logged entry' },
    ],
  },
  {
    name: 'Daily Metrics',
    icon: '📊',
    color: 'blue',
    desc: 'Primary consumer: Wynn. One row per day, upsert pattern — partial updates merge with existing.',
    functions: [
      { name: 'log_daily', params: 'date, weight?, calories_total?, protein_total?, kitchen_closed?, wake_up?, bedtime?, sleep_hrs?, energy?', desc: 'Upsert daily metrics. Partial updates merge.' },
      { name: 'get_daily', params: 'date', returns: 'metrics', desc: 'Single day\'s metrics' },
      { name: 'get_daily_range', params: 'start_date, end_date', returns: 'metrics[]', desc: 'Range of daily metrics' },
      { name: 'get_weight_trend', params: 'days', returns: 'weights[] + moving_avg_7d', desc: 'Weight entries with 7-day moving average' },
    ],
  },
  {
    name: 'Workouts',
    icon: '💪',
    color: 'blue',
    desc: 'Primary consumer: Wynn. Workout summaries and individual exercise details.',
    functions: [
      { name: 'log_workout', params: 'date, type?, location?, duration_min?, notes?', returns: 'id', desc: 'Add workout summary, returns ID for exercise logging' },
      { name: 'log_exercises', params: 'workout_id, exercises[{exercise, sets, reps, weight?, notes?}]', returns: 'count', desc: 'Add exercises to a workout' },
      { name: 'get_workouts', params: 'start_date?, end_date?', returns: 'workouts[]', desc: 'Workout history with exercises included' },
      { name: 'get_last_workout', params: '', returns: 'workout', desc: 'Most recent workout with exercises' },
    ],
  },
  {
    name: 'Cannabis',
    icon: '🌿',
    color: 'yellow',
    desc: 'Primary consumers: Wynn (logging), York (gate checks).',
    functions: [
      { name: 'log_cannabis', params: 'date, time?, session_number?', returns: 'id', desc: 'Record a session' },
      { name: 'get_cannabis_sessions', params: 'date?', returns: 'sessions[]', desc: 'Sessions for a date (default today)' },
      { name: 'get_cannabis_history', params: 'days', returns: 'days[] + avg_per_day + total', desc: 'Frequency and pattern data' },
    ],
  },
  {
    name: 'Observations',
    icon: '🦉',
    color: 'purple',
    desc: 'Primary consumer: Bede. Structured findings from transcript analysis.',
    functions: [
      { name: 'log_observation', params: 'domain, type, summary, evidence?, severity?, suggested_action?', returns: 'id', desc: 'Record a finding' },
      { name: 'get_observations', params: 'domain?, type?, status?, since?', returns: 'observations[]', desc: 'Query with filters' },
      { name: 'update_observation', params: 'id, status?, resolution?', desc: 'Mark as addressed, dismissed, or in-progress' },
    ],
  },
  {
    name: 'Cross-Domain',
    icon: '🔗',
    color: 'purple',
    desc: 'Aggregation queries across all tables. Used by Dagr (morning brief) and Bede (pattern analysis).',
    functions: [
      { name: 'get_day_summary', params: 'date', returns: 'everything', desc: 'All data for a date across all domains' },
      { name: 'get_trends', params: 'days', returns: 'combined view', desc: 'Weight, calories, protein, cannabis, workouts over N days' },
    ],
  },
]

const validations = [
  { field: 'Dates', rule: 'Must be YYYY-MM-DD. Reject anything else.' },
  { field: 'Calories', rule: '0–8000 range.' },
  { field: 'Protein', rule: '0–500g range.' },
  { field: 'Weight', rule: '100–400 lbs range.' },
  { field: 'Energy', rule: '1–5 integer.' },
  { field: 'Sleep', rule: '0–24 hours.' },
  { field: 'Session number', rule: '1–20 range.' },
  { field: 'Future dates', rule: 'No future dates for consumption/workouts.' },
]

const gotchas = [
  'Wrong column offset — LLM writes calories to protein column or vice versa',
  'Month sheet name — LLM uses "Feb Consumption" instead of "February Consumption"',
  'Range calculation — LLM guesses next empty row, sometimes overwrites existing data',
  'Parameter naming — data vs values, spreadsheet_id vs spreadsheetId',
  'Duplicate entries — LLM doesn\'t check if entry already exists before writing',
  'Dashboard row drift — LLM counts rows wrong when dashboard has gaps',
  'Date format chaos — "Mar 5" vs "March 5" vs "2026-03-05" vs "Mar 5, 2026"',
  'Unreasonable values — service enforces bounds (0-8000 cal, 0-500g protein)',
]

const notMvp = [
  'Google Sheets sync/export (future: write-only mirror for James to view)',
  'Task/chore storage (lives in Google Tasks)',
  'Home project tracking (deferred)',
  'Flags from agents ("Flag for Bede" global skill — add when needed)',
  'Backup automation (manual for now: copy the .db file)',
  'Auth/permissions (all agents get all functions)',
  'Schema versioning UI (migrations handle it)',
]

const migration = [
  'Build york-data with empty tables',
  'Write a one-time migration script that reads Sheets via mcporter and inserts into york-data',
  'Run it once',
  'Switch agents to york-data',
  'Google Sheets becomes read-only archive',
]
</script>

<style scoped>
.structure {
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
  font-family: monospace;
  font-size: 0.85rem;
}

.struct-item {
  display: flex;
  gap: 1rem;
  padding: 0.2rem 0.5rem;
}

.struct-path {
  color: #79c0ff;
  min-width: 280px;
}

.struct-desc {
  color: #8b949e;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

.table-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.table-card {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 6px;
  padding: 1rem;
}

.table-name {
  color: #d2a8ff;
  font-family: monospace;
  font-weight: 700;
  font-size: 1rem;
  margin-bottom: 0.25rem;
}

.table-desc {
  color: #8b949e;
  font-size: 0.8rem;
  margin-bottom: 0.75rem;
}

.col-list {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.col-item {
  display: flex;
  gap: 0.75rem;
  align-items: baseline;
  padding: 0.15rem 0;
  font-size: 0.8rem;
}

.col-name {
  color: #79c0ff;
  min-width: 120px;
}

.col-type {
  color: #d29922;
  min-width: 140px;
  font-size: 0.75rem;
}

.col-note {
  color: #8b949e;
  font-size: 0.75rem;
}

.func-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.func-item {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 4px;
  padding: 0.75rem;
}

.func-sig {
  font-family: monospace;
  font-size: 0.85rem;
  margin-bottom: 0.25rem;
}

.func-name { color: #d2a8ff; font-weight: 600; }
.func-params { color: #c9d1d9; }
.func-returns { color: #3fb950; }

.func-desc {
  color: #8b949e;
  font-size: 0.8rem;
}

.validation-list {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.validation-item {
  display: flex;
  gap: 1rem;
  padding: 0.4rem 0.65rem;
  background: #0d1117;
  border-radius: 4px;
  font-size: 0.85rem;
}

.validation-field {
  color: #f0f6fc;
  font-weight: 500;
  min-width: 120px;
}

.validation-rule {
  color: #8b949e;
}

.gotcha-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.gotcha {
  color: #c9d1d9;
  font-size: 0.85rem;
  padding: 0.5rem;
  background: #0d1117;
  border-radius: 4px;
}

.gotcha-icon {
  color: #f85149;
  font-weight: 700;
  margin-right: 0.5rem;
}

.not-mvp-list {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.not-mvp {
  color: #8b949e;
  font-size: 0.85rem;
  padding: 0.35rem 0;
}

.migration-steps {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.migration-step {
  display: flex;
  gap: 0.75rem;
  align-items: center;
  padding: 0.5rem;
  background: #0d1117;
  border-radius: 4px;
}

.step-num {
  background: #d29922;
  color: #0d1117;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.8rem;
  flex-shrink: 0;
}

.step-text {
  color: #c9d1d9;
  font-size: 0.85rem;
}

.border-green { border-color: #3fb950; }
.border-blue { border-color: #58a6ff; }
.border-yellow { border-color: #d29922; }
.border-purple { border-color: #bc8cff; }
</style>
