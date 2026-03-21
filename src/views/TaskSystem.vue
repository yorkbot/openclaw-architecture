<template>
  <div>
    <div class="card">
      <h2>🦡 Hild Task System Design</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Complete reference for the Hild home management task system. York-data (intelligence layer) + Google Tasks (execution layer) + daily planner cron + nudge crons. This document describes how the system actually works.
      </p>
    </div>

    <div class="card">
      <h3>System Architecture</h3>
      <div class="arch-grid">
        <div class="arch-box">
          <h4>york-data (Intelligence)</h4>
          <p>Task definitions, recurrence patterns, interval tracking, completion history, candidate backlog. The planner reads this.</p>
        </div>
        <div class="arch-box">
          <h4>Google Tasks (Execution)</h4>
          <p>What James sees on his phone. One Google Tasks list synced bidirectionally. Tasks appear when scheduled, disappear when completed.</p>
        </div>
        <div class="arch-box">
          <h4>Daily Planner (Opus, 5:30 AM)</h4>
          <p>Syncs yesterday's completions, computes today's stacks, surfaces adhoc tasks, evaluates candidates for promotion, syncs Google Tasks, writes plan to shared cache for Dagr.</p>
        </div>
        <div class="arch-box">
          <h4>Nudge Crons</h4>
          <p>4 PM M-F (after-work), 7 PM daily (after-dinner). Sync stack to Google Tasks, reset tasks, post to #hild.</p>
        </div>
      </div>
    </div>

    <div class="card">
      <h3>Data Model</h3>
      
      <h4 class="table-title">task_definitions</h4>
      <p class="table-desc">Core task definitions. Intelligence layer. Created via <code>create_task_definition()</code> or <code>create_google_task()</code> (which calls create + syncs to Google).</p>
      <table class="schema-table">
        <thead>
          <tr><th>Column</th><th>Type</th><th>Description</th></tr>
        </thead>
        <tbody>
          <tr><td><code>id</code></td><td>INTEGER</td><td>Primary key</td></tr>
          <tr><td><code>title</code></td><td>TEXT</td><td>Task name (e.g., "Cat litter", "Take garbage to curb")</td></tr>
          <tr><td><code>stack</code></td><td>TEXT</td><td>morning | after-work | after-dinner | weekend</td></tr>
          <tr><td><code>type</code></td><td>TEXT</td><td>permanent | recurring | candidate | adhoc (see Task Types below)</td></tr>
          <tr><td><code>effort_minutes</code></td><td>INTEGER</td><td>Estimated effort 1-60 min (default 5). Used for stack budget calculations.</td></tr>
          <tr><td><code>recur_pattern</code></td><td>TEXT</td><td>Recurrence rule (see Recurrence Patterns below). NULL for adhoc/candidate until promoted.</td></tr>
          <tr><td><code>interval_days</code></td><td>INTEGER</td><td>Custom interval in days for longer-cycle tasks (overrides pattern default). E.g., 90 for quarterly, 182 for biannual, 365 for annual.</td></tr>
          <tr><td><code>window_days</code></td><td>INTEGER</td><td>Days before due to start surfacing the task (default 7). For interval-based patterns only.</td></tr>
          <tr><td><code>last_completed_date</code></td><td>TEXT</td><td>YYYY-MM-DD of most recent completion. Used for interval-based recurrence calculations.</td></tr>
          <tr><td><code>position</code></td><td>INTEGER</td><td>Order within the stack (lower = first). Default 0.</td></tr>
          <tr><td><code>active</code></td><td>INTEGER</td><td>1=active, 0=soft-deleted/paused. Inactive tasks are kept for history.</td></tr>
          <tr><td><code>deadline</code></td><td>TEXT</td><td>YYYY-MM-DD deadline for adhoc tasks. Optional.</td></tr>
          <tr><td><code>notes</code></td><td>TEXT</td><td>Additional context. Synced to Google Tasks notes field.</td></tr>
          <tr><td><code>google_task_id</code></td><td>TEXT</td><td>Google Tasks ID for sync. NULL until task is synced to Google Tasks.</td></tr>
          <tr><td><code>followup_task_id</code></td><td>INTEGER</td><td>Task definition ID to auto-schedule when this task is completed (e.g., "Laundry" → "Put away laundry").</td></tr>
          <tr><td><code>followup_delay</code></td><td>TEXT</td><td>same_day (default) | next_day — when to schedule the followup task.</td></tr>
          <tr><td><code>created_at</code></td><td>TEXT</td><td>ISO timestamp of task creation</td></tr>
          <tr><td><code>completed_at</code></td><td>TEXT</td><td>When an adhoc task was completed (one-time tasks only). NULL for recurring.</td></tr>
        </tbody>
      </table>

      <h4 class="table-title">task_daily_log</h4>
      <p class="table-desc">Tracks which tasks were scheduled for each day and their completion. Written by <code>prepare_daily_stacks()</code> with <code>write_log=1</code>. Updated by <code>log_task_completion()</code>, <code>sync_stack()</code>, and <code>sync_completions()</code>.</p>
      <table class="schema-table">
        <thead>
          <tr><th>Column</th><th>Type</th><th>Description</th></tr>
        </thead>
        <tbody>
          <tr><td><code>id</code></td><td>INTEGER</td><td>Primary key</td></tr>
          <tr><td><code>date</code></td><td>TEXT</td><td>YYYY-MM-DD of the scheduled day</td></tr>
          <tr><td><code>task_definition_id</code></td><td>INTEGER</td><td>FK to task_definitions</td></tr>
          <tr><td><code>stack</code></td><td>TEXT</td><td>Which stack this task was scheduled for on this date</td></tr>
          <tr><td><code>scheduled</code></td><td>INTEGER</td><td>1 = placed on today's stack, 0 = skipped by planner</td></tr>
          <tr><td><code>completed</code></td><td>INTEGER</td><td>1 = done, 0 = not done</td></tr>
          <tr><td><code>skipped_reason</code></td><td>TEXT</td><td>Why the planner skipped it, or why James skipped it. Optional.</td></tr>
          <tr><td><code>created_at</code></td><td>TEXT</td><td>ISO timestamp</td></tr>
        </tbody>
      </table>
      <p class="note">Note: UNIQUE(date, task_definition_id) — each task appears at most once per day in the log.</p>

      <h4 class="table-title">task_config</h4>
      <p class="table-desc">Key-value config for the task system. Most important: <code>google_tasks_list_id</code>.</p>
      <table class="schema-table">
        <thead>
          <tr><th>Column</th><th>Type</th><th>Description</th></tr>
        </thead>
        <tbody>
          <tr><td><code>key</code></td><td>TEXT</td><td>Config key (PRIMARY KEY)</td></tr>
          <tr><td><code>value</code></td><td>TEXT</td><td>Config value</td></tr>
        </tbody>
      </table>
      <p class="note">Example: <code>key='google_tasks_list_id', value='MTU...'</code></p>
    </div>

    <div class="card">
      <h3>Task Types</h3>
      <p class="note">From the CHECK constraint: <code>type IN ('permanent', 'recurring', 'candidate', 'adhoc')</code></p>
      
      <div class="type-grid">
        <div class="type-box permanent">
          <h4>permanent</h4>
          <p><strong>What:</strong> Tasks that are always on the active stack. They show up every time their recurrence matches. Core habits.</p>
          <p><strong>Google Task ID:</strong> ✅ Yes — created when task is created or promoted.</p>
          <p><strong>Examples:</strong> Cat litter (daily), kitchen counters (daily), take garbage to curb (weekly:monday).</p>
          <p><strong>Lifecycle:</strong> Created → active → appears on matching days → completed daily → repeats forever.</p>
        </div>

        <div class="type-box recurring">
          <h4>recurring</h4>
          <p><strong>What:</strong> Like permanent but conceptually separate — used for weekday-specific routines. Same behavior, just a label distinction.</p>
          <p><strong>Google Task ID:</strong> ✅ Yes — created when task is created.</p>
          <p><strong>Examples:</strong> Garbage Monday, recycling bins Tuesday, laundry Thursday.</p>
          <p><strong>Lifecycle:</strong> Same as permanent. The type distinction is semantic, not functional.</p>
        </div>

        <div class="type-box candidate">
          <h4>candidate</h4>
          <p><strong>What:</strong> Aspirational tasks in the backlog. The planner evaluates them and promotes to permanent when the stack is ready.</p>
          <p><strong>Google Task ID:</strong> ❌ No — candidates don't appear in Google Tasks until promoted.</p>
          <p><strong>Examples:</strong> Sweep kitchen floor, wipe bathroom mirror, dust living room.</p>
          <p><strong>Lifecycle:</strong> Created as candidate → sits in backlog → planner promotes to permanent when stack has capacity → then gets google_task_id and starts appearing on schedule.</p>
          <p><strong>Promotion criteria:</strong> Weather, calendar, logical grouping, stack capacity, recurrence fit. Planner uses judgment based on completion stats.</p>
        </div>

        <div class="type-box adhoc">
          <h4>adhoc</h4>
          <p><strong>What:</strong> One-time tasks that have been surfaced to a stack by the planner or by James directly. When completed, they're marked done.</p>
          <p><strong>Google Task ID:</strong> ✅ Yes — created when surfaced (either by planner or via <code>create_google_task()</code>).</p>
          <p><strong>Examples:</strong> Call dentist, buy air filter, install new shelves.</p>
          <p><strong>Lifecycle:</strong> Created as adhoc + active + assigned to stack → planner or James surfaces it → gets google_task_id → appears in Google Tasks → completed → <code>completed_at</code> set, <code>active=0</code>.</p>
          <p><strong>Different from candidates:</strong> Adhoc tasks have been explicitly chosen for execution. Candidates are still in the backlog awaiting promotion.</p>
        </div>
      </div>
    </div>

    <div class="card">
      <h3>Stacks</h3>
      <p class="note">From the CHECK constraint: <code>stack IN ('morning', 'after-work', 'after-dinner', 'weekend')</code></p>
      
      <div class="stack-grid">
        <div class="stack-box morning">
          <h4>☀️ morning</h4>
          <p><strong>Active:</strong> Every day (7 days/week)</p>
          <p><strong>Effort budget:</strong> ~30 minutes</p>
          <p><strong>Sync:</strong> Daily planner at 5:30 AM</p>
          <p><strong>Examples:</strong> Cat litter, fresh water, movement</p>
        </div>

        <div class="stack-box after-work">
          <h4>🌆 after-work</h4>
          <p><strong>Active:</strong> Weekdays only (Monday-Friday)</p>
          <p><strong>Effort budget:</strong> ~20 minutes</p>
          <p><strong>Sync:</strong> 4 PM M-F nudge cron</p>
          <p><strong>Examples:</strong> Take garbage to curb (Monday), recycling bins (Tuesday)</p>
        </div>

        <div class="stack-box after-dinner">
          <h4>🌙 after-dinner</h4>
          <p><strong>Active:</strong> Every day (7 days/week)</p>
          <p><strong>Effort budget:</strong> ~30 minutes</p>
          <p><strong>Sync:</strong> 7 PM daily nudge cron</p>
          <p><strong>Examples:</strong> Kitchen counters, dishes</p>
        </div>

        <div class="stack-box weekend">
          <h4>🏡 weekend</h4>
          <p><strong>Active:</strong> Saturday and Sunday only</p>
          <p><strong>Effort budget:</strong> ~120 minutes total across both days</p>
          <p><strong>Sync:</strong> Saturday morning planner only (not Sunday — tasks stay in Google Tasks from Saturday)</p>
          <p><strong>Examples:</strong> Laundry, vacuum, grocery shopping</p>
          <p><strong>Notes:</strong> Weekend is ONE block across both days. James completes tasks on either Saturday or Sunday. Completing a task Saturday keeps it done through Sunday.</p>
        </div>
      </div>

      <h4 class="code-ref">activeStacksForDay() Logic</h4>
      <pre class="code-block">function activeStacksForDay(dateStr) {
  const d = new Date(dateStr + 'T12:00:00');
  const dow = d.getDay(); // 0=Sun, 1=Mon, ...
  const isWeekday = dow >= 1 && dow <= 5;
  const stacks = ['morning', 'after-dinner']; // always active
  if (isWeekday) {
    stacks.push('after-work');
  } else {
    stacks.push('weekend');
  }
  return stacks;
}</pre>
      <p class="note">Morning and after-dinner run 7 days/week. After-work is M-F only. Weekend is Sat-Sun only.</p>
    </div>

    <div class="card">
      <h3>Recurrence Patterns</h3>
      <p class="note">All valid patterns from the code's <code>validateRecurPattern()</code> and <code>matchesRecurrence()</code> functions.</p>

      <table class="recur-table">
        <thead>
          <tr><th>Pattern</th><th>Description</th><th>How matchesRecurrence() Works</th></tr>
        </thead>
        <tbody>
          <tr>
            <td><code>daily</code></td>
            <td>Every day</td>
            <td>Always returns true.</td>
          </tr>
          <tr>
            <td><code>weekdays</code></td>
            <td>Monday through Friday</td>
            <td>Returns true if day-of-week is 1-5 (JavaScript: Mon=1, Fri=5).</td>
          </tr>
          <tr>
            <td><code>weekends</code></td>
            <td>Saturday and Sunday</td>
            <td>Returns true if day-of-week is 0 (Sun) or 6 (Sat).</td>
          </tr>
          <tr>
            <td><code>weekly:&lt;day&gt;</code></td>
            <td>Every <em>day</em> (monday, tuesday, ..., sunday)</td>
            <td>Parses the day name, converts to JS day-of-week (adjusting for 0=Sunday), compares.</td>
          </tr>
          <tr>
            <td><code>weekly:weekend</code></td>
            <td>Every Saturday and Sunday</td>
            <td>Returns true if day-of-week is 0 or 6.</td>
          </tr>
          <tr>
            <td><code>biweekly:&lt;day&gt;</code></td>
            <td>Every other <em>day</em></td>
            <td>First checks if today matches the day-of-week. Then calls <code>matchesInterval(dateStr, task, 14)</code> to confirm it's the right week in the 14-day cycle.</td>
          </tr>
          <tr>
            <td><code>biweekly:weekend</code></td>
            <td>Every other weekend (both Sat and Sun)</td>
            <td>Returns true on both Saturday and Sunday when the interval matches. Uses Saturday as the anchor for the 14-day interval calculation. On Sunday, checks if yesterday (Saturday) matched the interval.</td>
          </tr>
          <tr>
            <td><code>monthly:&lt;day&gt;</code></td>
            <td>Every month on day <em>day</em> (1-31)</td>
            <td>Returns true if today's day-of-month equals the target day.</td>
          </tr>
          <tr>
            <td><code>quarterly</code></td>
            <td>Every 90 days (default)</td>
            <td>Interval-based. Calls <code>matchesInterval(dateStr, task, 90)</code>. Task appears when days since last completion ≥ (interval_days - window_days). If never completed, always due.</td>
          </tr>
          <tr>
            <td><code>biannual</code></td>
            <td>Every 182 days (default)</td>
            <td>Interval-based. Same logic as quarterly, default interval 182 days.</td>
          </tr>
          <tr>
            <td><code>annual</code></td>
            <td>Every 365 days (default)</td>
            <td>Interval-based. Same logic as quarterly, default interval 365 days.</td>
          </tr>
          <tr>
            <td><code>on_demand</code></td>
            <td>Followup-only or manually triggered</td>
            <td><strong>Never</strong> auto-scheduled by recurrence. Only scheduled when parent task completes (via followup_task_id) or manually by the planner.</td>
          </tr>
        </tbody>
      </table>

      <h4 class="code-ref">matchesInterval() Logic</h4>
      <pre class="code-block">function matchesInterval(dateStr, task, defaultInterval) {
  const intervalDays = task?.interval_days || defaultInterval;
  const windowDays = task?.window_days ?? 7;
  const lastCompleted = task?.last_completed_date;

  // Never completed = always due
  if (!lastCompleted) return true;

  const today = new Date(dateStr + 'T12:00:00');
  const last = new Date(lastCompleted + 'T12:00:00');
  const daysSince = Math.floor((today - last) / (1000 * 60 * 60 * 24));

  // Due when within the window: interval - window <= daysSince
  return daysSince >= (intervalDays - windowDays);
}</pre>
      <p class="note">Interval-based tasks surface within a window (default 7 days) before they're due. Example: quarterly task (90 days) with 7-day window starts appearing 83 days after last completion.</p>

      <h4 class="code-ref">intervalStatus() Return Values</h4>
      <table class="interval-status-table">
        <thead>
          <tr><th>Status</th><th>When</th><th>days_until_due</th><th>days_overdue</th></tr>
        </thead>
        <tbody>
          <tr><td><code>not_due</code></td><td>More than <code>window_days</code> away from due date</td><td>Positive</td><td>0</td></tr>
          <tr><td><code>upcoming</code></td><td>Within the window but not yet due</td><td>Positive (1 to window_days)</td><td>0</td></tr>
          <tr><td><code>due</code></td><td>Exactly on the due date</td><td>0</td><td>0</td></tr>
          <tr><td><code>overdue</code></td><td>Past the due date</td><td>0</td><td>Positive</td></tr>
        </tbody>
      </table>
      <p class="note">Used by <code>get_interval_tasks()</code> to report task status. The planner adds due/overdue tasks to the stack.</p>
    </div>

    <div class="card">
      <h3>Google Tasks Integration</h3>
      
      <div class="sync-flow">
        <h4>Sync Architecture</h4>
        <p>One Google Tasks list synced bidirectionally. List ID stored in <code>task_config</code> table as <code>google_tasks_list_id</code>.</p>
        <p><strong>Direction:</strong> York-data → Google Tasks for scheduling. Google Tasks → york-data for completion detection.</p>
      </div>

      <h4>Core Tools</h4>
      <div class="tool-grid">
        <div class="tool-box">
          <h5>sync_stack</h5>
          <p><strong>Purpose:</strong> All-in-one nudge tool. Syncs completions from Google Tasks, resets this stack's tasks, returns the stack ready to post.</p>
          <p><strong>When:</strong> 4 PM M-F (after-work), 7 PM daily (after-dinner), Saturday morning (weekend).</p>
          <p><strong>What it does:</strong></p>
          <ol>
            <li>Syncs completions from Google Tasks to york-data (same logic as sync_completions)</li>
            <li>Computes today's stack tasks</li>
            <li>Resets this stack's tasks in Google Tasks (uncompletes existing, creates any missing)</li>
            <li>Ensures daily log entries exist</li>
            <li>Returns earlier stacks' completion status for context</li>
            <li>Returns current streak for this stack</li>
          </ol>
          <p><strong>Output:</strong> Ready-to-post stack summary with tasks, effort, completions synced count, earlier stacks' status, streak.</p>
        </div>

        <div class="tool-box">
          <h5>sync_completions</h5>
          <p><strong>Purpose:</strong> Planner tool. Syncs all completions from Google Tasks to york-data for a given date.</p>
          <p><strong>When:</strong> First step of daily planner (5:30 AM) — syncs yesterday's completions before planning today.</p>
          <p><strong>What it does:</strong></p>
          <ol>
            <li>Fetches all tasks from Google Tasks (including completed)</li>
            <li>For each scheduled task in york-data with a google_task_id: if it's completed in Google Tasks OR missing from the list entirely (cleared after completion), mark it completed in york-data</li>
            <li>Updates adhoc tasks' <code>completed_at</code>, sets <code>active=0</code></li>
            <li>Updates interval tasks' <code>last_completed_date</code></li>
            <li>Schedules followup tasks if configured</li>
            <li>Clears completed tasks from Google Tasks</li>
          </ol>
          <p><strong>Output:</strong> Count of synced completions and list of task titles.</p>
        </div>

        <div class="tool-box">
          <h5>create_google_task</h5>
          <p><strong>Purpose:</strong> Creates a single task in both york-data and Google Tasks in one call.</p>
          <p><strong>When:</strong> Task intake (manual or via conversation with Hild).</p>
          <p><strong>What it does:</strong></p>
          <ol>
            <li>Validates parameters (effort, deadline, recur_pattern)</li>
            <li>Inserts into task_definitions</li>
            <li>Creates task in Google Tasks (unless type=candidate)</li>
            <li>Updates task_definitions with google_task_id</li>
          </ol>
          <p><strong>Note:</strong> Candidate tasks do NOT get google_task_ids until promoted to permanent.</p>
        </div>
      </div>

      <h4>Completion Detection</h4>
      <p>A task is considered "completed" if:</p>
      <ol>
        <li><strong>Explicitly completed in Google Tasks:</strong> Status = "completed"</li>
        <li><strong>Missing from Google Tasks entirely:</strong> Google Tasks auto-clears completed tasks, so if a task that was scheduled today with a google_task_id is no longer in the list (and not in the completed list either — it was cleared), it's done.</li>
      </ol>
      <p class="note">This dual detection handles both immediate completion (task still shows as completed) and delayed sync (task was completed and already cleared by Google Tasks).</p>

      <h4>google_task_id Lifecycle</h4>
      <div class="lifecycle-flow">
        <div class="lifecycle-step">
          <strong>Permanent/Recurring:</strong> Created at task creation via <code>create_google_task()</code> or when first synced via <code>sync_stack()</code>.
        </div>
        <div class="lifecycle-step">
          <strong>Candidate:</strong> NULL until promoted to permanent. Then created on next <code>sync_stack()</code> call.
        </div>
        <div class="lifecycle-step">
          <strong>Adhoc:</strong> Created when task is surfaced (either by planner calling <code>create_google_task()</code> or manually by James via conversation).
        </div>
        <div class="lifecycle-step">
          <strong>Deactivation:</strong> google_task_id stays in york-data even after task is deactivated. Allows re-activation or historical reference.
        </div>
      </div>
    </div>

    <div class="card">
      <h3>Task Lifecycle</h3>

      <div class="lifecycle-section">
        <h4>Permanent Task Lifecycle</h4>
        <div class="lifecycle-diagram">
          <div class="lifecycle-node">Created via <code>create_task_definition()</code> or <code>create_google_task()</code></div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Type = permanent, active = 1, recur_pattern set (e.g., daily, weekly:monday)</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Daily planner runs <code>prepare_daily_stacks()</code> — task matches recurrence, added to today's plan</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Planner calls <code>sync_stack()</code> for morning stack — task synced to Google Tasks (gets google_task_id if missing)</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Task appears on James's phone in Google Tasks app</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">James completes task in Google Tasks (or clears it manually)</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Next day's planner syncs yesterday via <code>sync_completions()</code> — detects completion, marks in york-data</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Task repeats on next matching day (e.g., tomorrow if daily, next Monday if weekly:monday)</div>
        </div>
      </div>

      <div class="lifecycle-section">
        <h4>Candidate → Permanent Promotion</h4>
        <div class="lifecycle-diagram">
          <div class="lifecycle-node">Created as candidate via <code>create_task_definition(type=candidate)</code></div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Sits in backlog. No google_task_id. Not scheduled on any day's stack.</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Daily planner evaluates candidates (Step 4): checks weather, calendar, stack capacity, recurrence fit, logical grouping</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Planner decides to promote: calls <code>update_task_definition(id=X, type=permanent)</code></div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Next <code>sync_stack()</code> call creates google_task_id, syncs to Google Tasks</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Task now appears on schedule like any other permanent task</div>
        </div>
        <p class="note">Promotion is gradual. Planner uses judgment based on completion stats and context. No artificial cooldowns or gates.</p>
      </div>

      <div class="lifecycle-section">
        <h4>Adhoc Task Lifecycle</h4>
        <div class="lifecycle-diagram">
          <div class="lifecycle-node">Created via <code>create_google_task(type=adhoc, stack=X)</code> — James says "I need to call the dentist"</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Type = adhoc, active = 1, assigned to a stack (e.g., after-work), google_task_id created immediately</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Planner evaluates: deadline pressure? Has it been sitting 5+ days? Fits stack budget?</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Planner surfaces it today (it's already active + assigned to stack, so <code>prepare_daily_stacks()</code> includes it)</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Task synced to Google Tasks via <code>sync_stack()</code> — appears on James's phone</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">James completes task</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Completion synced back: <code>completed_at</code> set, <code>active=0</code>. Task is done forever.</div>
        </div>
        <p class="note">Adhoc tasks are different from candidates: they've been explicitly chosen for execution. The planner decides WHEN to surface them, but they're already in the system as active tasks.</p>
      </div>

      <div class="lifecycle-section">
        <h4>Interval Task (Quarterly/Annual/etc.)</h4>
        <div class="lifecycle-diagram">
          <div class="lifecycle-node">Created with interval_days (e.g., 90 for quarterly) and optional last_completed_date</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Task is permanent, but only appears when within its window (default 7 days before due)</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Daily planner calls <code>get_interval_tasks()</code> — returns tasks that are upcoming, due, or overdue</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Planner adds due/overdue tasks to the weekend stack (or appropriate stack based on task.stack)</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Task synced to Google Tasks, James completes it</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Completion synced: <code>last_completed_date</code> updated to today</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Task disappears from daily stacks until interval_days - window_days have passed</div>
        </div>
        <p class="note">Interval tasks surface automatically based on time since last completion. No manual scheduling needed.</p>
      </div>

      <div class="lifecycle-section">
        <h4>Followup Task</h4>
        <div class="lifecycle-diagram">
          <div class="lifecycle-node">Parent task has followup_task_id pointing to a followup task (type=on_demand, recur_pattern=on_demand)</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Parent task is completed (via <code>log_task_completion()</code>, <code>sync_stack()</code>, or <code>sync_completions()</code>)</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node"><code>scheduleFollowup()</code> called automatically: inserts followup into task_daily_log for today or tomorrow (based on followup_delay)</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">If followup already has a google_task_id, uncompletes it. Otherwise, creates a new one.</div>
          <div class="lifecycle-arrow">↓</div>
          <div class="lifecycle-node">Followup task appears in Google Tasks immediately</div>
        </div>
        <p class="note">Example: "Laundry" (weekly:thursday) has followup_task_id pointing to "Put away laundry" (on_demand). When laundry is completed, "Put away laundry" is auto-scheduled for the same day (or next day if followup_delay=next_day).</p>
      </div>
    </div>

    <div class="card">
      <h3>Daily Planner Flow (5:30 AM, Opus)</h3>
      <p class="note">From <code>~/.openclaw/workspace-hild/skills/daily-planner/SKILL.md</code></p>

      <div class="planner-step">
        <h4>Step 1: Sync Yesterday's Completions</h4>
        <p><strong>Tool:</strong> <code>sync_completions(date=yesterday)</code></p>
        <p><strong>Why first:</strong> James completes tasks in Google Tasks throughout the day. This must run before any planning so yesterday's data is accurate.</p>
        <p><strong>Output:</strong> Count of synced tasks, list of completed task titles.</p>
      </div>

      <div class="planner-step">
        <h4>Step 2: Get Context</h4>
        <p><strong>Tools:</strong></p>
        <ul>
          <li><code>get_task_completion(date=yesterday)</code> — Yesterday's completion summary</li>
          <li><code>get_stack_stats()</code> — Rolling 7-day completion rates per stack</li>
        </ul>
        <p><strong>Files:</strong> Read <code>~/.openclaw/shared-cache/$TODAY.md</code> for weather and calendar context.</p>
        <p><strong>Output:</strong> Yesterday's done/scheduled counts, stack completion rates, today's weather, today's calendar.</p>
      </div>

      <div class="planner-step">
        <h4>Step 2b: Compute Today's Stacks</h4>
        <p><strong>Tool:</strong> <code>prepare_daily_stacks(date=today, write_log=1)</code></p>
        <p><strong>What it does:</strong> Filters all active permanent/recurring tasks by recurrence pattern. Groups by stack. Returns effort totals. Writes to daily log.</p>
        <p><strong>Output:</strong> Today's tasks grouped by stack, effort per stack, total tasks scheduled.</p>
      </div>

      <div class="planner-step">
        <h4>Step 2c: Check Interval Tasks</h4>
        <p><strong>Tool:</strong> <code>get_interval_tasks(date=today)</code></p>
        <p><strong>What it does:</strong> Returns all interval-based tasks (quarterly, annual, etc.) that are upcoming, due, or overdue.</p>
        <p><strong>Action:</strong> For each due/overdue task, add it to the appropriate stack (usually weekend). These are real tasks that need to happen. Don't just mention them in notes — surface them.</p>
        <p><strong>Output:</strong> List of interval tasks with status (upcoming/due/overdue), days until due, days overdue.</p>
      </div>

      <div class="planner-step">
        <h4>Step 3: Schedule Ad-Hoc Tasks</h4>
        <p><strong>Tool:</strong> <code>get_task_definitions(type=adhoc)</code></p>
        <p><strong>Decision factors:</strong></p>
        <ul>
          <li>Deadline within 2 days → MUST surface today</li>
          <li>Stack effort budget — does it fit without overloading?</li>
          <li>Age — has it been sitting 5+ days? Flag it.</li>
        </ul>
        <p><strong>Output:</strong> Adhoc tasks are already assigned to stacks. Planner decides if TODAY is the day to surface them based on deadline/priority/capacity.</p>
      </div>

      <div class="planner-step">
        <h4>Step 4: Evaluate Stack Growth</h4>
        <p><strong>Tools:</strong></p>
        <ul>
          <li><code>get_task_definitions(type=candidate)</code> — All backlog candidates</li>
          <li>Context from Steps 2 and 2a (weather, calendar, stack stats)</li>
        </ul>
        <p><strong>Evaluation criteria:</strong></p>
        <ul>
          <li><strong>Weather:</strong> Don't promote outdoor tasks on rainy/extreme days. Don't promote seasonal tasks out of season.</li>
          <li><strong>Calendar:</strong> Light day → good day to grow. Packed day → keep stacks light.</li>
          <li><strong>Effort budgets:</strong> Morning ~30m, After-work ~20m, After-dinner ~30m, Weekend ~120m total (not per day).</li>
          <li><strong>Recurrence fit:</strong> A daily candidate adds effort every day. A weekly:weekend candidate adds once per weekend. Weight accordingly.</li>
          <li><strong>Logical grouping:</strong> If James is already doing kitchen counters + dishes, sweeping the kitchen floor is a natural next add.</li>
          <li><strong>Completion stats:</strong> If a stack is at 90%+ completion for the past week, it can absorb more.</li>
        </ul>
        <p><strong>Action:</strong> Promote candidates to permanent via <code>update_task_definition(id=X, type=permanent)</code>.</p>
        <p><strong>Philosophy:</strong> Gradual growth. Use judgment. No artificial gates or cooldowns. If 3 stacks have room, promote into all 3.</p>
      </div>

      <div class="planner-step">
        <h4>Step 5: Sync Morning Stack (and Weekend on Saturday)</h4>
        <p><strong>Tool:</strong> <code>sync_stack(stack=morning, date=today)</code></p>
        <p><strong>What it does:</strong> Syncs any completions from overnight (unlikely but possible), resets morning tasks in Google Tasks, creates any missing google_task_ids, returns the stack ready to post.</p>
        <p><strong>Weekend sync (Saturday only):</strong> <code>sync_stack(stack=weekend, date=today)</code></p>
        <p><strong>Output:</strong> Morning stack tasks, effort, completions synced count, earlier stacks' status (none for morning), streak.</p>
      </div>

      <div class="planner-step">
        <h4>Step 6: Write to Shared Cache</h4>
        <p><strong>File:</strong> <code>~/.openclaw/shared-cache/$TODAY.md</code> under <code>## Home</code> section</p>
        <p><strong>Format:</strong></p>
        <pre class="code-block">## Home

**Yesterday:** Morning 3/3 ✅ | After-Dinner 2/2 ✅

**Today's Plan:**
- ☀️ Morning: cat litter → fresh water → movement (27m)
- 🌆 After-Work: take garbage to curb (5m)
- 🌙 After-Dinner: kitchen counters → dishes (15m)

**Planner notes:**
- Moved "call dentist" to after-work — deadline Friday
- Promoted "sweep kitchen floor" to after-dinner — natural fit
- 🔔 Change HVAC filter — quarterly, last done Dec 15, due this week
- ⚠️ Deep clean oven — biannual, 8 days overdue</pre>
        <p><strong>Rules:</strong></p>
        <ul>
          <li>Use <code>→</code> arrows for stack order</li>
          <li>Include effort totals per stack in parentheses</li>
          <li>Flag overdue interval tasks with ⚠️, upcoming with 🔔</li>
          <li>Omit "Planner notes" if no decisions were made</li>
        </ul>
        <p><strong>Output:</strong> Dagr reads this section and pastes it into the morning brief.</p>
      </div>
    </div>

    <div class="card">
      <h3>Nudge Crons</h3>
      
      <div class="cron-box">
        <h4>4 PM M-F: After-Work Nudge</h4>
        <p><strong>Tool:</strong> <code>sync_stack(stack=after-work, date=today)</code></p>
        <p><strong>What it does:</strong> Syncs any completions from morning/earlier, resets after-work tasks in Google Tasks, returns the stack.</p>
        <p><strong>Post to #hild:</strong> Format the output as a nudge message with today's tasks, effort, earlier stacks' completion status (morning, any carryover from other stacks).</p>
        <p><strong>Example:</strong></p>
        <pre class="nudge-example">🌆 **After-Work Stack** (5m)

→ Take garbage to curb

Morning: 3/3 ✅
Streak: 12 days</pre>
      </div>

      <div class="cron-box">
        <h4>7 PM Daily: After-Dinner Nudge</h4>
        <p><strong>Tool:</strong> <code>sync_stack(stack=after-dinner, date=today)</code></p>
        <p><strong>What it does:</strong> Syncs completions from morning + after-work, resets after-dinner tasks, returns the stack.</p>
        <p><strong>Post to #hild:</strong> Format the output with today's tasks, earlier stacks' status.</p>
        <p><strong>Example:</strong></p>
        <pre class="nudge-example">🌙 **After-Dinner Stack** (15m)

→ Kitchen counters → Dishes

Morning: 3/3 ✅ | After-Work: 1/1 ✅
Streak: 8 days</pre>
      </div>

      <div class="cron-box">
        <h4>No Weekend Nudge</h4>
        <p>Weekend stack is synced by the planner on Saturday morning. No separate nudge cron. James works through the weekend list at his own pace across both days.</p>
      </div>
    </div>

    <div class="card">
      <h3>Weekend Model</h3>
      
      <div class="weekend-section">
        <h4>Core Principle: Weekend is ONE block</h4>
        <p>Weekend spans Saturday and Sunday as a single time block, not two separate days. James completes tasks on either day. The system doesn't care which day — it only cares that tasks get done within the weekend block.</p>
      </div>

      <div class="weekend-section">
        <h4>Recurrence Patterns</h4>
        <p><code>weekly:weekend</code> and <code>biweekly:weekend</code> fire on BOTH Saturday and Sunday. This is intentional — the task appears on both days so James can do it on either day.</p>
        <p><strong>Example:</strong> "Laundry" with <code>weekly:weekend</code> appears on both Saturday and Sunday. James does it Saturday morning. It stays marked done through Sunday.</p>
      </div>

      <div class="weekend-section">
        <h4>Planning Schedule</h4>
        <ul>
          <li><strong>Saturday morning:</strong> Daily planner generates the full weekend task list and syncs it to Google Tasks via <code>sync_stack(stack=weekend)</code>.</li>
          <li><strong>Sunday morning:</strong> Daily planner does NOT re-sync the weekend stack. Tasks stay in Google Tasks from Saturday. The planner just checks what's still incomplete and includes that status in the shared cache.</li>
        </ul>
      </div>

      <div class="weekend-section">
        <h4>Completion Behavior</h4>
        <p>Completing a task on Saturday keeps it done through Sunday. The system doesn't reset weekend tasks between Saturday and Sunday — they're already scheduled for the full weekend block.</p>
        <p><strong>Example flow:</strong></p>
        <ol>
          <li>Saturday 5:30 AM: Planner syncs weekend stack → 5 tasks appear in Google Tasks</li>
          <li>Saturday afternoon: James completes 3 tasks</li>
          <li>Sunday morning: Planner syncs yesterday's completions (Saturday) → 3 tasks marked done in york-data</li>
          <li>Sunday: The remaining 2 tasks are still in Google Tasks from Saturday. James completes them Sunday afternoon.</li>
          <li>Monday morning: Planner syncs Sunday's completions → remaining 2 tasks marked done.</li>
        </ol>
      </div>

      <div class="weekend-section">
        <h4>Effort Budget</h4>
        <p><strong>~120 minutes total across both days,</strong> not 120 minutes per day.</p>
        <p>When the planner evaluates weekend candidates for promotion, it considers the full weekend block. A new 15-minute weekend task adds 15 minutes to the weekend total, not 15 minutes to each day.</p>
        <p><strong>Example:</strong> Current weekend stack is 90 minutes. Planner can promote candidates totaling up to ~30 more minutes across the weekend.</p>
      </div>
    </div>

    <div class="card">
      <h3>Effort Budgets</h3>
      <p class="note">Guidelines, not hard limits. Planner uses judgment on heavy vs light days.</p>
      
      <table class="budget-table">
        <thead>
          <tr><th>Stack</th><th>Effort Budget</th><th>Notes</th></tr>
        </thead>
        <tbody>
          <tr>
            <td>Morning</td>
            <td>~30 minutes</td>
            <td>Quick morning routine. Runs 7 days/week.</td>
          </tr>
          <tr>
            <td>After-Work</td>
            <td>~20 minutes</td>
            <td>Short post-work tasks. Weekdays only.</td>
          </tr>
          <tr>
            <td>After-Dinner</td>
            <td>~30 minutes</td>
            <td>Evening cleanup. Runs 7 days/week.</td>
          </tr>
          <tr>
            <td>Weekend</td>
            <td>~120 minutes total</td>
            <td>Across both Saturday and Sunday. Flexible timing within the block.</td>
          </tr>
        </tbody>
      </table>

      <p><strong>Planner guidance:</strong> If calendar shows a light day, stacks can go slightly over budget. If calendar shows a packed day, keep stacks under budget. Weekend has the most flexibility since it's a large block with no fixed schedule.</p>
    </div>

    <div class="card">
      <h3>System Rules</h3>
      
      <div class="rule-box critical">
        <h4>❌ Never skip/defer stacks based on travel or calendar</h4>
        <p>Even if the calendar shows James is away or traveling, present the full task list as normal. James decides what to do and when. The system doesn't make that call.</p>
        <p><strong>Example:</strong> James is traveling Saturday-Sunday. The weekend stack still gets synced Saturday morning with all weekend tasks. James may complete some before leaving, skip others, or catch up Monday. That's his decision, not the planner's.</p>
      </div>

      <div class="rule-box critical">
        <h4>❌ Never dump the entire backlog onto a weekend</h4>
        <p>The planner curates a focused, achievable list within effort budgets. Candidates are promoted gradually based on stack capacity and completion rates, not dumped all at once.</p>
      </div>

      <div class="rule-box important">
        <h4>✅ Planner curates within effort budgets</h4>
        <p>The planner uses judgment to keep stacks achievable. If a stack is consistently at 90%+ completion, it can absorb more. If it's struggling, hold off on promotions.</p>
      </div>

      <div class="rule-box important">
        <h4>✅ Interval tasks that are due/overdue get added to the stack</h4>
        <p>Don't just mention interval tasks in planner notes. If <code>get_interval_tasks()</code> returns a task with status "due" or "overdue", add it to the appropriate stack (usually weekend) and sync it to Google Tasks. These are real tasks that need to happen.</p>
      </div>

      <div class="rule-box important">
        <h4>✅ Candidates are evaluated based on multiple factors</h4>
        <p>Completion stats + weather + calendar + logical grouping + recurrence fit. The planner uses all available context to make promotion decisions. No single factor is a veto.</p>
      </div>
    </div>
  </div>
</template>

<script setup>
// No dynamic data needed — this is a reference document
</script>

<style scoped>
.card {
  background: #161b22;
  border: 1px solid #30363d;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1rem;
}

.card h2 {
  color: #f0f6fc;
  margin-bottom: 0.5rem;
  font-size: 1.4rem;
}

.card h3 {
  color: #58a6ff;
  margin-bottom: 1rem;
  margin-top: 1.5rem;
  font-size: 1.1rem;
  border-bottom: 1px solid #21262d;
  padding-bottom: 0.5rem;
}

.card h4 {
  color: #f0f6fc;
  margin-bottom: 0.75rem;
  margin-top: 1rem;
  font-size: 0.95rem;
}

.card h5 {
  color: #d2a8ff;
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
}

.card p, .card li {
  color: #c9d1d9;
  line-height: 1.6;
  font-size: 0.9rem;
}

.card ul, .card ol {
  margin-left: 1.5rem;
  margin-bottom: 0.75rem;
}

code {
  background: #0d1117;
  padding: 0.15rem 0.4rem;
  border-radius: 3px;
  font-size: 0.85rem;
  color: #79c0ff;
  font-family: 'SF Mono', Monaco, 'Cascadia Code', monospace;
}

.note {
  color: #8b949e;
  font-size: 0.85rem;
  font-style: italic;
  margin-top: 0.5rem;
}

/* Architecture Grid */
.arch-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
  margin-top: 1rem;
}

.arch-box {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 6px;
  padding: 1rem;
}

.arch-box h4 {
  color: #d29922;
  font-size: 0.9rem;
  margin-bottom: 0.5rem;
}

.arch-box p {
  font-size: 0.85rem;
  line-height: 1.5;
}

/* Schema Tables */
.table-title {
  color: #f0f6fc !important;
  font-size: 1rem !important;
  font-family: 'SF Mono', Monaco, monospace;
  background: #0d1117;
  padding: 0.5rem 0.75rem;
  border-radius: 4px;
  display: inline-block;
  margin-bottom: 0.5rem !important;
}

.table-desc {
  font-size: 0.85rem;
  margin-bottom: 0.75rem;
}

.schema-table, .recur-table, .interval-status-table, .budget-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 1rem;
  font-size: 0.85rem;
}

.schema-table th, .recur-table th, .interval-status-table th, .budget-table th {
  background: #0d1117;
  color: #8b949e;
  text-align: left;
  padding: 0.75rem;
  border-bottom: 2px solid #21262d;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.schema-table td, .recur-table td, .interval-status-table td, .budget-table td {
  padding: 0.75rem;
  border-bottom: 1px solid #21262d;
  vertical-align: top;
}

.schema-table code, .recur-table code, .interval-status-table code, .budget-table code {
  background: none;
  color: #d2a8ff;
  padding: 0;
}

/* Task Type Grid */
.type-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1rem;
  margin-top: 1rem;
}

.type-box {
  border: 2px solid;
  border-radius: 6px;
  padding: 1rem;
  background: #0d1117;
}

.type-box.permanent { border-color: #3fb950; }
.type-box.recurring { border-color: #58a6ff; }
.type-box.candidate { border-color: #d29922; }
.type-box.adhoc { border-color: #f85149; }

.type-box h4 {
  margin-top: 0;
  font-size: 1rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.type-box.permanent h4 { color: #3fb950; }
.type-box.recurring h4 { color: #58a6ff; }
.type-box.candidate h4 { color: #d29922; }
.type-box.adhoc h4 { color: #f85149; }

.type-box p {
  font-size: 0.85rem;
  margin-bottom: 0.5rem;
}

.type-box p strong {
  color: #f0f6fc;
}

/* Stack Grid */
.stack-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
  margin-top: 1rem;
}

.stack-box {
  border: 2px solid;
  border-radius: 6px;
  padding: 1rem;
  background: #0d1117;
}

.stack-box.morning { border-color: #f0c040; }
.stack-box.after-work { border-color: #58a6ff; }
.stack-box.after-dinner { border-color: #d2a8ff; }
.stack-box.weekend { border-color: #3fb950; }

.stack-box h4 {
  margin-top: 0;
  font-size: 0.95rem;
}

.stack-box p {
  font-size: 0.85rem;
  margin-bottom: 0.4rem;
}

.stack-box p strong {
  color: #f0f6fc;
}

/* Code Blocks */
.code-ref {
  font-family: 'SF Mono', Monaco, monospace;
  color: #79c0ff;
  font-size: 0.9rem;
  margin-top: 1.5rem !important;
}

.code-block {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 4px;
  padding: 1rem;
  overflow-x: auto;
  font-size: 0.8rem;
  line-height: 1.5;
  color: #c9d1d9;
  font-family: 'SF Mono', Monaco, monospace;
  margin-bottom: 1rem;
}

/* Tool Grid */
.tool-grid {
  display: grid;
  gap: 1rem;
  margin-top: 1rem;
}

.tool-box {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 6px;
  padding: 1rem;
}

.tool-box h5 {
  font-family: 'SF Mono', Monaco, monospace;
  background: #161b22;
  padding: 0.4rem 0.6rem;
  border-radius: 4px;
  display: inline-block;
  margin-bottom: 0.75rem;
}

.tool-box p {
  font-size: 0.85rem;
  margin-bottom: 0.5rem;
}

.tool-box ol, .tool-box ul {
  font-size: 0.85rem;
}

/* Lifecycle */
.lifecycle-section {
  margin-top: 1.5rem;
}

.lifecycle-section h4 {
  color: #58a6ff;
  margin-bottom: 1rem;
}

.lifecycle-diagram {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 6px;
  padding: 1.5rem;
  margin-bottom: 1rem;
}

.lifecycle-node {
  background: #161b22;
  border: 1px solid #30363d;
  border-radius: 4px;
  padding: 0.75rem;
  font-size: 0.85rem;
  line-height: 1.5;
  margin-bottom: 0.5rem;
}

.lifecycle-arrow {
  text-align: center;
  color: #58a6ff;
  font-size: 1.2rem;
  margin: 0.25rem 0;
}

.lifecycle-flow {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  margin-top: 1rem;
}

.lifecycle-step {
  background: #0d1117;
  border-left: 3px solid #58a6ff;
  padding: 0.75rem 1rem;
  font-size: 0.85rem;
  line-height: 1.5;
}

.lifecycle-step strong {
  color: #58a6ff;
}

/* Planner Steps */
.planner-step {
  background: #0d1117;
  border: 1px solid #21262d;
  border-left: 3px solid #d29922;
  border-radius: 4px;
  padding: 1rem;
  margin-bottom: 1rem;
}

.planner-step h4 {
  color: #d29922;
  margin-top: 0;
}

/* Cron Boxes */
.cron-box {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 6px;
  padding: 1rem;
  margin-bottom: 1rem;
}

.cron-box h4 {
  color: #58a6ff;
  margin-top: 0;
}

.nudge-example {
  background: #161b22;
  border: 1px solid #30363d;
  border-radius: 4px;
  padding: 1rem;
  font-size: 0.85rem;
  line-height: 1.6;
  color: #c9d1d9;
  font-family: 'SF Mono', Monaco, monospace;
  white-space: pre-wrap;
  margin-top: 0.5rem;
}

/* Weekend Sections */
.weekend-section {
  background: #0d1117;
  border-left: 3px solid #3fb950;
  padding: 1rem;
  margin-bottom: 1rem;
}

.weekend-section h4 {
  color: #3fb950;
  margin-top: 0;
}

/* Rules */
.rule-box {
  border: 2px solid;
  border-radius: 6px;
  padding: 1rem;
  margin-bottom: 1rem;
}

.rule-box.critical {
  border-color: #f85149;
  background: rgba(248, 81, 73, 0.05);
}

.rule-box.important {
  border-color: #3fb950;
  background: rgba(63, 185, 80, 0.05);
}

.rule-box h4 {
  margin-top: 0;
  font-size: 0.95rem;
}

.rule-box.critical h4 {
  color: #f85149;
}

.rule-box.important h4 {
  color: #3fb950;
}

.sync-flow {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 6px;
  padding: 1rem;
  margin-bottom: 1rem;
}

.sync-flow h4 {
  margin-top: 0;
}

@media (max-width: 768px) {
  .arch-grid, .type-grid, .stack-grid {
    grid-template-columns: 1fr;
  }
}
</style>
