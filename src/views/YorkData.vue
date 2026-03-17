<template>
  <div>
    <div class="card">
      <h2>york-data Service Layer</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Custom MCP server sitting between agents and data. Agents call typed functions with named parameters.
        The service validates, enforces schema, and handles all DB operations. LLMs never see raw queries or column indices.
      </p>
      <p style="color: #c9d1d9; margin-bottom: 0; line-height: 1.6; font-size: 0.9rem; border-left: 3px solid #d29922; padding-left: 1rem;">
        <strong style="color: #d29922;">Stack:</strong> Node.js MCP server + <code style="color: #d29922;">node:sqlite</code> (built-in, zero external deps for DB, WAL mode, single file).
        <code style="color: #d29922;">@modelcontextprotocol/sdk</code> + <code style="color: #d29922;">zod</code> for tool registration and validation.
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
  { path: '  package.json', desc: '@modelcontextprotocol/sdk + zod (ESM)' },
  { path: '  API.md', desc: 'Tool reference for all agents' },
  { path: '  src/index.js', desc: 'MCP server entry, registers all domains, stdio transport' },
  { path: '  src/db.js', desc: 'DatabaseSync init, WAL mode, migration runner' },
  { path: '  src/domains/system.js', desc: 'ping (health check with row counts)' },
  { path: '  src/domains/consumption.js', desc: 'Food, drink, supplements (6 tools)' },
  { path: '  src/domains/daily.js', desc: 'Daily metrics — upsert pattern (4 tools)' },
  { path: '  src/domains/workouts.js', desc: 'Exercise catalog + workout sessions (6 tools)' },
  { path: '  src/domains/cross.js', desc: 'Cross-domain aggregation queries (2 tools)' },
  { path: '  src/domains/programs.js', desc: 'Workout programs + daily prescriptions (8 tools)' },
  { path: '  src/domains/tasks.js', desc: 'Hild task definitions, daily planning, completion tracking, interval tasks, followup tasks (10 tools)' },
  { path: '  migrations/001-initial-schema.sql', desc: 'All MVP tables with constraints + indexes' },
  { path: '  migrations/002-programs.sql', desc: 'Programs + program_days tables' },
  { path: '  migrations/003-tasks.sql', desc: 'Task definitions + daily log tables' },
  { path: '  migrations/004-task-config.sql', desc: 'Task config key-value store' },
  { path: '  migrations/005-interval-tasks.sql', desc: 'Interval tasks: interval_days, window_days, last_completed_date columns' },
  { path: '  migrations/006-followup-tasks.sql', desc: 'Followup tasks: followup_task_id, followup_delay columns' },
  { path: '  scripts/migrate-sheets.js', desc: 'One-time Google Sheets → SQLite migration' },
]

const tables = [
  {
    name: 'consumption',
    desc: 'Food, drinks, and supplements.',
    columns: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'date', type: 'TEXT NOT NULL', note: 'YYYY-MM-DD' },
      { name: 'time', type: 'TEXT', note: 'HH:MM or null' },
      { name: 'item', type: 'TEXT NOT NULL', note: 'what was consumed' },
      { name: 'quantity', type: 'TEXT', note: '"2 slices", "1 bowl"' },
      { name: 'calories', type: 'INTEGER', note: 'estimated or known' },
      { name: 'protein', type: 'INTEGER', note: 'grams' },
      { name: 'type', type: 'TEXT DEFAULT \'food\'', note: 'food, drink, supplement, other' },
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
    desc: 'Reusable exercise catalog. Names are unique (case-insensitive).',
    columns: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'name', type: 'TEXT NOT NULL UNIQUE', note: 'e.g. "bench press", "barbell squat"' },
      { name: 'category', type: 'TEXT', note: 'chest, legs, back, arms, core, cardio' },
      { name: 'notes', type: 'TEXT', note: '' },
      { name: 'created_at', type: 'TEXT', note: 'auto' },
    ],
  },
  {
    name: 'workout_exercises',
    desc: 'Instances of exercises performed in a workout. FK-validated against both workouts and exercise catalog.',
    columns: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'workout_id', type: 'INTEGER FK', note: '→ workouts.id (CASCADE delete)' },
      { name: 'exercise_id', type: 'INTEGER FK', note: '→ exercises.id (validated before insert)' },
      { name: 'sets', type: 'INTEGER', note: '' },
      { name: 'reps', type: 'INTEGER', note: '' },
      { name: 'weight', type: 'REAL', note: 'lbs, null for bodyweight' },
      { name: 'notes', type: 'TEXT', note: '' },
    ],
  },
  {
    name: 'programs',
    desc: 'Multi-week training programs. Only one can be active at a time.',
    columns: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'name', type: 'TEXT NOT NULL', note: 'e.g. "Bodyweight Foundation - Weeks 1-4"' },
      { name: 'start_date', type: 'TEXT NOT NULL', note: 'YYYY-MM-DD' },
      { name: 'end_date', type: 'TEXT NOT NULL', note: 'YYYY-MM-DD (calculated)' },
      { name: 'weeks', type: 'INTEGER NOT NULL', note: '1-52' },
      { name: 'status', type: 'TEXT NOT NULL', note: 'draft, active, completed, cancelled' },
      { name: 'goals', type: 'TEXT', note: 'free text' },
      { name: 'notes', type: 'TEXT', note: '' },
      { name: 'created_at', type: 'TEXT', note: 'auto' },
    ],
  },
  {
    name: 'program_days',
    desc: 'Daily workout prescriptions within a program.',
    columns: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'program_id', type: 'INTEGER FK', note: '→ programs.id' },
      { name: 'week_number', type: 'INTEGER NOT NULL', note: '1-based' },
      { name: 'day_number', type: 'INTEGER NOT NULL', note: '1=Mon, 7=Sun' },
      { name: 'date', type: 'TEXT NOT NULL', note: 'YYYY-MM-DD (future dates allowed)' },
      { name: 'type', type: 'TEXT NOT NULL', note: 'strength, cardio, active_recovery, rest, flexibility' },
      { name: 'plan', type: 'TEXT NOT NULL', note: 'JSON string — workout prescription' },
      { name: 'notes', type: 'TEXT', note: '' },
      { name: 'swapped_with_date', type: 'TEXT', note: 'original date if weather-swapped' },
      { name: 'created_at', type: 'TEXT', note: 'auto' },
    ],
  },

  {
    name: 'task_definitions',
    desc: 'Hild task definitions — permanent habits, recurring chores, growth candidates, ad-hoc tasks. Supports interval-based recurrence (quarterly, annual) and followup task chains (laundry → put away).',
    fields: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'title', type: 'TEXT NOT NULL', note: 'Task name' },
      { name: 'stack', type: 'TEXT NOT NULL', note: 'morning, after-work, after-dinner, weekend' },
      { name: 'type', type: 'TEXT NOT NULL', note: 'permanent, recurring, candidate, adhoc' },
      { name: 'effort_minutes', type: 'INTEGER', note: '1-60. >60 rejected.' },
      { name: 'recur_pattern', type: 'TEXT', note: 'daily, weekdays, weekends, weekly:<day>, biweekly:<day>, monthly:<day>, quarterly, biannual, annual, on_demand' },
      { name: 'position', type: 'INTEGER', note: 'Order within stack (lower = first)' },
      { name: 'active', type: 'INTEGER', note: '1=active, 0=paused/completed' },
      { name: 'deadline', type: 'TEXT', note: 'YYYY-MM-DD, for adhoc tasks' },
      { name: 'notes', type: 'TEXT', note: '' },
      { name: 'google_task_id', type: 'TEXT', note: 'FK to Google Tasks for sync' },
      { name: 'created_at', type: 'TEXT', note: 'auto' },
      { name: 'completed_at', type: 'TEXT', note: 'When adhoc task was completed' },
      { name: 'interval_days', type: 'INTEGER', note: 'Days between completions (14=biweekly, 90=quarterly, etc.)' },
      { name: 'window_days', type: 'INTEGER', note: 'Days before due to start surfacing (default 7)' },
      { name: 'last_completed_date', type: 'TEXT', note: 'YYYY-MM-DD of most recent completion' },
      { name: 'followup_task_id', type: 'INTEGER FK', note: '→ task_definitions.id. Auto-schedule on completion.' },
      { name: 'followup_delay', type: 'TEXT', note: 'same_day or next_day' },
    ],
  },
  {
    name: 'task_daily_log',
    desc: 'Per-date scheduling and completion tracking. Written by prepare_daily_stacks, updated by log_task_completion.',
    fields: [
      { name: 'id', type: 'INTEGER PK', note: 'auto' },
      { name: 'date', type: 'TEXT NOT NULL', note: 'YYYY-MM-DD' },
      { name: 'task_definition_id', type: 'INTEGER FK', note: '→ task_definitions.id' },
      { name: 'stack', type: 'TEXT NOT NULL', note: 'Stack at time of scheduling' },
      { name: 'scheduled', type: 'INTEGER', note: '1=placed on stack, 0=skipped' },
      { name: 'completed', type: 'INTEGER', note: '0=not done, 1=done' },
      { name: 'skipped_reason', type: 'TEXT', note: 'Why skipped' },
      { name: 'created_at', type: 'TEXT', note: 'auto' },
    ],
  },
]

const domains = [
  {
    name: 'Consumption',
    icon: '🍎',
    color: 'green',
    desc: 'Primary consumer: Wynn. Food, drinks, and supplements.',
    functions: [
      { name: 'log_food', params: 'date, item, quantity?, calories?, protein?, time?, type?, notes?', desc: 'Append a consumption item (food, drink, supplement, or other)' },
      { name: 'get_consumption', params: 'date, type?', returns: 'items[]', desc: 'All items for a date, optionally filtered by type' },
      { name: 'get_consumption_range', params: 'start_date, end_date, type?', returns: 'items[]', desc: 'Items across date range' },
      { name: 'update_consumption', params: 'id, fields...', desc: 'Fix a specific entry' },
      { name: 'delete_consumption', params: 'id', desc: 'Remove a mis-logged entry' },
      { name: 'get_type_history', params: 'days', returns: 'days[] + avg_per_day + total', desc: 'Convenience query: frequency and patterns for a given consumption type' },
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
    name: 'Exercise Catalog',
    icon: '📋',
    color: 'blue',
    desc: 'Reusable exercise definitions. Must create exercises here before referencing them in workouts.',
    functions: [
      { name: 'create_exercise', params: 'name, category?, notes?', returns: 'id', desc: 'Add exercise to catalog. Names are unique (case-insensitive dedup).' },
      { name: 'get_exercises', params: 'category?', returns: 'exercises[]', desc: 'List catalog entries. Filter by category.' },
    ],
  },
  {
    name: 'Workouts',
    icon: '💪',
    color: 'blue',
    desc: 'Primary consumer: Wynn. Workout sessions with FK-validated exercise references.',
    functions: [
      { name: 'log_workout', params: 'date, type?, location?, duration_min?, notes?', returns: 'id', desc: 'Log a workout session, returns ID for adding exercises' },
      { name: 'log_workout_exercises', params: 'workout_id, exercises[{exercise_id, sets?, reps?, weight?, notes?}]', returns: 'count', desc: 'Add exercises to a workout. Validates workout_id and all exercise_ids exist before inserting.' },
      { name: 'get_workouts', params: 'start_date?, end_date?', returns: 'workouts[]', desc: 'Workout history with joined exercise names and categories' },
      { name: 'get_last_workout', params: '', returns: 'workout', desc: 'Most recent workout with exercises' },
    ],
  },

  {
    name: 'Programs',
    icon: '📅',
    color: 'yellow',
    desc: 'Primary consumer: Wynn. Multi-week training programs and daily workout prescriptions. Weather-aware swapping, adherence tracking.',
    functions: [
      { name: 'create_program', params: 'name, start_date, weeks, goals?, notes?', returns: 'id, start_date, end_date', desc: 'Create and activate a program. Auto-completes any existing active program. Calculates end_date.' },
      { name: 'add_program_days', params: 'program_id, days[{week_number, day_number, date, type, plan, notes?}]', returns: 'added', desc: 'Bulk insert daily workout prescriptions. Validates dates, types, JSON plans.' },
      { name: 'get_active_program', params: 'include_days?', returns: 'program + days[]', desc: 'Get the active program with all its days. include_days defaults to true.' },
      { name: 'get_program_day', params: 'date?', returns: 'day', desc: 'Get today\'s (or specified date\'s) workout from the active program.' },
      { name: 'swap_program_days', params: 'day_id_1, day_id_2, reason?', returns: 'swapped info', desc: 'Swap type/plan/notes between two days. Records swap history. For weather swaps.' },
      { name: 'update_program_day', params: 'id, type?, plan?, notes?', returns: 'updated', desc: 'Partial update of a program day. Validates JSON if plan provided.' },
      { name: 'complete_program', params: 'program_id', returns: 'completed', desc: 'Set program status to completed.' },
      { name: 'get_program_summary', params: 'program_id', returns: 'program + adherence stats', desc: 'Cross-references workouts table for adherence: training_days, workouts_logged, adherence_pct.' },
    ],
  },
  {
    name: 'Tasks',
    icon: '✅',
    color: 'green',
    desc: 'Primary consumer: Hild. Task definitions, daily stack planning, completion tracking. Intelligence layer — Google Tasks is the execution layer.',
    functions: [
      { name: 'create_task_definition', params: 'title, stack, type?, recur_pattern?, interval_days?, window_days?, followup_task_id?, followup_delay?, ...', returns: 'id, title, stack, type', desc: 'Create a task. Supports interval patterns (quarterly, annual) and followup chains. 60m+ rejected.' },
      { name: 'update_task_definition', params: 'id, fields...', returns: 'updated', desc: 'Partial update. All fields from create plus active, google_task_id, interval/followup fields.' },
      { name: 'delete_task_definition', params: 'id, hard?', returns: 'deactivated or deleted', desc: 'Soft-delete (default) or hard delete with history.' },
      { name: 'get_task_definitions', params: 'stack?, type?, active?', returns: 'tasks[]', desc: 'List definitions filtered by stack, type, active status.' },
      { name: 'prepare_daily_stacks', params: 'date?, write_log?', returns: 'stacks{}, effort{}, total', desc: 'Daily plan. Matches recurrence to date, includes interval tasks in window. write_log=1 records to daily log.' },
      { name: 'log_task_completion', params: 'task_definition_id, date?', returns: 'completed + followup_scheduled?', desc: 'Mark done. Adhoc auto-deactivate. Updates last_completed_date. Triggers followup task if configured.' },
      { name: 'get_task_completion', params: 'date?, start_date?, end_date?, stack?', returns: 'entries[] + summary', desc: 'Completion status with scheduled/completed/rate.' },
      { name: 'get_stack_stats', params: 'days?, stack?', returns: 'stats per stack', desc: 'Completion rates over N days.' },
      { name: 'get_interval_tasks', params: 'date?, include_not_due?', returns: 'tasks[] + summary', desc: 'All interval tasks with due status (upcoming/due/overdue). Used by planner.' },
      { name: 'sync_stack', params: 'stack, date?', returns: 'stack plan + sync stats', desc: 'All-in-one: sync completions (missing = done), compute stack, reset Google Tasks, create missing. Handles followups.' },
      { name: 'sync_completions', params: 'date?', returns: 'synced count', desc: 'Sync all Google Tasks completions. Detects cleared tasks (missing = done). Triggers followups.' },
      { name: 'create_google_task', params: '(same as create_task_definition)', returns: 'id + google_task_id', desc: 'Create in york-data AND Google Tasks in one call.' },
    ],
  },
  {
    name: 'Cross-Domain',
    icon: '🔗',
    color: 'purple',
    desc: 'Aggregation queries across all tables. Used by Dagr (morning brief) and Bede (pattern analysis).',
    functions: [
      { name: 'get_day_summary', params: 'date', returns: 'consumption + metrics + workouts', desc: 'All data for a date across all domains' },
      { name: 'get_range_summary', params: 'start_date, end_date', returns: 'metrics[] + consumption_by_day[] + workouts[]', desc: 'Multi-day overview with per-day totals. For weekly reviews and pattern analysis.' },
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
  { field: 'Type', rule: 'Must be: food, drink, supplement, or other.' },
  { field: 'Future dates', rule: 'No future dates for consumption/workouts. Program days allow future dates (planning data).' },
  { field: 'Program status', rule: 'Must be: draft, active, completed, or cancelled.' },
  { field: 'Day type', rule: 'Must be: strength, cardio, active_recovery, rest, or flexibility.' },
  { field: 'Plan JSON', rule: 'Must be valid JSON string.' },
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
  'Build queue / pipeline for Offa (needed soon — ad-hoc for now)',
  'Observations domain (add when Bede is built)',
  'Flags from agents ("Flag for Bede" global skill — add when needed)',
  'Backup automation (manual for now: copy the .db file)',
  'Auth/permissions (all agents get all functions)',
  'Schema versioning UI (migrations handle it)',
]

const migration = [
  '✅ Built york-data MCP server with all tables and 27 tools (19 MVP + 8 programs)',
  '✅ Wrote migrate-sheets.js — reads all Sheets via mcporter, writes to SQLite',
  '✅ Migrated: 69 daily metrics, 743 consumption entries, 5 workouts, 2 exercises',
  '✅ Added programs domain: 8 tools for multi-week workout programming',
  '⬜ Switch agents to york-data (build agent-specific usage skills)',
  '⬜ Google Sheets becomes read-only archive',
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
