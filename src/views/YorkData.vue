<template>
  <div>
    <div class="card">
      <h2>york-data Service Layer</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Custom MCP server sitting between agents and data. Agents call typed functions with named parameters.
        The service validates, enforces schema, and handles all DB operations. LLMs never see raw queries or column indices.
      </p>
      <p style="color: #c9d1d9; margin-bottom: 0; line-height: 1.6; font-size: 0.9rem; border-left: 3px solid #d29922; padding-left: 1rem;">
        <strong style="color: #d29922;">Why MCP?</strong> mcporter is OpenClaw's CLI for calling MCP servers.
        Agents already call <code style="color: #d29922;">mcporter call google-sheets.update_cells ...</code> today.
        york-data replaces that with <code style="color: #d29922;">mcporter call york-data.log_food ...</code>.
        Same tooling, same call pattern, completely different reliability.
      </p>
    </div>

    <!-- Function Inventory -->
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

    <!-- Current Schema -->
    <div class="card">
      <h3>Current Data Schemas (from Google Sheets audit)</h3>
      <p style="color: #8b949e; margin-bottom: 1rem; font-size: 0.85rem;">
        Audited from every mcporter call across all current skills. These are the actual columns York reads/writes today.
      </p>

      <div class="schema-list">
        <div class="schema-item" v-for="s in schemas" :key="s.name">
          <div class="schema-name">{{ s.name }}</div>
          <div class="schema-cols">{{ s.columns }}</div>
          <div class="schema-users">Used by: {{ s.usedBy }}</div>
        </div>
      </div>
    </div>

    <!-- Gotchas Eliminated -->
    <div class="card border-green">
      <h3>Problems the Service Layer Eliminates</h3>
      <div class="gotcha-list">
        <div class="gotcha" v-for="g in gotchas" :key="g">
          <span class="gotcha-icon">✕</span> {{ g }}
        </div>
      </div>
    </div>

    <!-- What Stays Outside -->
    <div class="card">
      <h3>What Stays Outside york-data</h3>
      <p style="color: #8b949e; margin-bottom: 1rem; font-size: 0.85rem;">
        Not everything moves. External services and file-based data stay as-is.
      </p>
      <div class="external-list">
        <div class="external-item" v-for="e in external" :key="e.name">
          <span class="external-name">{{ e.icon }} {{ e.name }}</span>
          <span class="external-reason">{{ e.reason }}</span>
        </div>
      </div>
    </div>

    <!-- Task Tracking (Draft Notes) -->
    <div class="card border-yellow">
      <h3>📋 Draft: Task & Observation Storage</h3>
      <p style="color: #d29922; font-size: 0.85rem; margin-bottom: 1rem; font-style: italic;">
        Early thinking — not committed to this design yet. Capturing for discussion.
      </p>

      <div class="draft-section">
        <h4>Two separate concerns currently conflated:</h4>
        <div class="draft-comparison">
          <div class="draft-col">
            <div class="draft-col-header">Tasks (explicit)</div>
            <p>Things James asks York to do, or York commits to doing. Currently lost in conversation or forgotten.</p>
            <p><strong>Possible:</strong> DB table via york-data. <code>add_task()</code>, <code>get_tasks(status?)</code>, <code>update_task(id, status)</code>. Checked during heartbeats. Source tracked (james / york / observer).</p>
          </div>
          <div class="draft-col">
            <div class="draft-col-header">Observations (implicit)</div>
            <p>Things the observation system notices: corrections, failures, friction patterns. Currently not captured at all.</p>
            <p><strong>Possible:</strong> DB records written by the observer script. Queried by Improvement Analyst. Recurring patterns surface as tasks if fixable.</p>
          </div>
        </div>
        <p style="color: #8b949e; font-size: 0.85rem; margin-top: 1rem;">
          Key principle: nothing depends on the LLM remembering to write to a file.
          The observer script writes automatically. Tasks are stored in the DB. Agents read what's there.
        </p>
      </div>
    </div>

    <!-- Open Questions -->
    <div class="card">
      <h3>Open Questions</h3>
      <div class="question-list">
        <div class="question" v-for="q in questions" :key="q">❓ {{ q }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
const domains = [
  {
    name: 'Consumption',
    icon: '🍎',
    color: 'green',
    desc: 'Highest frequency, most error-prone domain. Currently the #1 source of corrections from James.',
    functions: [
      { name: 'log_food', params: 'date, time, item, quantity, calories, protein, notes?', desc: 'Append to consumption log' },
      { name: 'get_consumption', params: 'date', returns: 'items[]', desc: 'All items for a date' },
      { name: 'get_consumption_range', params: 'start_date, end_date', returns: 'items[]', desc: 'Items across date range' },
      { name: 'delete_consumption', params: 'id', desc: 'Remove a mis-logged entry' },
      { name: 'update_consumption', params: 'id, fields...', desc: 'Fix a specific entry' },
    ],
  },
  {
    name: 'Daily Metrics',
    icon: '📊',
    color: 'blue',
    desc: 'Dashboard daily log: weight, calories, sleep, energy. One row per day, upsert pattern.',
    functions: [
      { name: 'log_daily', params: 'date, weight?, calories?, protein?, kitchen_closed?, wake_up?, bedtime?, sleep_hrs?, energy?', desc: 'Upsert daily metrics row' },
      { name: 'get_daily', params: 'date', returns: 'metrics', desc: 'Single day\'s metrics' },
      { name: 'get_daily_range', params: 'start_date, end_date', returns: 'metrics[]', desc: 'Range of daily metrics' },
      { name: 'get_weight_trend', params: 'days', returns: 'weights[] + moving_avg', desc: 'Weight entries with moving average' },
    ],
  },
  {
    name: 'Workouts',
    icon: '💪',
    color: 'blue',
    desc: 'Workout summaries and individual exercise details.',
    functions: [
      { name: 'log_workout', params: 'date, type, location, duration, notes?', desc: 'Add workout summary' },
      { name: 'log_exercises', params: 'date, exercises[{name, sets, reps, weight, notes?}]', desc: 'Add exercise details' },
      { name: 'get_workouts', params: 'start_date?, end_date?', returns: 'workouts[]', desc: 'Workout history' },
      { name: 'get_last_workout', params: '', returns: 'workout', desc: 'Most recent workout date and details' },
    ],
  },
  {
    name: 'Cannabis',
    icon: '🌿',
    color: 'yellow',
    desc: 'Currently embedded in consumption, could be its own domain for pattern analysis.',
    functions: [
      { name: 'log_cannabis', params: 'date, time, session_number', desc: 'Record a session' },
      { name: 'get_cannabis_sessions', params: 'date?', returns: 'sessions[]', desc: 'Sessions for a date' },
      { name: 'get_cannabis_history', params: 'days', returns: 'frequency + patterns', desc: 'Frequency and pattern data' },
    ],
  },
  {
    name: 'Chores & Home',
    icon: '🏠',
    color: 'yellow',
    desc: 'Chore tracking and home projects.',
    functions: [
      { name: 'get_chores', params: '', returns: 'chores[]', desc: 'All chores with frequency and last-completed' },
      { name: 'log_chore', params: 'task, completed_date', desc: 'Mark a chore done' },
      { name: 'get_chore_status', params: '', returns: 'due + overdue', desc: 'What\'s due, what\'s overdue' },
      { name: 'get_projects', params: '', returns: 'projects[]', desc: 'All projects with priority and status' },
      { name: 'update_project', params: 'task, fields...', desc: 'Update project status/priority' },
    ],
  },
  {
    name: 'Cross-Domain',
    icon: '🔗',
    color: 'purple',
    desc: 'Aggregation queries for weekly review, pattern analysis, and morning brief.',
    functions: [
      { name: 'get_trends', params: 'days, domains[]', returns: 'combined view', desc: 'Weight, calories, protein, workouts, cannabis over N days' },
      { name: 'get_day_summary', params: 'date', returns: 'everything', desc: 'All data for a specific day across domains' },
    ],
  },
]

const schemas = [
  { name: 'Dashboard (daily log)', columns: 'Date, Weight, Calories, Protein (g), Kitchen Closed, Wake Up, Bedtime, Sleep (hrs), Energy (1-5)', usedBy: 'york-consumption, nutrition-weight-cache, morning-orchestrator, weekly-checkin' },
  { name: '[Month] Consumption', columns: 'DateTime, Item, Quantity, Calories, Protein, Notes', usedBy: 'york-consumption, nutrition-summary, consumption-gap, meal-system, morning-brief' },
  { name: 'Workouts', columns: 'Date, Type, Location, Duration, Notes', usedBy: 'york-workout, weekly-checkin' },
  { name: 'Exercises', columns: 'Date, Exercise, Sets, Reps, Weight, Notes', usedBy: 'york-workout' },
  { name: 'Chores', columns: 'Task, Frequency, Last Completed, ...', usedBy: 'york-chores, morning-orchestrator' },
  { name: 'Home Projects', columns: 'Task, Zone(s), Priority, ...', usedBy: 'morning-brief' },
]

const gotchas = [
  'Wrong column offset — LLM writes calories to protein column or vice versa',
  'Month sheet name — LLM uses "Feb Consumption" instead of "February Consumption"',
  'Range calculation — LLM guesses next empty row, sometimes overwrites existing data',
  'Parameter naming — data vs values, spreadsheet_id vs spreadsheetId',
  'Duplicate entries — LLM doesn\'t check if entry already exists before writing',
  'Dashboard row drift — LLM counts rows wrong when dashboard has gaps',
  'Date format chaos — "Mar 5" vs "March 5" vs "2026-03-05" vs "Mar 5, 2026"',
  'Unreasonable values — service can enforce bounds (0-5000 cal, 0-500g protein)',
]

const external = [
  { icon: '📅', name: 'Google Calendar', reason: 'External source of truth. Read-only via existing MCP.' },
  { icon: '📁', name: 'Memory Files', reason: 'Daily context, agent communication. Markdown stays.' },
  { icon: '📖', name: 'D&D Wiki', reason: 'Markdown files managed by DnD agent.' },
  { icon: '📷', name: 'Cameras', reason: 'Hardware integration via ffmpeg scripts.' },
  { icon: '💬', name: 'Discord', reason: 'Message tool, stays as-is.' },
  { icon: '🖥️', name: 'Panel', reason: 'status.json file write for genmon.' },
]

const questions = [
  'DB engine: any lightweight embeddable DB works. Node built-in SQLite (node:sqlite) is interesting. Leave open until implementation.',
  'Should york-data handle auth/permissions? Or do all agents get all functions?',
  'How does schema evolution work? Adding a column to daily metrics, for example.',
  'Should there be a read-only mode for agents that only need to query, not write?',
  'Backup strategy: how often, where, automated?',
  'Data migration: one-time script from Sheets, or gradual dual-write period?',
  'Should Google Sheets stay as a sync target (write-only export for James to view)?',
]
</script>

<style scoped>
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

.schema-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.schema-item {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 4px;
  padding: 0.75rem;
}

.schema-name {
  color: #f0f6fc;
  font-weight: 600;
  font-size: 0.9rem;
  margin-bottom: 0.25rem;
}

.schema-cols {
  color: #c9d1d9;
  font-family: monospace;
  font-size: 0.8rem;
  margin-bottom: 0.25rem;
}

.schema-users {
  color: #8b949e;
  font-size: 0.75rem;
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

.external-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.external-item {
  display: flex;
  justify-content: space-between;
  padding: 0.5rem;
  background: #0d1117;
  border-radius: 4px;
  font-size: 0.85rem;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.external-name { color: #f0f6fc; font-weight: 500; }
.external-reason { color: #8b949e; }

.draft-section h4 {
  color: #8b949e;
  font-size: 0.85rem;
  margin-bottom: 1rem;
}

.draft-comparison {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

.draft-col {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1rem;
}

.draft-col-header {
  color: #d29922;
  font-weight: 700;
  margin-bottom: 0.5rem;
}

.draft-col p {
  color: #c9d1d9;
  font-size: 0.85rem;
  line-height: 1.5;
  margin-bottom: 0.5rem;
}

.draft-col code {
  color: #d29922;
  font-size: 0.8rem;
}

.question-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.question {
  color: #c9d1d9;
  font-size: 0.85rem;
  padding: 0.5rem;
  background: #0d1117;
  border-radius: 4px;
  line-height: 1.5;
}

.border-green { border-color: #3fb950; }
.border-blue { border-color: #58a6ff; }
.border-yellow { border-color: #d29922; }
.border-purple { border-color: #bc8cff; }

@media (max-width: 768px) {
  .draft-comparison { grid-template-columns: 1fr; }
}
</style>
