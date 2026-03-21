<template>
  <div>
    <!-- Overview -->
    <div class="card">
      <h2>📋 Task System Design</h2>
      <p class="subtitle">Hild's chore and task management system. Google Tasks is the execution layer (what James sees on his phone). york-data is the intelligence layer (definitions, recurrence, history, planning).</p>
      <p class="subtitle"><strong>Source code:</strong> <code>~/york-data/src/domains/tasks.js</code> · <strong>Schema:</strong> migrations 003–007</p>
    </div>

    <!-- Data Model -->
    <div class="card">
      <h2>🗄️ Data Model</h2>

      <h3>task_definitions</h3>
      <p class="subtitle">Every task that exists in the system — active, candidate, completed, or paused.</p>
      <table class="ref-table">
        <thead><tr><th>Column</th><th>Type</th><th>Description</th></tr></thead>
        <tbody>
          <tr v-for="col in taskDefCols" :key="col.name">
            <td><code>{{ col.name }}</code></td>
            <td>{{ col.type }}</td>
            <td v-html="col.desc"></td>
          </tr>
        </tbody>
      </table>

      <h3 style="margin-top: 1.5rem;">task_daily_log</h3>
      <p class="subtitle">Tracks what was scheduled each day and whether it was completed. One row per task per day. UNIQUE(date, task_definition_id).</p>
      <table class="ref-table">
        <thead><tr><th>Column</th><th>Type</th><th>Description</th></tr></thead>
        <tbody>
          <tr v-for="col in dailyLogCols" :key="col.name">
            <td><code>{{ col.name }}</code></td>
            <td>{{ col.type }}</td>
            <td>{{ col.desc }}</td>
          </tr>
        </tbody>
      </table>

      <h3 style="margin-top: 1.5rem;">task_config</h3>
      <p class="subtitle">Key-value config. Currently stores <code>google_tasks_list_id</code>.</p>
    </div>

    <!-- Task Types -->
    <div class="card">
      <h2>📦 Task Types</h2>
      <p class="subtitle">Types: <code>permanent</code>, <code>candidate</code>, <code>adhoc</code></p>
      <table class="ref-table">
        <thead><tr><th>Type</th><th>What It Is</th><th>Has google_task_id?</th><th>Lifecycle</th></tr></thead>
        <tbody>
          <tr v-for="tt in taskTypes" :key="tt.type">
            <td><strong>{{ tt.type }}</strong></td>
            <td v-html="tt.desc"></td>
            <td>{{ tt.hasGtId }}</td>
            <td v-html="tt.lifecycle"></td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Stacks -->
    <div class="card">
      <h2>📚 Stacks</h2>
      <p class="subtitle">CHECK constraint: <code>stack IN ('morning', 'after-work', 'after-dinner', 'weekend')</code></p>
      <table class="ref-table">
        <thead><tr><th>Stack</th><th>When Active</th><th>Budget</th><th>Sync Mechanism</th></tr></thead>
        <tbody>
          <tr v-for="s in stacks" :key="s.name">
            <td><strong>{{ s.emoji }} {{ s.name }}</strong></td>
            <td>{{ s.when }}</td>
            <td>{{ s.budget }}</td>
            <td v-html="s.sync"></td>
          </tr>
        </tbody>
      </table>
      <div class="code-block">
        <strong>activeStacksForDay(dateStr)</strong><br>
        <code>morning</code> + <code>after-dinner</code> → always active<br>
        <code>after-work</code> → weekdays only (Mon–Fri)<br>
        <code>weekend</code> → Saturday + Sunday only
      </div>
    </div>

    <!-- Recurrence Patterns -->
    <div class="card">
      <h2>🔄 Recurrence Patterns</h2>
      <table class="ref-table">
        <thead><tr><th>Pattern</th><th>Fires When</th><th>Notes</th></tr></thead>
        <tbody>
          <tr v-for="r in recurrencePatterns" :key="r.pattern">
            <td><code>{{ r.pattern }}</code></td>
            <td>{{ r.fires }}</td>
            <td v-html="r.notes"></td>
          </tr>
        </tbody>
      </table>
      <div class="code-block">
        <strong>Interval matching (quarterly, biannual, annual, biweekly):</strong><br>
        Uses <code>last_completed_date</code> + <code>interval_days</code> + <code>window_days</code>.<br>
        Task is due when: today ≥ (last_completed + interval - window).<br>
        If <code>last_completed_date</code> is NULL, task is always due (never been done).<br>
        <code>get_interval_tasks</code> returns status: upcoming, due, or overdue.
      </div>
    </div>

    <!-- Google Tasks Integration -->
    <div class="card">
      <h2>📱 Google Tasks Integration</h2>
      <p class="subtitle">One shared Google Tasks list. Config key: <code>google_tasks_list_id</code>.</p>

      <h3>Tools</h3>
      <table class="ref-table">
        <thead><tr><th>Tool</th><th>Used By</th><th>What It Does</th></tr></thead>
        <tbody>
          <tr v-for="t in gtTools" :key="t.name">
            <td><code>{{ t.name }}</code></td>
            <td>{{ t.usedBy }}</td>
            <td v-html="t.desc"></td>
          </tr>
        </tbody>
      </table>

      <h3 style="margin-top: 1.5rem;">Completion Detection</h3>
      <div class="code-block">
        A task is considered completed if:<br>
        (a) It's marked <code>status: completed</code> in Google Tasks, OR<br>
        (b) It's <strong>missing entirely</strong> from Google Tasks (Google clears completed items)<br><br>
        <strong>⚠️ This means deleting a task from Google Tasks = marking it done.</strong><br>
        This is by design — Google Tasks auto-clears completed items. But it means accidentally deleting tasks (or agents deleting them) causes false completions.
      </div>

      <h3 style="margin-top: 1.5rem;">google_task_id Lifecycle</h3>
      <div class="code-block">
        <strong>Created:</strong> When a task is first synced to Google Tasks (via <code>sync_stack</code> or <code>create_google_task</code>)<br>
        <strong>Persists:</strong> Stays on the task_definition permanently. Used to uncomplete/reset the same Google Task each day.<br>
        <strong>Cleared:</strong> Only when a task is deactivated or deleted from york-data.<br><br>
        <strong>Rule: Only permanent, recurring, and actively-surfaced adhoc tasks should have google_task_ids.</strong><br>
        Candidates should NEVER have google_task_ids — they don't exist in Google Tasks yet.
      </div>
    </div>

    <!-- Task Lifecycle -->
    <div class="card">
      <h2>🔄 Task Lifecycle</h2>

      <h3>Candidate → Permanent (Promotion)</h3>
      <div class="lifecycle">
        <div class="step">Candidate sits in backlog (type=candidate, no google_task_id)</div>
        <div class="arrow">↓</div>
        <div class="step">Planner evaluates: completion stats, effort budget, weather, calendar, logical grouping</div>
        <div class="arrow">↓</div>
        <div class="step">Planner promotes: <code>update_task_definition id:X type=permanent</code></div>
        <div class="arrow">↓</div>
        <div class="step"><code>sync_stack</code> creates it in Google Tasks → gets google_task_id</div>
        <div class="arrow">↓</div>
        <div class="step">Shows up on James's phone every time recurrence matches</div>
        <div class="arrow">↓</div>
        <div class="step">James completes it → <code>sync_completions</code> or <code>sync_stack</code> marks it done in daily_log</div>
      </div>

      <h3 style="margin-top: 1.5rem;">Adhoc Task (One-Time)</h3>
      <div class="lifecycle">
        <div class="step">Created as candidate (type=candidate) in a stack</div>
        <div class="arrow">↓</div>
        <div class="step">Planner or James surfaces it: <code>update_task_definition id:X type=adhoc</code></div>
        <div class="arrow">↓</div>
        <div class="step"><code>sync_stack</code> creates it in Google Tasks → gets google_task_id</div>
        <div class="arrow">↓</div>
        <div class="step">James completes it → <code>completed_at</code> set on task_definition</div>
        <div class="arrow">↓</div>
        <div class="step">Task is done. Stays in DB for history.</div>
      </div>

      <h3 style="margin-top: 1.5rem;">Interval Tasks (Quarterly, Biannual, Annual)</h3>
      <div class="lifecycle">
        <div class="step">Permanent task with <code>interval_days</code> set (e.g., 90 for quarterly)</div>
        <div class="arrow">↓</div>
        <div class="step"><code>get_interval_tasks</code> checks: is today within window of next due date?</div>
        <div class="arrow">↓</div>
        <div class="step">If due or overdue → planner adds to stack via <code>sync_stack</code></div>
        <div class="arrow">↓</div>
        <div class="step">When completed → <code>last_completed_date</code> updated → next due date recalculated</div>
      </div>

      <h3 style="margin-top: 1.5rem;">Followup Tasks</h3>
      <div class="lifecycle">
        <div class="step">Task A has <code>followup_task_id</code> pointing to Task B</div>
        <div class="arrow">↓</div>
        <div class="step">When Task A is completed, Task B is automatically scheduled</div>
        <div class="arrow">↓</div>
        <div class="step"><code>followup_delay</code>: 'same_day' or 'next_day'</div>
      </div>
      <p class="subtitle" style="margin-top: 0.5rem;">Example: "Start laundry" (Thursday) → followup → "Put away laundry" (same day or next day)</p>
    </div>

    <!-- Daily Planner Flow -->
    <div class="card">
      <h2>📅 Daily Planner Flow</h2>
      <p class="subtitle">Runs 5:30 AM daily. Agent: Hild (Opus). Skill: <code>daily-planner/SKILL.md</code></p>
      <table class="ref-table">
        <thead><tr><th>Step</th><th>Action</th><th>Tool</th></tr></thead>
        <tbody>
          <tr v-for="s in plannerSteps" :key="s.step">
            <td><strong>{{ s.step }}</strong></td>
            <td v-html="s.action"></td>
            <td><code>{{ s.tool }}</code></td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Nudge Crons -->
    <div class="card">
      <h2>⏰ Sync &amp; Nudge Schedule</h2>
      <table class="ref-table">
        <thead><tr><th>Time</th><th>Days</th><th>What</th><th>Tool</th></tr></thead>
        <tbody>
          <tr><td>5:30 AM</td><td>Daily</td><td>Daily planner (Opus) — full planning + morning sync + weekend sync (Sat only)</td><td><code>sync_completions</code>, <code>prepare_daily_stacks</code>, <code>sync_stack</code></td></tr>
          <tr><td>4:00 PM</td><td>M–F</td><td>After-work nudge — syncs + posts to #hild</td><td><code>sync_stack(after-work)</code></td></tr>
          <tr><td>7:00 PM</td><td>Daily</td><td>After-dinner nudge — syncs + posts to #hild</td><td><code>sync_stack(after-dinner)</code></td></tr>
        </tbody>
      </table>
    </div>

    <!-- Weekend Model -->
    <div class="card">
      <h2>🏡 Weekend Model</h2>
      <div class="rules-list">
        <div class="rule" v-for="r in weekendRules" :key="r">{{ r }}</div>
      </div>
    </div>

    <!-- Hard Rules -->
    <div class="card">
      <h2>🚨 Hard Rules</h2>
      <div class="rules-list">
        <div class="rule" v-for="r in hardRules" :key="r" v-html="r"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
const taskDefCols = [
  { name: 'id', type: 'INTEGER PK', desc: 'Auto-increment ID' },
  { name: 'title', type: 'TEXT NOT NULL', desc: 'Task name shown to James' },
  { name: 'stack', type: 'TEXT NOT NULL', desc: 'morning, after-work, after-dinner, or weekend' },
  { name: 'type', type: 'TEXT NOT NULL', desc: 'permanent, recurring, candidate, or adhoc' },
  { name: 'effort_minutes', type: 'INTEGER', desc: 'Estimated effort (1–60 min, default 5)' },
  { name: 'recur_pattern', type: 'TEXT', desc: 'When task fires. See recurrence patterns. NULL for unsurfaced adhocs/candidates.' },
  { name: 'position', type: 'INTEGER', desc: 'Order within the stack (lower = first)' },
  { name: 'active', type: 'INTEGER', desc: '1 = live, 0 = soft-deleted/paused. Inactive tasks are kept for history.' },
  { name: 'deadline', type: 'TEXT', desc: 'YYYY-MM-DD. Optional. For adhoc tasks with a hard deadline.' },
  { name: 'notes', type: 'TEXT', desc: 'Freeform notes. Shown in Google Tasks.' },
  { name: 'google_task_id', type: 'TEXT', desc: 'Google Tasks ID for sync. <strong>Only permanent, recurring, and actively-surfaced adhoc tasks should have this. Candidates must NOT.</strong>' },
  { name: 'created_at', type: 'TEXT', desc: 'When the task was created in york-data' },
  { name: 'completed_at', type: 'TEXT', desc: 'When an adhoc task was completed (one-time). NULL for recurring/permanent.' },
  { name: 'interval_days', type: 'INTEGER', desc: 'Days between completions for interval tasks (14=biweekly, 90=quarterly, 365=annual). NULL for non-interval.' },
  { name: 'window_days', type: 'INTEGER', desc: 'Days before due date to start surfacing (default 7)' },
  { name: 'last_completed_date', type: 'TEXT', desc: 'YYYY-MM-DD of most recent completion. Used by interval matching. NULL = never done.' },
  { name: 'followup_task_id', type: 'INTEGER FK', desc: 'Task definition to auto-schedule when this task completes. NULL if no followup.' },
  { name: 'followup_delay', type: 'TEXT', desc: "'same_day' or 'next_day'. When the followup gets scheduled relative to parent completion." },
]

const dailyLogCols = [
  { name: 'id', type: 'INTEGER PK', desc: 'Auto-increment ID' },
  { name: 'date', type: 'TEXT NOT NULL', desc: 'YYYY-MM-DD' },
  { name: 'task_definition_id', type: 'INTEGER FK', desc: 'References task_definitions.id' },
  { name: 'stack', type: 'TEXT NOT NULL', desc: 'Stack the task was in that day' },
  { name: 'scheduled', type: 'INTEGER', desc: '1 = placed on stack, 0 = skipped by planner' },
  { name: 'completed', type: 'INTEGER', desc: '1 = done, 0 = not done' },
  { name: 'skipped_reason', type: 'TEXT', desc: 'Why planner or James skipped it' },
  { name: 'created_at', type: 'TEXT', desc: 'When the log entry was created' },
]

const taskTypes = [
  { type: 'permanent', desc: 'Active stack tasks. Show up every time their recurrence matches. Includes daily tasks, weekly tasks, interval tasks — anything that\'s on the active list.', hasGtId: 'Yes — always', lifecycle: 'Created by planner promoting a candidate. Lives on the stack indefinitely.' },
  { type: 'candidate', desc: 'Backlog. The planner evaluates and promotes when ready.', hasGtId: '<strong>No — never.</strong> Not in Google Tasks until promoted.', lifecycle: 'Sits in backlog → planner promotes to permanent or adhoc → gets google_task_id.' },
  { type: 'adhoc', desc: 'One-time tasks that have been surfaced for execution by the planner or by James directly.', hasGtId: 'Yes — when surfaced', lifecycle: 'Candidate → surfaced as adhoc → gets google_task_id → completed → completed_at set → done.' },
]

const stacks = [
  { emoji: '☀️', name: 'morning', when: 'Every day (including weekends)', budget: '~30m', sync: 'Planner at 5:30 AM via <code>sync_stack(morning)</code>' },
  { emoji: '🌆', name: 'after-work', when: 'Weekdays only (Mon–Fri)', budget: '~20m', sync: 'Nudge cron at 4:00 PM via <code>sync_stack(after-work)</code>' },
  { emoji: '🌙', name: 'after-dinner', when: 'Every day (including weekends)', budget: '~30m', sync: 'Nudge cron at 7:00 PM via <code>sync_stack(after-dinner)</code>' },
  { emoji: '🏡', name: 'weekend', when: 'Saturday + Sunday', budget: '~120m total (both days)', sync: 'Planner on <strong>Saturday only</strong> via <code>sync_stack(weekend)</code>. Sunday does NOT re-sync.' },
]

const recurrencePatterns = [
  { pattern: 'daily', fires: 'Every day', notes: '' },
  { pattern: 'weekdays', fires: 'Mon–Fri', notes: '' },
  { pattern: 'weekends', fires: 'Sat + Sun', notes: '' },
  { pattern: 'weekly:<day>', fires: 'Specific day of week', notes: 'e.g., <code>weekly:monday</code>, <code>weekly:thursday</code>' },
  { pattern: 'weekly:weekend', fires: 'Every Saturday + Sunday', notes: 'Use for weekend-stack tasks. Fires both days.' },
  { pattern: 'biweekly:<day>', fires: 'Every other <day>', notes: 'Uses interval matching (14 days) against <code>last_completed_date</code>' },
  { pattern: 'biweekly:weekend', fires: 'Every other weekend (Sat + Sun)', notes: 'Interval check uses Saturday as anchor. If today is Sunday, checks if yesterday (Saturday) matched.' },
  { pattern: 'monthly:<day>', fires: 'Day N of each month', notes: 'e.g., <code>monthly:1</code> = 1st of every month' },
  { pattern: 'quarterly', fires: 'Every ~90 days', notes: 'Interval-based. Uses <code>interval_days=90</code>, <code>window_days=7</code>, <code>last_completed_date</code>.' },
  { pattern: 'biannual', fires: 'Every ~182 days', notes: 'Interval-based.' },
  { pattern: 'annual', fires: 'Every ~365 days', notes: 'Interval-based.' },
  { pattern: 'on_demand', fires: 'Never auto-scheduled', notes: 'Followup-only or manually triggered. Used by followup tasks.' },
]

const gtTools = [
  { name: 'sync_stack', usedBy: 'Nudge crons + planner', desc: 'All-in-one: syncs completions from Google Tasks → york-data, resets/uncompletes this stack\'s tasks in Google Tasks, creates any missing tasks, returns the stack ready to post. <strong>This is the primary sync mechanism.</strong>' },
  { name: 'sync_completions', usedBy: 'Planner (5:30 AM)', desc: 'Syncs all completions for a date across all stacks. Used at start of planner to capture yesterday\'s results.' },
  { name: 'create_google_task', usedBy: 'Planner', desc: 'Creates a single task in Google Tasks. Used when promoting a candidate and it needs a google_task_id immediately.' },
  { name: 'set_task_config', usedBy: 'Setup', desc: 'Sets config values (e.g., <code>google_tasks_list_id</code>).' },
]

const plannerSteps = [
  { step: '1', action: '<strong>Sync yesterday:</strong> <code>sync_completions(yesterday)</code> — pull completed tasks from Google Tasks into york-data', tool: 'sync_completions' },
  { step: '2', action: '<strong>Get context:</strong> Yesterday\'s completion, 7-day stack stats, weather, calendar from shared cache', tool: 'get_task_completion, get_stack_stats' },
  { step: '3', action: '<strong>Compute stacks:</strong> Get today\'s tasks by stack, filtered by recurrence. Writes to daily log.', tool: 'prepare_daily_stacks' },
  { step: '4', action: '<strong>Interval tasks:</strong> Check for due/overdue interval tasks. Due or overdue → add to the stack. Upcoming → mention in notes.', tool: 'get_interval_tasks' },
  { step: '5', action: '<strong>Evaluate candidates:</strong> Check completion stats, effort budgets, weather, calendar, logical grouping. Promote candidates to permanent when ready. <strong>Same process for all stacks — no special cases for weekends.</strong>', tool: 'update_task_definition' },
  { step: '6', action: '<strong>Sync morning:</strong> Always. <strong>Sync weekend:</strong> Saturday only. After-work and after-dinner are synced by their own nudge crons.', tool: 'sync_stack' },
  { step: '7', action: '<strong>Write cache:</strong> Write Home section to shared cache for Dagr\'s morning brief.', tool: 'write file' },
]

const weekendRules = [
  'Weekend is ONE block across Saturday and Sunday — not two separate days.',
  'weekly:weekend and biweekly:weekend recurrence patterns fire on both Saturday and Sunday.',
  'Saturday planner generates and syncs the full weekend task list to Google Tasks.',
  'Sunday planner does NOT re-sync. Tasks stay in Google Tasks from Saturday. Sunday shows what\'s still incomplete.',
  'Completing a task on Saturday keeps it done through Sunday — no re-appearance.',
  'Weekend effort budget is ~120m total across both days, not per day.',
  'Candidate promotion for weekend follows the exact same process as weekday stacks. No special cases.',
]

const hardRules = [
  '<strong>Never skip or defer stacks based on travel or calendar.</strong> Present the full list. James decides what to do.',
  '<strong>Never dump the entire backlog onto a weekend.</strong> Planner curates a focused list within effort budgets.',
  '<strong>Candidates must NOT have google_task_ids.</strong> They don\'t exist in Google Tasks until promoted.',
  '<strong>Interval tasks that are due or overdue go on the stack</strong> — not just mentioned in planner notes. Put them on the list or don\'t mention them.',
  '<strong>Deleting a Google Task = completing it</strong> (missing = done logic). Agents must never delete tasks from Google Tasks unless intentionally marking them done.',
  '<strong>The planner process is identical for all stacks.</strong> Morning, after-dinner, weekend — same promotion logic, same evaluation criteria, same effort budget enforcement. No special cases.',
  '<strong>Adhoc tasks follow the same promotion gate as candidates.</strong> They start as candidates, get promoted to adhoc when surfaced, get google_task_ids, appear on the phone.',
]
</script>

<style scoped>
.subtitle {
  color: #8b949e;
  margin-bottom: 0.75rem;
  font-size: 0.85rem;
}

.ref-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.85rem;
  margin-bottom: 1rem;
}

.ref-table th {
  text-align: left;
  padding: 0.5rem;
  border-bottom: 2px solid #30363d;
  color: #8b949e;
  font-size: 0.75rem;
  text-transform: uppercase;
}

.ref-table td {
  padding: 0.5rem;
  border-bottom: 1px solid #21262d;
  vertical-align: top;
}

.ref-table code {
  background: #161b22;
  padding: 0.15rem 0.4rem;
  border-radius: 3px;
  font-size: 0.8rem;
  color: #79c0ff;
}

.code-block {
  background: #161b22;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1rem;
  font-size: 0.85rem;
  line-height: 1.6;
  margin: 0.75rem 0;
}

.code-block code {
  background: #0d1117;
  padding: 0.1rem 0.3rem;
  border-radius: 3px;
  color: #79c0ff;
}

.lifecycle {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  margin: 0.75rem 0;
}

.lifecycle .step {
  background: #161b22;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 0.5rem 0.75rem;
  font-size: 0.85rem;
}

.lifecycle .step code {
  background: #0d1117;
  padding: 0.1rem 0.3rem;
  border-radius: 3px;
  color: #79c0ff;
}

.lifecycle .arrow {
  text-align: center;
  color: #8b949e;
  font-size: 0.75rem;
}

.rules-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.rule {
  background: #161b22;
  border-left: 3px solid #f85149;
  padding: 0.5rem 0.75rem;
  font-size: 0.85rem;
  line-height: 1.5;
}

.rule code {
  background: #0d1117;
  padding: 0.1rem 0.3rem;
  border-radius: 3px;
  color: #79c0ff;
}
</style>
