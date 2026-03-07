<template>
  <div>
    <div class="card">
      <h2>Data Flow & Scheduling</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        How data moves between agents, scripts, and storage. Cron jobs produce cached files that other agents consume.
        This eliminates redundant API calls and keeps agent context small.
      </p>
      <p style="color: #c9d1d9; margin-bottom: 0; line-height: 1.6; font-size: 0.9rem; border-left: 3px solid #d29922; padding-left: 1rem;">
        <strong style="color: #d29922;">Service layer principle:</strong> All structured data (consumption, weight, workouts, chores)
        flows through typed service functions. Agents call <code style="color: #d29922;">log_food()</code>, not raw DB queries.
        External services that stay (Google Calendar) keep their existing MCP tools.
        The DB engine behind the service layer is an open question — any will work.
      </p>
    </div>

    <div class="card">
      <h3>Daily Timeline</h3>
      <div class="timeline">
        <div class="time-block" v-for="block in timeline" :key="block.time">
          <div class="time-label">{{ block.time }}</div>
          <div class="time-events">
            <div
              class="time-event"
              v-for="event in block.events"
              :key="event.name"
            >
              <span :class="['tag', event.tagClass]">{{ event.tag }}</span>
              <span class="event-name">{{ event.name }}</span>
              <span class="event-flow" v-if="event.flow">{{ event.flow }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="card">
      <h3>File-Based Agent Communication</h3>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Agents communicate through files, not direct calls. This is the key architectural pattern.
      </p>
      <div class="flow-diagram">
        <div class="flow-row" v-for="flow in flows" :key="flow.label">
          <div class="flow-producer">
            <span :class="['tag', flow.producerClass]">{{ flow.producer }}</span>
          </div>
          <div class="flow-arrow">
            <span class="flow-file">{{ flow.file }}</span>
            →
          </div>
          <div class="flow-consumer">
            <span :class="['tag', flow.consumerClass]">{{ flow.consumer }}</span>
          </div>
          <div class="flow-desc">{{ flow.desc }}</div>
        </div>
      </div>
    </div>

    <div class="card">
      <h3>Cannabis Gate Data Flow</h3>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        The gate is the most complex multi-agent interaction. Here's how data flows when James asks to smoke.
      </p>
      <div class="gate-flow">
        <div class="gate-step" v-for="(step, i) in gateSteps" :key="i">
          <div class="gate-num">{{ i + 1 }}</div>
          <div class="gate-content">
            <div class="gate-actor">
              <span :class="['tag', step.actorClass]">{{ step.actor }}</span>
            </div>
            <div class="gate-action">{{ step.action }}</div>
          </div>
        </div>
      </div>
    </div>

    <div class="card">
      <h3>Cron vs Heartbeat vs On-Demand</h3>
      <div class="comparison">
        <div class="comp-col">
          <h4>⏰ Cron</h4>
          <p>Exact timing, isolated session, no memory of prior runs. Best for: scheduled data collection, cache refresh, reports.</p>
          <ul>
            <li>Weather/calendar cache scripts</li>
            <li>Nutrition cache (4:15 AM)</li>
            <li>Morning brief (8:00 AM)</li>
            <li>Overnight work (1-6 AM)</li>
            <li>Weekly review (Sun 8 PM)</li>
            <li>Daily avatar (7:45 AM)</li>
            <li>DnD research (3:45 AM)</li>
          </ul>
        </div>
        <div class="comp-col">
          <h4>💓 Heartbeat</h4>
          <p>Session continuity, remembers earlier context. Best for: contextual awareness, nudges, judgment calls.</p>
          <ul>
            <li>York — every 30m, 8AM-12:30AM</li>
            <li>Evaluates: should I nudge?</li>
            <li>Updates panel presence</li>
            <li>Most beats are silent</li>
          </ul>
        </div>
        <div class="comp-col">
          <h4>🎯 On-Demand</h4>
          <p>Spawned by orchestrator when needed. Best for: user requests, reactive tasks.</p>
          <ul>
            <li>Food/weight logging</li>
            <li>Cannabis gate checks</li>
            <li>Research tasks</li>
            <li>Builder (implementation)</li>
            <li>Work prep</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const timeline = [
  {
    time: '1:00-6:00 AM',
    events: [
      { name: 'Overnight Worker', tag: 'Cron', tagClass: 'cron', flow: 'Picks task → executes or spawns Builder' },
    ],
  },
  {
    time: '3:45 AM',
    events: [
      { name: 'DnD Wiki Research', tag: 'Cron', tagClass: 'cron', flow: 'Explores wiki → writes questions to memory' },
    ],
  },
  {
    time: '4:15 AM',
    events: [
      { name: 'Nutrition Cache', tag: 'Cron', tagClass: 'cron', flow: 'Spreadsheet → memory/YYYY-MM-DD.md' },
    ],
  },
  {
    time: '5:15 AM',
    events: [
      { name: 'Consumption Gap Cache', tag: 'Cron', tagClass: 'cron', flow: 'Analyzes gaps → writes questions to memory' },
    ],
  },
  {
    time: '6:30 AM',
    events: [
      { name: 'Calendar Cache', tag: 'Script', tagClass: 'script', flow: 'Google Calendar → memory file' },
    ],
  },
  {
    time: '7:30 AM',
    events: [
      { name: 'Weather Cache', tag: 'Script', tagClass: 'script', flow: 'wttr.in → memory file' },
      { name: 'Compile Overnight', tag: 'Cron', tagClass: 'cron', flow: 'Sessions → overnight-results.md' },
    ],
  },
  {
    time: '7:45 AM',
    events: [
      { name: 'Daily Avatar', tag: 'Cron', tagClass: 'cron', flow: 'Memory → prompt → image → Discord' },
    ],
  },
  {
    time: '8:00 AM',
    events: [
      { name: 'Morning Brief', tag: 'Cron', tagClass: 'cron', flow: 'Reads ALL cached files → #general' },
      { name: 'York Heartbeat begins', tag: 'Heartbeat', tagClass: 'heartbeat', flow: 'Every 30m until 12:30 AM' },
    ],
  },
  {
    time: 'Throughout day',
    events: [
      { name: 'James conversations', tag: 'On-demand', tagClass: 'small', flow: '#general → York → sub-agents' },
      { name: 'Food/weight logging', tag: 'On-demand', tagClass: 'small', flow: 'York → Health agent → york-data service → local DB' },
      { name: 'Cannabis gate', tag: 'On-demand', tagClass: 'small', flow: 'York → Chores + Health → judgment' },
    ],
  },
  {
    time: '8:00 PM Sun',
    events: [
      { name: 'Weekly Review', tag: 'Cron', tagClass: 'cron', flow: 'Full week data → analysis → #general' },
    ],
  },
  {
    time: '11:00 PM',
    events: [
      { name: 'Daily Memory Summary', tag: 'Cron', tagClass: 'cron', flow: 'Sessions → memory/YYYY-MM-DD.md' },
    ],
  },
]

const flows = [
  {
    producer: 'Weather Script',
    producerClass: 'script',
    file: 'memory/YYYY-MM-DD.md ## Weather',
    consumer: 'Morning Brief',
    consumerClass: 'medium',
    desc: 'Pre-cached weather data, no live fetch at brief time',
  },
  {
    producer: 'Calendar Script',
    producerClass: 'script',
    file: 'memory/YYYY-MM-DD.md ## Calendar',
    consumer: 'Morning Brief',
    consumerClass: 'medium',
    desc: 'Pre-cached calendar events in ET',
  },
  {
    producer: 'Nutrition Cache',
    producerClass: 'small',
    file: 'york-data.get_consumption(yesterday) → memory file',
    consumer: 'Morning Brief',
    consumerClass: 'medium',
    desc: 'Yesterday\'s totals, trends, gaps — pulled from local DB, cached to memory',
  },
  {
    producer: 'DnD Research',
    producerClass: 'large',
    file: 'memory/YYYY-MM-DD.md ## DnD Questions',
    consumer: 'Morning Brief',
    consumerClass: 'medium',
    desc: 'Overnight wiki questions for James',
  },
  {
    producer: 'Health Agent',
    producerClass: 'small',
    file: 'york-data.get_trends(7d)',
    consumer: 'Weekly Review',
    consumerClass: 'large',
    desc: 'Full week of consumption/weight data via service layer',
  },
  {
    producer: 'Overnight Worker',
    producerClass: 'medium',
    file: 'overnight-results.md',
    consumer: 'Morning Brief',
    consumerClass: 'medium',
    desc: 'What York did overnight',
  },
]

const gateSteps = [
  { actor: 'James', actorClass: 'small', action: '"Can I smoke?"' },
  { actor: 'York (Large)', actorClass: 'large', action: 'Receives request. Checks time of day, location, recent context.' },
  { actor: 'Chores (Small)', actorClass: 'small', action: 'Spawned: grab camera snapshots, read chore-state.md, assess house state.' },
  { actor: 'Health (Small)', actorClass: 'small', action: 'Spawned: calls york-data.get_consumption(today) for calorie pace, checks dinner plan status.' },
  { actor: 'York (Large)', actorClass: 'large', action: 'Receives both reports. Makes judgment call: approve, deny, or negotiate. Considers patterns, time of day, calorie risk.' },
  { actor: 'York (Large)', actorClass: 'large', action: 'Delivers decision conversationally. If approved: logs session, mentions pending quick tasks (momentum stacking).' },
  { actor: 'Health (Small)', actorClass: 'small', action: 'Spawned: calls york-data.log_cannabis(time, session_number) to record the session.' },
]
</script>

<style scoped>
.timeline {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.time-block {
  display: flex;
  border-bottom: 1px solid #21262d;
  padding: 0.75rem 0;
}

.time-block:last-child {
  border-bottom: none;
}

.time-label {
  min-width: 140px;
  color: #58a6ff;
  font-weight: 600;
  font-size: 0.85rem;
  padding-top: 0.25rem;
}

.time-events {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  flex: 1;
}

.time-event {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  font-size: 0.85rem;
}

.event-name {
  color: #f0f6fc;
  font-weight: 500;
}

.event-flow {
  color: #8b949e;
  font-size: 0.8rem;
}

.flow-diagram {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.flow-row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.5rem 0;
  border-bottom: 1px solid #21262d;
  flex-wrap: wrap;
}

.flow-arrow {
  color: #484f58;
  font-size: 0.8rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.flow-file {
  color: #d29922;
  font-family: monospace;
  font-size: 0.75rem;
}

.flow-desc {
  color: #8b949e;
  font-size: 0.8rem;
}

.gate-flow {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.gate-step {
  display: flex;
  gap: 1rem;
  padding: 0.75rem 0;
  border-left: 2px solid #30363d;
  margin-left: 1rem;
  padding-left: 1rem;
}

.gate-step:last-child {
  border-left-color: transparent;
}

.gate-num {
  background: #30363d;
  color: #f0f6fc;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.75rem;
  font-weight: 700;
  flex-shrink: 0;
}

.gate-content {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.gate-action {
  color: #c9d1d9;
  font-size: 0.9rem;
}

.comparison {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.comp-col {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1rem;
}

.comp-col h4 {
  color: #f0f6fc;
  margin-bottom: 0.5rem;
}

.comp-col p {
  color: #8b949e;
  font-size: 0.8rem;
  line-height: 1.5;
  margin-bottom: 0.75rem;
}

.comp-col ul {
  list-style: none;
  padding: 0;
}

.comp-col li {
  color: #c9d1d9;
  font-size: 0.8rem;
  padding: 0.2rem 0;
}

.comp-col li::before {
  content: '· ';
  color: #484f58;
}

@media (max-width: 768px) {
  .comparison {
    grid-template-columns: 1fr;
  }
}
</style>
