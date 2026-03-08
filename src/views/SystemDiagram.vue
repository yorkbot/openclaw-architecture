<template>
  <div>
    <div class="card">
      <h2>System Overview</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        One OpenClaw instance on yorks-vm. York orchestrates everything. Specialized agents handle domains.
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
          <div class="node sms-node">
            <div class="node-icon">📱</div>
            <div class="node-title">SMS (Twilio)</div>
            <div class="node-detail">Nudges & alerts</div>
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
              <span class="tag large">Opus 4.6</span>
              <span class="tag heartbeat">Heartbeat 30m</span>
            </div>
            <div class="node-desc">
              Conversation, judgment calls, routing, cannabis gate, accountability.
              Reads york-data + Google Tasks directly for gate decisions. Spawns specialists.
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

      <div class="connector-down">▼ read/write · 🦉 Bede reads transcripts from ALL agents + collects from york-data</div>

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
  { name: 'dnd', routes: '→ Caedmon (direct)' },
  { name: 'reviews', routes: '→ York (proposals from Bede/Offa)' },
]

const agents = [
  {
    id: 'bede',
    name: '🦉 Bede',
    tierClass: 'large',
    tierLabel: 'Opus 4.6',
    schedule: 'cron',
    scheduleLabel: 'Cron (varied)',
    desc: 'Analyst. Collects data from all agents, analyzes patterns, suggests improvements, measures outcomes.',
    border: 'border-yellow',
  },
  {
    id: 'offa',
    name: '🦫 Offa',
    tierClass: 'large',
    tierLabel: 'Opus 4.6',
    schedule: null,
    scheduleLabel: null,
    desc: 'Builder. Reads suggestions from york-data, implements new skills and skill edits.',
    border: 'border-yellow',
  },
  {
    id: 'wynn',
    name: '🦌 Wynn',
    tierClass: 'sonnet',
    tierLabel: 'Sonnet',
    schedule: null,
    scheduleLabel: null,
    desc: 'Health & fitness. Logging (consumption, weight, workouts, cannabis) and coaching.',
    border: 'border-blue',
  },
  {
    id: 'hild',
    name: '🦡 Hild',
    tierClass: 'sonnet',
    tierLabel: 'Sonnet',
    schedule: null,
    scheduleLabel: null,
    desc: 'Home management. Chore tracking via Google Tasks, smart scheduling.',
    border: 'border-blue',
  },
  {
    id: 'caedmon',
    name: '🐉 Caedmon',
    tierClass: 'large',
    tierLabel: 'Opus 4.6',
    schedule: 'cron',
    scheduleLabel: 'Cron overnight',
    desc: 'D&D campaign. Wiki, lore, session prep/recap, art generation, overnight research.',
    border: 'border-yellow',
  },
  {
    id: 'dagr',
    name: '🐓 Dagr',
    tierClass: 'sonnet',
    tierLabel: 'Sonnet',
    schedule: 'cron',
    scheduleLabel: 'Cron 8AM',
    desc: 'Morning brief. Compile cached data, editorialize, post.',
    border: 'border-blue',
  },
]

const scripts = [
  // Caching (Dagr morning brief inputs)
  { name: 'weather-cache', desc: 'curl wttr.in / Open-Meteo → memory file' },
  { name: 'calendar-cache', desc: 'mcporter google-calendar → memory file' },

  // Camera pipeline
  { name: 'camera-snapshot', desc: 'ffmpeg RTSP → /tmp/ image. On-demand, single frame.' },
  { name: 'camera-monitor', desc: 'Frame-diff motion detection daemon → state events to york-data' },

  // Identity & presence
  { name: 'panel-update', desc: 'Write status.json for XFCE genmon widget' },
  { name: 'avatar-set', desc: 'Discord API avatar/banner upload' },

  // Communication
  { name: 'twilio-send', desc: 'Send SMS via Twilio API. Nudges, alerts, urgent notifications.' },

  // D&D wiki
  { name: 'wiki-search', desc: 'grep/ripgrep across D&D wiki markdown files. Fast local search.' },
  { name: 'wiki-pr', desc: 'Branch, commit, push, open PR on bunglers repo. Used by Caedmon for wiki edits.' },

  // Data caching for Bede
  { name: 'transcript-export', desc: 'Extract session transcripts from JSONL into readable format for Bede analysis.' },
]

const dataStores = [
  { icon: '🗄️', name: 'Local DB (york-data)', status: 'New — MCP server, typed functions, replaces Sheets' },
  { icon: '📅', name: 'Google Calendar', status: 'Read + write via MCP' },
  { icon: '✅', name: 'Google Tasks', status: 'New — chore tracking for Hild' },
  { icon: '📁', name: 'Memory Files', status: 'Daily context, agent communication, Bede reflections' },
  { icon: '📖', name: 'D&D Wiki', status: 'Git repo — markdown files, PRs' },
  { icon: '📷', name: 'Cameras (RTSP)', status: 'Tapo C103 + planned expansions' },
  { icon: '🖥️', name: 'Panel (Genmon)', status: 'status.json on disk' },
  { icon: '📱', name: 'Twilio SMS', status: 'Planned — nudges and alerts' },
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

.sms-node {
  border-color: #f22f46;
  min-width: 160px;
  border-style: dashed;
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
