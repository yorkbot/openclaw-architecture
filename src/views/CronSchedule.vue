<template>
  <div>
    <div class="card">
      <h2>⏰ Cron Schedule</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Scheduled jobs across all agents. All times in ET.
        <strong>Built</strong> = agent and skill ready, cron not yet switched to the new agent.
        <strong>On</strong> = currently running on the listed agent.
        Existing main agent crons will be retired when their replacement agent's cron is activated.
      </p>
    </div>

    <div class="card" v-for="group in cronGroups" :key="group.label">
      <h3>{{ group.icon }} {{ group.label }}</h3>
      <table class="cron-table">
        <thead>
          <tr>
            <th>Time</th>
            <th>Job</th>
            <th>Current Agent</th>
            <th>Target Agent</th>
            <th>Status</th>
            <th>Notes</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="job in group.jobs" :key="job.name" :class="{ 'row-disabled': job.status === 'Not built' }">
            <td class="cron-time">{{ job.time }}</td>
            <td>{{ job.name }}</td>
            <td>{{ job.currentAgent }}</td>
            <td>{{ job.targetAgent }}</td>
            <td><span :class="['status-tag', statusClass(job.status)]">{{ job.status }}</span></td>
            <td class="cron-notes">{{ job.notes }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="card">
      <h3>Migration Plan</h3>
      <p style="color: #8b949e; line-height: 1.6;">
        All crons currently run on the <code>main</code> agent. As specialized agents are verified,
        their crons will be switched from main → target agent. "Built" means the agent workspace and
        skills are ready — just need cron activation. When all are migrated, the main agent's
        duplicate crons are deleted.
      </p>
    </div>
  </div>
</template>

<script setup>
const cronGroups = [
  {
    icon: '🌙',
    label: 'Overnight (1 AM – 7 AM)',
    jobs: [
      {
        time: '12:30 AM',
        name: 'Queue Audit',
        currentAgent: 'main',
        targetAgent: 'York 🦝',
        status: 'On',
        notes: 'Cleans improvement-queue.md: moves done/rejected items, dedupes',
      },
      {
        time: 'Every 30m, 1–6 AM',
        name: 'Overnight Work',
        currentAgent: 'main',
        targetAgent: 'York 🦝',
        status: 'On',
        notes: 'Picks one task from overnight queue, executes or spawns sub-agent',
      },
      {
        time: '3:45 AM',
        name: 'D&D Questions Cache',
        currentAgent: 'main (default)',
        targetAgent: 'Caedmon 🐉',
        status: 'Built',
        notes: 'Finds wiki gaps, spawns research sub-agents, writes worldbuilding questions. Caedmon skill ready.',
      },
      {
        time: '4:15 AM',
        name: 'Nutrition + Weight Cache',
        currentAgent: 'main (default)',
        targetAgent: 'Dagr 🐓',
        status: 'Built',
        notes: 'Caches yesterday\'s nutrition and weight to daily memory file. Dagr skill ready.',
      },
      {
        time: '5:15 AM',
        name: 'Consumption Gap Cache',
        currentAgent: 'main (default)',
        targetAgent: 'Dagr 🐓',
        status: 'Built',
        notes: 'Checks for missing consumption data. Dagr skill ready.',
      },
      {
        time: '6:30 AM + 6:30 PM',
        name: 'Calendar Context',
        currentAgent: 'main',
        targetAgent: 'Dagr 🐓',
        status: 'Built',
        notes: 'Fetches Google Calendar, writes to daily memory. Dagr skill ready.',
      },
    ],
  },
  {
    icon: '🌅',
    label: 'Morning (7 AM – 9 AM)',
    jobs: [
      {
        time: '7:30 AM + 7:30 PM',
        name: 'Weather Cache',
        currentAgent: 'main (unassigned)',
        targetAgent: 'Dagr 🐓',
        status: 'Built',
        notes: 'Fetches weather for Cleveland Heights (+ Ann Arbor if traveling). Dagr skill ready.',
      },
      {
        time: '7:30 AM',
        name: 'Compile Overnight',
        currentAgent: 'main',
        targetAgent: 'Dagr 🐓',
        status: 'Built',
        notes: 'Compiles overnight sub-agent results into overnight-results.md. Dagr skill ready.',
      },
      {
        time: '7:40 AM Mon',
        name: 'Weekly Banner',
        currentAgent: 'main',
        targetAgent: 'York 🦝',
        status: 'On',
        notes: 'Generates Discord server banner via Gemini image gen. Stays on main/York.',
      },
      {
        time: '7:45 AM',
        name: 'Daily Avatar',
        currentAgent: 'main',
        targetAgent: 'York 🦝',
        status: 'On',
        notes: 'Generates daily Discord avatar via Gemini image gen. Stays on main/York.',
      },
      {
        time: '8:00 AM',
        name: 'Morning Orchestrator',
        currentAgent: 'main',
        targetAgent: 'Dagr 🐓',
        status: 'Built',
        notes: 'Master brief compilation. Reads all caches, verifies, composes, posts. Dagr skill ready.',
      },
    ],
  },
  {
    icon: '🔄',
    label: 'Periodic',
    jobs: [
      {
        time: 'Every 30m (8AM–12:30AM)',
        name: 'Heartbeat',
        currentAgent: 'main',
        targetAgent: 'York 🦝',
        status: 'On',
        notes: 'Session continuity check. Silent most beats. Stays on main/York.',
      },
      {
        time: '8:00 PM Sun',
        name: 'Weekly Check-in',
        currentAgent: 'main',
        targetAgent: 'York 🦝',
        status: 'On',
        notes: 'Sunday evening health/fitness recap to #general. Stays on main/York.',
      },
      {
        time: '11:00 PM',
        name: 'Daily Memory Summary',
        currentAgent: 'main (unassigned)',
        targetAgent: 'York 🦝',
        status: 'On',
        notes: 'End-of-day summary of all sessions to memory file. Stays on main/York.',
      },
    ],
  },
  {
    icon: '🔮',
    label: 'Planned (Not Yet Built)',
    jobs: [
      {
        time: '1:00–3:30 AM',
        name: 'Memory Audits (6 agents)',
        currentAgent: '—',
        targetAgent: 'Bede 🦉',
        status: 'Not built',
        notes: 'Audit each agent\'s workspace memory quality. Bede spawns Sonnet sub-agents.',
      },
    ],
  },
]

function statusClass(status) {
  if (status === 'On') return 'status-on'
  if (status === 'Built') return 'status-built'
  if (status === 'Not built') return 'status-notbuilt'
  return ''
}
</script>

<style scoped>
.cron-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.85rem;
}

.cron-table th {
  text-align: left;
  color: #8b949e;
  padding: 0.5rem 0.75rem;
  border-bottom: 1px solid #30363d;
  font-weight: 600;
}

.cron-table td {
  padding: 0.5rem 0.75rem;
  border-bottom: 1px solid #21262d;
  color: #c9d1d9;
}

.cron-time {
  font-family: monospace;
  color: #79c0ff;
  white-space: nowrap;
}

.cron-notes {
  color: #8b949e;
  font-size: 0.8rem;
}

.row-disabled {
  opacity: 0.7;
}

.status-tag {
  font-size: 0.75rem;
  padding: 0.15rem 0.5rem;
  border-radius: 10px;
  white-space: nowrap;
}

.status-on {
  background: rgba(63, 185, 80, 0.15);
  color: #3fb950;
}

.status-built {
  background: rgba(31, 111, 235, 0.15);
  color: #58a6ff;
}

.status-notbuilt {
  background: rgba(139, 148, 158, 0.15);
  color: #8b949e;
}

code {
  background: #161b22;
  padding: 0.15rem 0.4rem;
  border-radius: 3px;
  font-size: 0.85em;
  color: #79c0ff;
}
</style>
