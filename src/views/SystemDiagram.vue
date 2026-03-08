<template>
  <div>
    <div class="card">
      <h2>System Overview</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        One OpenClaw instance on RPi. York orchestrates everything. Specialized agents handle domains.
        Scripts replace LLMs where possible. Cron for scheduled work, heartbeat for contextual awareness.
        Model selection is by task complexity, not brand loyalty.
      </p>
      <p style="color: #c9d1d9; margin-bottom: 0; line-height: 1.6; font-size: 0.9rem; border-left: 3px solid #1f6feb; padding-left: 1rem;">
        This is about being <strong style="color: #58a6ff;">intentional</strong>, not cheap. Every agent gets the model
        that matches its task complexity. More focused contexts mean better performance AND lower cost.
        The system gets better and cheaper at the same time.
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
              <span class="tag large">Large</span>
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
              <span :class="['tag', agent.tierClass]">{{ agent.tierLabel }}</span>
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

      <div class="connector-down">▼ typed function calls</div>

      <!-- Service Layer -->
      <div class="layer service-layer">
        <div class="layer-label">SERVICE LAYER (MCP Server)</div>
        <div class="layer-content">
          <div class="node service-node">
            <div class="node-title">york-data</div>
            <div class="node-desc">
              Typed functions: log_food(), log_weight(), log_workout(), get_consumption(), get_trends().
              Validates inputs, enforces schema, handles all DB operations.
              LLMs never see raw DB queries, column indices, or JSON blobs.
            </div>
          </div>
        </div>
      </div>

      <div class="connector-down">▼ validated writes</div>

      <!-- Data Layer -->
      <div class="layer">
        <div class="layer-label">DATA & SERVICES</div>
        <div class="layer-content">
          <div class="node data-node" v-for="d in dataStores" :key="d.name">
            <div class="node-icon">{{ d.icon }}</div>
            <div class="node-title">{{ d.name }}</div>
            <div class="node-status" v-if="d.status">{{ d.status }}</div>
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
            <span class="tag medium">Medium</span>
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
    tierClass: 'small',
    tierLabel: 'Small',
    schedule: 'cron',
    scheduleLabel: 'Cron',
    desc: 'Food logging, calorie tracking, dashboard updates, weight trends',
    border: 'border-blue',
  },
  {
    id: 'chores',
    name: '🏠 Chores & Home',
    tierClass: 'small',
    tierLabel: 'Small',
    schedule: null,
    scheduleLabel: null,
    desc: 'Chore state, camera analysis, gate data prep. Spawned by York for gate checks.',
    border: 'border-blue',
  },
  {
    id: 'dnd',
    name: '🎲 D&D Campaign',
    tierClass: 'large',
    tierLabel: 'Large',
    schedule: 'cron',
    scheduleLabel: 'Cron + Direct Channel',
    desc: 'Wiki lookups, lore, worldbuilding. Large context needed for interconnected wiki. Direct channel routing for interactive sessions.',
    border: 'border-red',
  },
  {
    id: 'morning',
    name: '☀️ Morning Brief',
    tierClass: 'medium',
    tierLabel: 'Medium',
    schedule: 'cron',
    scheduleLabel: 'Cron 8AM',
    desc: 'Read cached data files, format brief, post to #general. No live data fetching.',
    border: 'border-yellow',
  },
  {
    id: 'research',
    name: '🔍 Research',
    tierClass: 'medium',
    tierLabel: 'Varies',
    schedule: null,
    scheduleLabel: null,
    desc: 'Web search, browsing, synthesis. Model scales with task complexity at spawn time.',
    border: 'border-yellow',
  },
  {
    id: 'builder',
    name: '🔨 Builder',
    tierClass: 'xl',
    tierLabel: 'XL',
    schedule: null,
    scheduleLabel: null,
    desc: 'Feature implementation, skill creation, debugging. The hard stuff. Spawned on-demand.',
    border: 'border-purple',
  },
  {
    id: 'analyst',
    name: '🔬 Improvement Analyst',
    tierClass: 'large',
    tierLabel: 'Large',
    schedule: 'cron',
    scheduleLabel: 'Cron overnight',
    desc: 'Reads friction journal, finds patterns across domains, diagnoses failures, spawns experiments.',
    border: 'border-red',
  },
  {
    id: 'weekly',
    name: '📊 Weekly Review',
    tierClass: 'large',
    tierLabel: 'Large',
    schedule: 'cron',
    scheduleLabel: 'Cron Sun 8PM',
    desc: 'Pull full week data, calculate trends, form opinions. Needs analytical depth.',
    border: 'border-red',
  },
]

const scripts = [
  { name: 'weather-cache', desc: 'curl wttr.in → memory file' },
  { name: 'calendar-cache', desc: 'mcporter → memory file' },
  { name: 'weight-log', desc: 'york-data.log_weight() → local DB' },
  { name: 'workout-log', desc: 'york-data.log_workout() → local DB' },
  { name: 'task-check', desc: 'Query pending tasks from york-data' },
  { name: 'camera-snapshot', desc: 'ffmpeg RTSP → /tmp/ images' },
  { name: 'avatar-set', desc: 'Discord API avatar upload' },
  { name: 'panel-update', desc: 'Write status.json for genmon' },
]

const dataStores = [
  { icon: '🗄️', name: 'Local DB', status: 'New — replaces Sheets for owned data. DB engine TBD.' },
  { icon: '📅', name: 'Google Calendar', status: 'Stays — external service, read-only via MCP' },
  { icon: '📊', name: 'Google Sheets', status: 'Demoted — ad-hoc tool only. Data migrates to local DB.' },
  { icon: '📁', name: 'Memory Files', status: 'Stays — daily context, agent communication' },
  { icon: '📖', name: 'D&D Wiki', status: 'Stays — markdown files' },
  { icon: '📷', name: 'Cameras (RTSP)', status: 'Stays — snapshots via ffmpeg' },
  { icon: '🖥️', name: 'Panel (Genmon)', status: 'Stays — status.json on disk' },
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

.service-layer {
  border-color: #d29922;
  background: #1a1815;
}

.service-node {
  border-color: #d29922;
  min-width: 400px;
  max-width: 600px;
}

.data-node {
  min-width: 140px;
  border-color: #484f58;
}

.node-status {
  font-size: 0.7rem;
  color: #d29922;
  margin-top: 0.25rem;
  font-style: italic;
}

.border-blue { border-color: #58a6ff; }
.border-red { border-color: #f85149; }
.border-yellow { border-color: #d29922; }
.border-green { border-color: #3fb950; }
.border-purple { border-color: #bc8cff; }

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
