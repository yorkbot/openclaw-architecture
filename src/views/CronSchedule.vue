<template>
  <div>
    <div class="card">
      <h2>⏰ Cron Schedule</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Scheduled jobs across all agents. All times in ET. None of these are active yet —
        they'll be turned on as agents come online. Existing main agent crons will be retired
        when their replacement agent is operational.
      </p>
    </div>

    <div class="card" v-for="group in cronGroups" :key="group.label">
      <h3>{{ group.icon }} {{ group.label }}</h3>
      <table class="cron-table">
        <thead>
          <tr>
            <th>Time</th>
            <th>Job</th>
            <th>Agent</th>
            <th>Model</th>
            <th>Status</th>
            <th>Notes</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="job in group.jobs" :key="job.name" :class="{ 'row-disabled': job.status === 'Not built' }">
            <td class="cron-time">{{ job.time }}</td>
            <td>{{ job.name }}</td>
            <td>{{ job.agent }}</td>
            <td>{{ job.model }}</td>
            <td><span :class="['status-tag', statusClass(job.status)]">{{ job.status }}</span></td>
            <td class="cron-notes">{{ job.notes }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="card">
      <h3>Migration Plan</h3>
      <p style="color: #8b949e; line-height: 1.6;">
        The current <code>main</code> agent has cron jobs for morning brief, nutrition/weight cache,
        consumption gap cache, D&D questions cache, and overnight work. As each specialized agent
        comes online, its cron replaces the corresponding main agent cron. When all are migrated,
        the main agent's crons are deleted.
      </p>
    </div>
  </div>
</template>

<script setup>
const cronGroups = [
  {
    icon: '🌅',
    label: 'Morning',
    jobs: [
      {
        time: '7:30 AM',
        name: 'Weather + Calendar Cache',
        agent: 'Dagr 🐓',
        model: 'Sonnet',
        status: 'Not built',
        notes: 'Replaces main agent weather/calendar cache crons',
      },
      {
        time: '7:30 AM',
        name: 'Nutrition + Weight Cache',
        agent: 'Bede 🦉',
        model: 'Sonnet sub-agent',
        status: 'Not built',
        notes: 'Replaces main agent nutrition-weight-cache cron',
      },
      {
        time: '8:00 AM',
        name: 'Morning Brief',
        agent: 'Dagr 🐓',
        model: 'Sonnet',
        status: 'Not built',
        notes: 'Replaces main agent morning-orchestrator cron',
      },
      {
        time: '8:15 AM',
        name: 'Consumption Gap Follow-up',
        agent: 'Bede 🦉',
        model: 'Sonnet sub-agent',
        status: 'Not built',
        notes: 'Replaces main agent consumption-gap cron',
      },
    ],
  },
  {
    icon: '🌙',
    label: 'Overnight',
    jobs: [
      {
        time: '1:00 AM',
        name: 'Memory Audit — Offa',
        agent: 'Offa 🦫',
        model: 'Sonnet sub-agent',
        status: 'Not built',
        notes: 'Audit Offa workspace memory quality',
      },
      {
        time: '1:30 AM',
        name: 'Memory Audit — Wynn',
        agent: 'Wynn 🦌',
        model: 'Sonnet sub-agent',
        status: 'Not built',
        notes: 'Audit Wynn workspace memory quality',
      },
      {
        time: '2:00 AM',
        name: 'Memory Audit — Hild',
        agent: 'Hild 🦡',
        model: 'Sonnet sub-agent',
        status: 'Not built',
        notes: 'Audit Hild workspace memory quality',
      },
      {
        time: '2:30 AM',
        name: 'Memory Audit — Caedmon',
        agent: 'Caedmon 🐉',
        model: 'Sonnet sub-agent',
        status: 'Not built',
        notes: 'Audit Caedmon workspace memory quality',
      },
      {
        time: '3:00 AM',
        name: 'Memory Audit — Dagr',
        agent: 'Dagr 🐓',
        model: 'Sonnet sub-agent',
        status: 'Not built',
        notes: 'Audit Dagr workspace memory quality',
      },
      {
        time: '3:30 AM',
        name: 'Memory Audit — York',
        agent: 'York 🦝',
        model: 'Sonnet sub-agent',
        status: 'Not built',
        notes: 'Audit York workspace memory quality',
      },
      {
        time: '3:30 AM',
        name: 'D&D Wiki Research',
        agent: 'Caedmon 🐉',
        model: 'Opus 4.6',
        status: 'Not built',
        notes: 'Explore wiki, find gaps, generate worldbuilding questions. Replaces main agent dnd-questions-cache cron.',
      },
    ],
  },
  {
    icon: '🔄',
    label: 'Periodic',
    jobs: [
      {
        time: 'Every 30m (8AM-12:30AM)',
        name: 'Heartbeat',
        agent: 'York 🦝',
        model: 'Opus 4.6',
        status: 'Not built',
        notes: 'Replaces main agent heartbeat. Silent most beats.',
      },
    ],
  },
]

function statusClass(status) {
  if (status === 'Active') return 'status-active'
  if (status === 'Not built') return 'status-notbuilt'
  if (status === 'Planned') return 'status-planned'
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

.status-active {
  background: rgba(63, 185, 80, 0.15);
  color: #3fb950;
}

.status-notbuilt {
  background: rgba(139, 148, 158, 0.15);
  color: #8b949e;
}

.status-planned {
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
