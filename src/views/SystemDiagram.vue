<template>
  <div>
    <div class="card">
      <h2>System Overview</h2>
      <p style="color: #8b949e; margin-bottom: 1.5rem;">
        One OpenClaw instance on RPi. York orchestrates everything. Specialized agents handle domains.
        Scripts replace LLMs where possible. Cron for scheduled work, heartbeat for contextual awareness.
      </p>
    </div>

    <div class="diagram">
      <!-- James Layer -->
      <div class="layer">
        <div class="layer-label">JAMES</div>
        <div class="layer-content">
          <div class="node discord-node">
            <div class="node-icon">💬</div>
            <div class="node-title">Discord</div>
            <div class="node-detail">Primary interface</div>
          </div>
        </div>
      </div>

      <div class="connector-down">▼</div>

      <!-- Discord Channels -->
      <div class="layer">
        <div class="layer-label">DISCORD CHANNELS</div>
        <div class="layer-content">
          <div class="node channel-node" v-for="ch in channels" :key="ch.name">
            <div class="node-title">#{{ ch.name }}</div>
            <div class="node-detail">{{ ch.routes }}</div>
          </div>
        </div>
      </div>

      <div class="connector-down">▼</div>

      <!-- Orchestrator -->
      <div class="layer orchestrator-layer">
        <div class="layer-label">ORCHESTRATOR</div>
        <div class="layer-content">
          <div class="node orchestrator-node">
            <div class="node-icon">🦝</div>
            <div class="node-title">York</div>
            <div class="node-detail">
              <span class="tag large">Large Model</span>
              <span class="tag heartbeat">Heartbeat 30m</span>
            </div>
            <div class="node-desc">
              Conversation, judgment calls, routing, cannabis gate, accountability.
              Minimal own workspace — delegates everything else.
            </div>
          </div>
        </div>
      </div>

      <div class="connector-down">▼ spawns / routes</div>

      <!-- Agent Layer -->
      <div class="layer">
        <div class="layer-label">SPECIALIZED AGENTS</div>
        <div class="layer-content agents-grid">
          <div
            class="node agent-node"
            v-for="agent in agents"
            :key="agent.id"
            :class="agent.border"
          >
            <div class="node-title">{{ agent.name }}</div>
            <div class="node-detail">
              <span :class="['tag', agent.model]">{{ agent.modelLabel }}</span>
              <span :class="['tag', agent.schedule]" v-if="agent.scheduleLabel">{{ agent.scheduleLabel }}</span>
            </div>
            <div class="node-desc">{{ agent.desc }}</div>
          </div>
        </div>
      </div>

      <div class="connector-down">▼ read/write</div>

      <!-- Scripts Layer -->
      <div class="layer">
        <div class="layer-label">SCRIPTS (No LLM)</div>
        <div class="layer-content agents-grid">
          <div class="node script-node" v-for="s in scripts" :key="s.name">
            <div class="node-title">{{ s.name }}</div>
            <div class="node-detail">
              <span class="tag script">bash/cron</span>
            </div>
            <div class="node-desc">{{ s.desc }}</div>
          </div>
        </div>
      </div>

      <div class="connector-down">▼</div>

      <!-- Data Layer -->
      <div class="layer">
        <div class="layer-label">DATA & SERVICES</div>
        <div class="layer-content">
          <div class="node data-node" v-for="d in dataStores" :key="d.name">
            <div class="node-icon">{{ d.icon }}</div>
            <div class="node-title">{{ d.name }}</div>
          </div>
        </div>
      </div>
    </div>

    <!-- Separate Install -->
    <div class="card" style="margin-top: 2rem; border-color: #d29922;">
      <h2>🃏 Separate OpenClaw Instance (On-Demand)</h2>
      <div class="separate-grid">
        <div class="node agent-node border-yellow">
          <div class="node-title">MTG Agent</div>
          <div class="node-detail">
            <span class="tag medium">Medium Model</span>
          </div>
          <div class="node-desc">
            Own install, own Discord server or channel, own workspace.
            Toggled on when brewing/drafting, off otherwise. Zero cost when idle.
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const channels = [
  { name: 'general', routes: '→ York (orchestrator)' },
  { name: 'dnd', routes: '→ DnD agent direct' },
  { name: 'reviews', routes: '→ York (proposals)' },
  { name: 'health', routes: '→ York (private data)' },
]

const agents = [
  {
    id: 'health',
    name: '🍎 Health & Nutrition',
    model: 'sonnet',
    modelLabel: 'Sonnet',
    schedule: 'cron',
    scheduleLabel: 'Cron',
    desc: 'Food logging, calorie tracking, dashboard updates, weight trends, weekly analysis',
    border: 'border-blue',
  },
  {
    id: 'chores',
    name: '🏠 Chores & Home',
    model: 'sonnet',
    modelLabel: 'Sonnet',
    schedule: null,
    scheduleLabel: null,
    desc: 'Chore state, camera analysis, gate data prep. Spawned by York for gate checks.',
    border: 'border-blue',
  },
  {
    id: 'dnd',
    name: '🎲 D&D Campaign',
    model: 'sonnet',
    modelLabel: 'Sonnet',
    schedule: 'heartbeat',
    scheduleLabel: 'Heartbeat',
    desc: 'Wiki lookups, lore questions, worldbuilding. Direct channel routing for interactive sessions. Overnight research via cron.',
    border: 'border-blue',
  },
  {
    id: 'morning',
    name: '☀️ Morning Brief',
    model: 'sonnet',
    modelLabel: 'Sonnet',
    schedule: 'cron',
    scheduleLabel: 'Cron 8AM',
    desc: 'Read cached data files, format brief, post to #general. No live data fetching.',
    border: 'border-blue',
  },
  {
    id: 'research',
    name: '🔍 Research',
    model: 'grok',
    modelLabel: 'Varies',
    schedule: null,
    scheduleLabel: null,
    desc: 'Web search, browsing, synthesis. Model scales with task: quick lookup = cheap, deep research = Opus.',
    border: 'border-yellow',
  },
  {
    id: 'builder',
    name: '🔨 Builder',
    model: 'opus',
    modelLabel: 'Opus',
    schedule: null,
    scheduleLabel: null,
    desc: 'Feature implementation, skill creation, debugging. The hard stuff. Spawned on-demand.',
    border: 'border-red',
  },
  {
    id: 'overnight',
    name: '🌙 Overnight',
    model: 'sonnet',
    modelLabel: 'Sonnet',
    schedule: 'cron',
    scheduleLabel: 'Cron 1-6AM',
    desc: 'Self-improvement, queue work, wiki research. Dispatches to Builder for hard tasks.',
    border: 'border-blue',
  },
  {
    id: 'weekly',
    name: '📊 Weekly Review',
    model: 'opus',
    modelLabel: 'Large',
    schedule: 'cron',
    scheduleLabel: 'Cron Sun 8PM',
    desc: 'Pull full week data, calculate trends, form opinions. Needs analytical depth.',
    border: 'border-red',
  },
]

const scripts = [
  { name: 'weather-cache', desc: 'curl wttr.in → memory file' },
  { name: 'calendar-cache', desc: 'mcporter → memory file' },
  { name: 'weight-log', desc: 'mcporter → spreadsheet row' },
  { name: 'workout-log', desc: 'mcporter → spreadsheet row' },
  { name: 'queue-audit', desc: 'Cleanup improvement-queue.md' },
  { name: 'camera-snapshot', desc: 'ffmpeg RTSP → /tmp/ images' },
  { name: 'avatar-set', desc: 'Discord API avatar upload' },
  { name: 'panel-update', desc: 'Write status.json for genmon' },
]

const dataStores = [
  { icon: '📊', name: 'Google Sheets' },
  { icon: '📅', name: 'Google Calendar' },
  { icon: '📁', name: 'Memory Files' },
  { icon: '📖', name: 'D&D Wiki' },
  { icon: '📷', name: 'Cameras (RTSP)' },
  { icon: '🖥️', name: 'Panel (Genmon)' },
]
</script>

<style scoped>
.diagram {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0;
}

.layer {
  width: 100%;
  background: #161b22;
  border: 1px solid #30363d;
  border-radius: 8px;
  padding: 1.5rem;
}

.layer-label {
  font-size: 0.7rem;
  font-weight: 700;
  color: #8b949e;
  letter-spacing: 0.1em;
  margin-bottom: 1rem;
}

.layer-content {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  justify-content: center;
}

.orchestrator-layer {
  border-color: #f85149;
  background: #1a1215;
}

.connector-down {
  color: #484f58;
  font-size: 0.8rem;
  padding: 0.5rem;
  text-align: center;
}

.node {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1rem;
  min-width: 140px;
  text-align: center;
}

.node-icon {
  font-size: 1.5rem;
  margin-bottom: 0.25rem;
}

.node-title {
  font-weight: 600;
  color: #f0f6fc;
  font-size: 0.9rem;
  margin-bottom: 0.25rem;
}

.node-detail {
  margin-bottom: 0.5rem;
}

.node-desc {
  font-size: 0.75rem;
  color: #8b949e;
  line-height: 1.4;
  text-align: left;
}

.discord-node {
  border-color: #5865f2;
  min-width: 200px;
}

.channel-node {
  border-color: #5865f2;
  min-width: 160px;
}

.orchestrator-node {
  border-color: #f85149;
  min-width: 400px;
  max-width: 600px;
}

.agent-node {
  max-width: 250px;
  min-width: 200px;
}

.script-node {
  border-style: dashed;
  border-color: #56d364;
  max-width: 200px;
}

.data-node {
  min-width: 120px;
  border-color: #484f58;
}

.border-blue { border-color: #58a6ff; }
.border-red { border-color: #f85149; }
.border-yellow { border-color: #d29922; }
.border-green { border-color: #3fb950; }

.agents-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
  width: 100%;
}

.separate-grid {
  display: flex;
  justify-content: center;
}
</style>
