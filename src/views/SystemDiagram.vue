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
            <div class="node-icon">🦊</div>
            <div class="node-title">York</div>
            <div class="node-detail">
              <span class="tag large">Opus 4.6</span>
              <span class="tag heartbeat">Heartbeat 30m</span>
            </div>
            <div class="node-desc">
              Conversation, judgment calls, routing, accountability.
              Reads york-data + Google Tasks directly. Spawns specialists.
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
        <div class="layer-label">SERVICE LAYER (MCP Servers)</div>
        <div class="layer-content" style="gap: 1rem;">
          <div class="node service-node">
            <div class="node-title">york-data</div>
            <div class="node-desc">
              Data layer. Typed functions: log_food(), log_daily(), get_consumption(), get_trends().
              Validates inputs, enforces schema, handles all DB operations.
            </div>
          </div>
          <div class="node service-node" style="border-color: #3fb950;">
            <div class="node-title">york-tools</div>
            <div class="node-desc">
              Utility tools. camera_snapshot(), weather_fetch(),
              panel_update(), avatar_set(). Stateless functions as MCP calls.
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

    <!-- Skills Discovery -->
    <div class="card" style="margin-top: 2rem; border-color: #bc8cff;">
      <h2>🧠 Skills Discovery</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        OpenClaw discovers skills via YAML frontmatter in <code>SKILL.md</code> files.
        Without frontmatter, a skill exists on disk but is invisible to the agent.
      </p>

      <div class="infra-section">
        <h3>Required Frontmatter</h3>
        <div class="code-block">
          <code>---<br>name: my-skill<br>description: What this skill does and when to use it.<br>---</code>
        </div>
        <p class="infra-note">Every SKILL.md must start with this. The <code>name</code> and <code>description</code> are indexed into the <code>&lt;available_skills&gt;</code> block in the system prompt. No frontmatter = no discovery.</p>
      </div>

      <div class="infra-section">
        <h3>Load Locations (precedence high → low)</h3>
        <table class="infra-table">
          <thead><tr><th>Location</th><th>Scope</th><th>Purpose</th></tr></thead>
          <tbody>
            <tr>
              <td><code>&lt;workspace&gt;/skills/</code></td>
              <td>Per-agent only</td>
              <td>Agent-specific skills. Wiglaf's vault-schema, Wynn's log-consumption, etc.</td>
            </tr>
            <tr>
              <td><code>~/.openclaw/skills/</code></td>
              <td>All agents</td>
              <td>Global shared skills: git-workflow, image-gen, memory-audit, memory-management.</td>
            </tr>
            <tr>
              <td>Bundled (npm package)</td>
              <td>All agents</td>
              <td>Ships with OpenClaw. weather, github, clawhub, etc.</td>
            </tr>
          </tbody>
        </table>
        <p class="infra-note">If the same skill name exists in multiple locations, workspace wins. No cross-contamination — workspace skills only load for their agent.</p>
      </div>
    </div>

    <!-- Shared Infrastructure -->
    <div class="card" style="margin-top: 2rem; border-color: #3fb950;">
      <h2>📁 Shared Infrastructure</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Cross-agent paths for coordination. Each agent has private memory in
        <code>workspace-&lt;agent&gt;/memory/</code> — these shared dirs are NOT agent memory.
      </p>

      <div class="infra-section">
        <h3>📋 ~/.openclaw/shared-cache/</h3>
        <p class="infra-desc">Cron output staging. Overnight scripts write sections here; Dagr and Wiglaf read at their scheduled times.</p>
        <table class="infra-table">
          <thead><tr><th>Detail</th><th>Value</th></tr></thead>
          <tbody>
            <tr><td>Format</td><td><code>YYYY-MM-DD.md</code> with <code>##</code> headed sections (Weather, Calendar Context, Nutrition, D&D, etc.)</td></tr>
            <tr><td>Writers</td><td>weather-cache.sh, calendar-cache.sh, nutrition script (Wynn), consumption-gap script (Wynn), D&D questions (Caedmon)</td></tr>
            <tr><td>Readers</td><td>Dagr (morning brief at 8 AM), Wiglaf (morning-prep at 7 AM, now-management every 15 min)</td></tr>
          </tbody>
        </table>
      </div>

      <div class="infra-section">
        <h3>⚙️ ~/.openclaw/shared-scripts/</h3>
        <p class="infra-desc">Utility scripts. Cron-triggered, no LLM. Pure bash + python.</p>
        <table class="infra-table">
          <thead><tr><th>Detail</th><th>Value</th></tr></thead>
          <tbody>
            <tr><td>Scripts</td><td>weather-cache.sh, calendar-cache.sh</td></tr>
            <tr><td>Output</td><td>Writes sections to ~/.openclaw/shared-cache/YYYY-MM-DD.md</td></tr>
          </tbody>
        </table>
      </div>

      <div class="infra-section">
        <h3>🧠 ~/.openclaw/skills/</h3>
        <p class="infra-desc">Global skills shared across all agents via OpenClaw's skill discovery.</p>
        <table class="infra-table">
          <thead><tr><th>Skill</th><th>Status</th><th>Purpose</th></tr></thead>
          <tbody>
            <tr><td>git-workflow</td><td><span class="status-tag status-on">Live</span></td><td>Branch strategy, commit messages, PR workflow. yorkbot repos push to main; dohertyj08 repos branch + PR.</td></tr>
            <tr><td>image-gen</td><td><span class="status-tag status-on">Live</span></td><td>Multi-model image generation via york-tools. Model selection, prompting by family.</td></tr>
            <tr><td>memory-audit</td><td><span class="status-tag status-on">Live</span></td><td>Nightly Sonnet sub-agent cron. Prune stale entries, flag conflicts, verify accuracy.</td></tr>
            <tr><td>memory-management</td><td><span class="status-tag status-on">Live</span></td><td>How all agents read/write MEMORY.md and memory/ files.</td></tr>
            <tr><td>image-analysis</td><td><span class="status-tag status-todo">TODO</span></td><td>Calling the image-analysis tool. Prompt framing for room assessment, document reading, visual inspection.</td></tr>
            <tr><td>york-data-conventions</td><td><span class="status-tag status-todo">TODO</span></td><td>How to call york-data MCP functions. Parameter naming, error handling, upsert patterns, date formats.</td></tr>
            <tr><td>flag-for-bede</td><td><span class="status-tag status-todo">TODO</span></td><td>Any agent can flag something unusual for Bede. Structured flag to york-data for Bede's next collection run.</td></tr>
          </tbody>
        </table>
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
  { name: '🦊york', routes: '→ York 🦊' },
  { name: '🐓dagr', routes: '→ Dagr 🐓 (direct)' },
  { name: '🐉caedmon', routes: '→ Caedmon 🐉 (direct)' },
  { name: '🦫offa', routes: '→ Offa 🦫 (direct)' },
  { name: '🦌wynn', routes: '→ Wynn 🦌 (direct)' },
  { name: '🐻wiglaf', routes: '→ Wiglaf 🐻 (direct)' },
  { name: '🦡hild', routes: '→ Hild 🦡 (direct)' },
  { name: '🐺guthlac', routes: '→ Guthlac 🐺 (direct)' },
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
    desc: 'Health & fitness. Logging (consumption, weight, workouts) and coaching.',
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
  {
    id: 'wiglaf',
    name: '🐻 Wiglaf',
    tierClass: 'large',
    tierLabel: 'Opus 4.6',
    schedule: null,
    scheduleLabel: null,
    desc: 'Private work agent. Prioritization, note processing, meeting prep, work context. Vault-watcher driven, on-demand. Excluded from Bede analysis.',
    border: 'border-yellow',
  },
  {
    id: 'guthlac',
    name: '🐺 Guthlac',
    tierClass: 'grok',
    tierLabel: 'Grok 4.1',
    schedule: null,
    scheduleLabel: null,
    desc: 'Private personal agent. Conversation, venting, thinking. Personality shaped over time. Excluded from Bede analysis.',
    border: 'border-green',
  },
]

const scripts = [
  // The only true daemon
  { name: 'camera-monitor', desc: 'Frame-diff motion detection daemon → state events to york-data. Long-running process, not an MCP tool.' },
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

.infra-section {
  margin-bottom: 1.5rem;
}

.infra-section:last-child {
  margin-bottom: 0;
}

.infra-section h3 {
  font-size: 0.95rem;
  color: #f0f6fc;
  margin-bottom: 0.5rem;
}

.infra-desc {
  color: #c9d1d9;
  font-size: 0.85rem;
  margin-bottom: 0.75rem;
}

.infra-note {
  color: #8b949e;
  font-size: 0.8rem;
  margin-top: 0.5rem;
  line-height: 1.5;
}

.infra-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.85rem;
  margin-top: 0.5rem;
}

.infra-table th {
  text-align: left;
  color: #8b949e;
  padding: 0.4rem 0.75rem;
  border-bottom: 1px solid #30363d;
  font-weight: 600;
}

.infra-table td {
  padding: 0.4rem 0.75rem;
  border-bottom: 1px solid #21262d;
  color: #c9d1d9;
}

.code-block {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 0.75rem 1rem;
  margin: 0.5rem 0;
}

.code-block code {
  color: #79c0ff;
  font-size: 0.85rem;
  line-height: 1.6;
}

.status-tag {
  font-size: 0.7rem;
  padding: 0.1rem 0.4rem;
  border-radius: 10px;
  font-weight: 600;
  white-space: nowrap;
}

.status-on {
  background: rgba(63, 185, 80, 0.15);
  color: #3fb950;
}

.status-todo {
  background: rgba(139, 148, 158, 0.15);
  color: #8b949e;
}
</style>
