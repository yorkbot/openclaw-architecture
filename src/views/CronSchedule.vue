<template>
  <div>
    <div class="card">
      <h2>⏰ Cron Schedule</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        All scheduled jobs across agents. Times in ET. Status reflects current state.
        Memory audit skills are built (shared-skills), crons just need activation.
      </p>
    </div>

    <div class="card" v-for="group in cronGroups" :key="group.label">
      <h3>{{ group.icon }} {{ group.label }}</h3>
      <p v-if="group.desc" style="color: #8b949e; margin-bottom: 0.75rem; font-size: 0.85rem;">{{ group.desc }}</p>
      <table class="cron-table">
        <thead>
          <tr>
            <th>Time</th>
            <th>Job</th>
            <th>Agent</th>
            <th>LLM</th>
            <th>Status</th>
            <th>Notes</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="job in group.jobs" :key="job.name">
            <td class="cron-time">{{ job.time }}</td>
            <td>{{ job.name }}</td>
            <td>{{ job.agent }}</td>
            <td>{{ job.llm }}</td>
            <td><span :class="['status-tag', statusClass(job.status)]">{{ job.status }}</span></td>
            <td class="cron-notes">{{ job.notes }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="card">
      <h3>Migration Plan</h3>
      <p style="color: #8b949e; line-height: 1.6;">
        Remaining <code>main</code> agent crons will be migrated or deleted as specialized agents come online.
        When all responsibilities are distributed, <code>main</code> is deleted entirely.
      </p>
    </div>
  </div>
</template>

<script setup>
const cronGroups = [
  {
    icon: '🌙',
    label: 'Part 1: Memory Audit (1:00 AM – 3:00 AM)',
    desc: 'Per-agent memory quality audit. Each agent gets a Sonnet sub-agent to review its memory files. Skills built at shared-skills/memory-audit/. Staggered to avoid overlap.',
    jobs: [
      {
        time: '1:00 AM',
        name: 'Memory Audit — York',
        agent: 'York 🦝',
        llm: 'Sonnet sub-agent',
        status: 'Skill built',
        notes: 'Audit York workspace memory quality',
      },
      {
        time: '1:30 AM',
        name: 'Memory Audit — Offa',
        agent: 'Offa 🦫',
        llm: 'Sonnet sub-agent',
        status: 'Skill built',
        notes: 'Audit Offa workspace memory quality',
      },
      {
        time: '2:00 AM',
        name: 'Memory Audit — Wynn',
        agent: 'Wynn 🦌',
        llm: 'Sonnet sub-agent',
        status: 'Skill built',
        notes: 'Audit Wynn workspace memory quality',
      },
      {
        time: '2:30 AM',
        name: 'Memory Audit — Caedmon',
        agent: 'Caedmon 🐉',
        llm: 'Sonnet sub-agent',
        status: 'Skill built',
        notes: 'Audit Caedmon workspace memory quality',
      },
      {
        time: '3:00 AM',
        name: 'Memory Audit — Dagr',
        agent: 'Dagr 🐓',
        llm: 'Sonnet sub-agent',
        status: 'Skill built',
        notes: 'Audit Dagr workspace memory quality',
      },
    ],
  },
  {
    icon: '🔧',
    label: 'Part 2: Overnight Research (3:00 AM – 4:00 AM)',
    desc: 'Long-running overnight tasks that need creative reasoning.',
    jobs: [
      {
        time: '3:45 AM',
        name: 'D&D Questions Cache',
        agent: 'Caedmon 🐉',
        llm: 'Opus 4.6',
        status: 'On (main)',
        notes: 'Wiki research, worldbuilding questions. Currently runs on main/default agent. Migrate to Caedmon.',
      },
    ],
  },
  {
    icon: '🌅',
    label: 'Part 3: Morning Cache + Prep (4:00 AM – 8:00 AM)',
    desc: 'Data fetching and brief preparation.',
    jobs: [
      {
        time: '4:15 AM',
        name: 'Nutrition + Weight Cache',
        agent: 'Bede 🦉',
        llm: 'Sonnet sub-agent',
        status: 'On (main)',
        notes: 'Fetch from york-data, write to memory. Currently on default agent.',
      },
      {
        time: '5:15 AM',
        name: 'Consumption Gap Cache',
        agent: 'Bede 🦉',
        llm: 'Sonnet sub-agent',
        status: 'On (main)',
        notes: 'Check for missing consumption data, write gap questions. Currently on default agent.',
      },
      {
        time: '6:30 AM',
        name: 'Calendar Context',
        agent: 'Dagr 🐓',
        llm: 'Yes',
        status: 'On (main)',
        notes: 'mcporter google-calendar → write to memory. Also runs at 6:30 PM.',
      },
      {
        time: '7:30 AM',
        name: 'Weather Cache',
        agent: 'Dagr 🐓',
        llm: 'Yes',
        status: 'On (main)',
        notes: 'curl wttr.in → write to memory. Location-aware. Also runs at 7:30 PM.',
      },
      {
        time: '7:40 AM Mon',
        name: 'Weekly Banner',
        agent: 'York 🦝',
        llm: 'Opus 4.6',
        status: 'On',
        notes: 'Generate Discord server banner via Gemini image gen.',
      },
      {
        time: '7:45 AM',
        name: 'Daily Avatar',
        agent: 'York 🦝',
        llm: 'Opus 4.6',
        status: 'On',
        notes: 'Generate daily Discord avatar via Gemini image gen.',
      },
      {
        time: '8:00 AM',
        name: 'Morning Brief',
        agent: 'Dagr 🐓',
        llm: 'Sonnet',
        status: 'On (main)',
        notes: 'Read cached data, editorialize, post. Currently runs on main. Migrate to Dagr.',
      },
    ],
  },
  {
    icon: '🔄',
    label: 'Daytime (Periodic)',
    desc: 'Runs during active hours only.',
    jobs: [
      {
        time: 'Every 30m (8AM–12:30AM)',
        name: 'Heartbeat',
        agent: 'York 🦝',
        llm: 'Opus 4.6',
        status: 'On',
        notes: 'Session continuity. Silent most beats.',
      },
      {
        time: '8:00 PM Sun',
        name: 'Weekly Check-in',
        agent: 'York 🦝',
        llm: 'Opus 4.6',
        status: 'On',
        notes: 'Sunday evening health/fitness recap to #general.',
      },
    ],
  },
]

function statusClass(status) {
  if (status === 'On') return 'status-on'
  if (status === 'On (main)') return 'status-on-main'
  if (status === 'Skill built') return 'status-built'
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

.status-on-main {
  background: rgba(210, 153, 34, 0.15);
  color: #d29922;
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
