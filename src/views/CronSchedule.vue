<template>
  <div>
    <div class="card">
      <h2>⏰ Cron Schedule</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        All times ET. Session resets at <strong>1:00 AM</strong>. Day starts at <strong>8:00 AM</strong>.
        Night window: 1:00 AM – 7:55 AM. Nothing runs during the day except heartbeat and weekly check-in.
      </p>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        <span class="status-tag status-on">On</span> = currently running.
        <span class="status-tag status-built">Built</span> = agent/script ready, not yet switched.
        <span class="status-tag status-notbuilt">Not built</span> = planned.
        <span class="status-tag status-script">Script</span> = no LLM, runs as bash.
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
            <th>LLM?</th>
            <th>Status</th>
            <th>Notes</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="job in group.jobs" :key="job.name" :class="{ 'row-disabled': job.status === 'Not built' }">
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
        All crons currently run on <code>main</code>. "Built" items have the target agent's workspace
        and skills ready — just need the cron pointed at the new agent. "Script" items need no LLM
        and will run as bash via exec cron. When verified, <code>main</code> duplicates are deleted.
      </p>
    </div>
  </div>
</template>

<script setup>
const cronGroups = [
  {
    icon: '🌙',
    label: 'Part 1: Memory (1:00 AM – 3:00 AM)',
    desc: 'Per-agent memory management. Each agent gets a slot to summarize its sessions and clean memory. Staggered to avoid overlap.',
    jobs: [
      {
        time: '1:00 AM',
        name: 'York Memory Summary',
        agent: 'York 🦝',
        llm: 'Yes',
        status: 'Not built',
        notes: 'Summarize York\'s sessions, write to memory file',
      },
      {
        time: '1:15 AM',
        name: 'Queue Audit',
        agent: 'York 🦝',
        llm: 'Yes',
        status: 'On',
        notes: 'Clean improvement-queue.md: done/rejected items, dedupes. Currently at 12:30 AM.',
      },
      {
        time: '1:30 AM',
        name: 'Offa Memory Summary',
        agent: 'Offa 🦫',
        llm: 'Yes',
        status: 'Not built',
        notes: 'Summarize Offa\'s build sessions, update memory',
      },
      {
        time: '2:00 AM',
        name: 'Wynn Memory Summary',
        agent: 'Wynn 🦌',
        llm: 'Yes',
        status: 'Not built',
        notes: 'Summarize health logging sessions, update memory',
      },
      {
        time: '2:30 AM',
        name: 'Caedmon Memory Summary',
        agent: 'Caedmon 🐉',
        llm: 'Yes',
        status: 'Not built',
        notes: 'Summarize D&D sessions, update wiki-related memory',
      },
      {
        time: '3:00 AM',
        name: 'Dagr Memory Summary',
        agent: 'Dagr 🐓',
        llm: 'Yes',
        status: 'Not built',
        notes: 'Summarize brief corrections, format preferences',
      },
    ],
  },
  {
    icon: '🔧',
    label: 'Part 2: Overnight Work (1:00 AM – 6:00 AM)',
    desc: 'Long-running overnight tasks that need LLM reasoning.',
    jobs: [
      {
        time: 'Every 30m, 1–6 AM',
        name: 'Overnight Work',
        agent: 'York 🦝',
        llm: 'Yes',
        status: 'On',
        notes: 'Picks one task from overnight queue, executes or spawns sub-agent',
      },
      {
        time: '3:45 AM',
        name: 'D&D Questions Cache',
        agent: 'Caedmon 🐉',
        llm: 'Yes',
        status: 'Built',
        notes: 'Wiki research, worldbuilding questions. Needs creative reasoning.',
      },
    ],
  },
  {
    icon: '🌅',
    label: 'Part 3: Morning Cache + Prep (4:00 AM – 7:55 AM)',
    desc: 'Data fetching and brief preparation. Scripts where possible — no LLM for data pulls.',
    jobs: [
      {
        time: '4:15 AM',
        name: 'Nutrition + Weight Cache',
        agent: 'Wynn 🦌',
        llm: 'No — script',
        status: 'Built',
        notes: 'Fetch Google Sheets rows, extract fields, write to memory. Bash + python.',
      },
      {
        time: '5:15 AM',
        name: 'Consumption Gap Cache',
        agent: 'Wynn 🦌',
        llm: 'No — script',
        status: 'Built',
        notes: 'Read nutrition cache, check empty fields, write gap questions. Bash + python.',
      },
      {
        time: '6:30 AM',
        name: 'Calendar Context',
        agent: 'Script',
        llm: 'No — script',
        status: 'Built',
        notes: 'mcporter call google-calendar → parse JSON → write to memory. Also runs at 6:30 PM.',
      },
      {
        time: '7:30 AM',
        name: 'Weather Cache',
        agent: 'Script',
        llm: 'No — script',
        status: 'Built',
        notes: 'curl wttr.in → write to memory. Location-aware (Cleveland/Ann Arbor).',
      },
      {
        time: '7:40 AM Mon',
        name: 'Weekly Banner',
        agent: 'York 🦝',
        llm: 'Yes',
        status: 'On',
        notes: 'Generate Discord server banner via Gemini image gen.',
      },
      {
        time: '7:45 AM',
        name: 'Daily Avatar',
        agent: 'York 🦝',
        llm: 'Yes',
        status: 'On',
        notes: 'Generate daily Discord avatar via Gemini image gen.',
      },
      {
        time: '8:00 AM',
        name: 'Morning Brief',
        agent: 'Dagr 🐓',
        llm: 'Yes',
        status: 'Built',
        notes: 'Read all cached data, verify, editorialize, post to #dagr.',
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
        llm: 'Yes',
        status: 'On',
        notes: 'Session continuity. Silent most beats.',
      },
      {
        time: '8:00 PM Sun',
        name: 'Weekly Check-in',
        agent: 'York 🦝',
        llm: 'Yes',
        status: 'On',
        notes: 'Sunday evening health/fitness recap to #general.',
      },
    ],
  },
]

function statusClass(status) {
  if (status === 'On') return 'status-on'
  if (status === 'Built') return 'status-built'
  if (status === 'Not built') return 'status-notbuilt'
  if (status === 'Script') return 'status-script'
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

.status-script {
  background: rgba(210, 153, 34, 0.15);
  color: #d29922;
}

code {
  background: #161b22;
  padding: 0.15rem 0.4rem;
  border-radius: 3px;
  font-size: 0.85em;
  color: #79c0ff;
}
</style>
