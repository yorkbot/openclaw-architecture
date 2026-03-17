<template>
  <div>
    <div class="card">
      <h2>⏰ Cron Schedule</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        All scheduled jobs across agents. Times in ET.
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
            <th>Status</th>
            <th>Notes</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="job in group.jobs" :key="job.name">
            <td class="cron-time">{{ job.time }}</td>
            <td>{{ job.name }}</td>
            <td>{{ job.agent }}</td>
            <td><span :class="['status-tag', statusClass(job.status)]">{{ job.status }}</span></td>
            <td class="cron-notes">{{ job.notes }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
const cronGroups = [
  {
    icon: '🌙',
    label: 'Overnight: Maintenance (12:00 – 4:30 AM)',
    desc: 'DB backup + per-agent memory quality audit via Sonnet sub-agent. Skill: skills/memory-audit/',
    jobs: [
      { time: '12:00 AM', name: 'york-data DB Backup', agent: 'Offa 🦫', status: 'On', notes: 'sqlite3 .backup to ~/york-data/backups/. 14-day rotation.' },
      { time: '1:00 AM', name: 'Memory Audit — York', agent: 'York 🦊 (main)', status: 'On', notes: '' },
      { time: '1:30 AM', name: 'Memory Audit — Offa', agent: 'Offa 🦫', status: 'On', notes: '' },
      { time: '2:00 AM', name: 'Memory Audit — Wynn', agent: 'Wynn 🦌', status: 'On', notes: '' },
      { time: '2:30 AM', name: 'Memory Audit — Caedmon', agent: 'Caedmon 🐉', status: 'On', notes: '' },
      { time: '3:00 AM', name: 'Memory Audit — Dagr', agent: 'Dagr 🐓', status: 'On', notes: '' },
      { time: '3:30 AM', name: 'Memory Audit — Wiglaf', agent: 'Wiglaf 🐻', status: 'On', notes: '' },
      { time: '3:45 AM', name: 'Memory Audit — Hild', agent: 'Hild 🦡', status: 'On', notes: '' },
      { time: '4:15 AM', name: 'Memory Audit — Guthlac', agent: 'Guthlac 🐺', status: 'On', notes: 'Sonnet sub-agent. Excluded from Bede analysis.' },
    ],
  },
  {
    icon: '🌅',
    label: 'Morning: Cache + Prep (4:00 – 8:00 AM)',
    jobs: [
      { time: '4:00 AM', name: 'Wiki Improvement', agent: 'Caedmon 🐉', status: 'On', notes: 'Find one sparse article, research, improve, open PR, write questions for morning brief. Opus 4.6.' },
      { time: '4:15 AM', name: 'Nutrition + Weight Cache', agent: 'main (temp)', status: 'On', notes: 'Stays on main until Bede is built.' },
      { time: '4:30 AM / 7:30 PM', name: 'Weather Cache (5-day)', agent: 'Dagr 🐓 (script)', status: 'On', notes: 'Open-Meteo API → shared-cache. Runs before planner so Hild has weather context.' },
      { time: '4:45 AM / 6:30 PM', name: 'Calendar Context', agent: 'Dagr 🐓 (script)', status: 'On', notes: 'Bash script → shared-cache. Runs before planner so Hild has calendar context.' },
      { time: '5:00 AM', name: 'Consumption Gap Cache', agent: 'main (temp)', status: 'On', notes: 'Stays on main until Bede is built.' },
      { time: '5:30 AM', name: 'Daily Planner', agent: 'Hild 🦡 (Opus)', status: 'On', notes: 'Sync yesterday, read weather/calendar, compute stacks, check interval tasks, evaluate growth (weather/calendar-aware), sync morning to Google Tasks. Daily.' },
      { time: '6:00 AM', name: 'Workout Cache', agent: 'Wynn 🦌 (script)', status: 'On', notes: 'Reads active program, checks weather, swaps outdoor/indoor if needed, writes to shared-cache. Daily.' },
      { time: '7:45 AM', name: 'Daily Avatar', agent: 'York 🦊 (main)', status: 'On', notes: 'york-tools.image_generate (Gemini Pro). 1:1.' },
      { time: '7:00 AM M-F', name: 'Morning Prep', agent: 'Wiglaf 🐻', status: 'On', notes: 'Opus. Creates daily note, ages board, reads calendar cache, refreshes now.md.' },
      { time: '8:00 AM', name: 'Morning Brief', agent: 'Dagr 🐓', status: 'On', notes: 'Posts to #🦊york.' },
    ],
  },
  {
    icon: '🔄',
    label: 'Daytime: Periodic',
    jobs: [
      { time: '3:00 AM Mon', name: 'Banner — Monday', agent: 'York 🦊 (main)', status: 'On', notes: 'york-tools.image_generate (Gemini Pro). 16:9.' },
      { time: '4:00 PM Fri', name: 'Banner — Friday', agent: 'York 🦊 (main)', status: 'On', notes: 'york-tools.image_generate (Gemini Pro). 16:9. Weekend fresh.' },
      { time: '4:00 PM M-F', name: 'After-Work Reset + Nudge', agent: 'Hild 🦡', status: 'On', notes: 'Reads after-work stack from york-data, syncs to Google Tasks, posts to #hild. Checks morning completion.' },
      { time: '7:00 PM', name: 'After-Dinner Reset + Nudge', agent: 'Hild 🦡', status: 'On', notes: 'Reads after-dinner stack from york-data, syncs to Google Tasks, posts to #hild. Last check on today\'s stacks. Daily.' },

      { time: 'Every 30m (8AM–12:30AM)', name: 'Heartbeat', agent: 'York 🦊 (main)', status: 'On', notes: 'Silent most beats.' },
      { time: '8:00 PM Sun', name: 'Weekly Check-in', agent: 'York 🦊 (main)', status: 'On', notes: 'Health/fitness recap to #general.' },
      { time: 'Weekly (last week of program)', name: 'Program Review', agent: 'Wynn 🦌 (Opus)', status: 'On', notes: 'Review current program adherence, design next program. Spawns Opus sub-agent.' },
    ],
  },
]

function statusClass(status) {
  if (status === 'On') return 'status-on'
  if (status === 'Not built') return 'status-notbuilt'
  if (status === 'Not wired') return 'status-notwired'
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

.status-notbuilt {
  background: rgba(139, 148, 158, 0.15);
  color: #8b949e;
}

.status-notwired {
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
