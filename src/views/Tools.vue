<template>
  <div>
    <div class="card">
      <h2>🔧 Tools</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        MCP servers and services that agents call via mcporter.
        Everything goes through mcporter — one consistent interface for all tools.
      </p>
    </div>

    <div class="card" v-for="tool in tools" :key="tool.name" :class="'border-' + tool.color">
      <div class="tool-header">
        <h3>{{ tool.icon }} {{ tool.name }}</h3>
        <span :class="['tag', tool.statusClass]">{{ tool.status }}</span>
      </div>
      <p class="tool-desc">{{ tool.desc }}</p>

      <div class="tool-details" v-if="tool.details?.length">
        <div class="tool-detail" v-for="d in tool.details" :key="d.label">
          <span class="detail-label">{{ d.label }}</span>
          <span class="detail-value">{{ d.value }}</span>
        </div>
      </div>

      <div class="tool-used-by" v-if="tool.usedBy">
        <span class="used-by-label">Used by:</span> {{ tool.usedBy }}
      </div>
    </div>

    
  </div>
</template>

<script setup>
const tools = [
  {
    icon: '🗄️',
    name: 'york-data',
    color: 'green',
    status: 'Built',
    statusClass: 'small',
    desc: 'Custom MCP server. Central data layer. Typed functions with validation, schema enforcement. Replaces direct Google Sheets calls for health data.',
    details: [
      { label: 'Type', value: 'MCP Server (Node.js, better-sqlite3)' },
      { label: 'Domains', value: 'Consumption (food/drink/supplement/cannabis), Daily Metrics, Workouts, Exercises' },
      { label: 'Tools', value: '19 tools across 3 domains + cross-domain queries' },
      { label: 'Repo', value: 'yorkbot/york-data' },
      { label: 'Access', value: 'mcporter call york-data.<tool>(...)' },
    ],
    usedBy: 'Wynn (health logging), Bede (analysis), Dagr (morning brief data), York (gate context).',
  },
  {
    icon: '🔧',
    name: 'york-tools',
    color: 'green',
    status: 'Built',
    statusClass: 'small',
    desc: 'MCP server for utility operations. No database — just functions. Cameras, weather, panel status, Discord avatar, and multi-model image generation.',
    details: [
      { label: 'Type', value: 'MCP Server (Node.js, no database)' },
      { label: 'Utility tools', value: 'camera_snapshot, weather_fetch, panel_update, avatar_set' },
      { label: 'Image tools', value: 'image_generate (multi-model, per-model param validation), image_models (catalog + capabilities)' },
      { label: 'Image models', value: 'Gemini Pro (best), Flash 3.1 (fast), Flash 2.5 (fastest), Grok Imagine (xAI). Extensible for SD, Flux via provider registry.' },
      { label: 'Image params', value: 'Core: prompt, model. Model-dependent: aspect_ratio, resolution, negative_prompt, cfg_scale, steps, seed, reference_image' },
      { label: 'Repo', value: 'yorkbot/york-tools (private)' },
      { label: 'Access', value: 'mcporter call york-tools.<tool>(...)' },
    ],
    usedBy: 'York (camera, panel, avatar, weather), Caedmon (image gen for D&D art), Dagr (weather), Hild (camera for chore state), Guthlac (grok-imagine-image). Any agent via global image-gen skill.',
  },
  {
    icon: '✅',
    name: 'google-tasks',
    color: 'green',
    status: 'Built',
    statusClass: 'small',
    desc: 'MCP server wrapping Google Tasks REST API. 12 tools, OAuth2 auth with auto-refresh. Chore tracking with built-in recurrence. Hild manages the intelligence layer on top.',
    details: [
      { label: 'Type', value: 'MCP Server (Node.js, pure API wrapper, no database)' },
      { label: 'Tools', value: '12 tools: list/create/delete task lists, list/get/create/update/complete/uncomplete/delete/move tasks, clear completed' },
      { label: 'Auth', value: 'OAuth2 with auto-refresh (google-oauth-token.json)' },
      { label: 'Repo', value: '~/york/google-tasks/' },
      { label: 'Access', value: 'mcporter call google-tasks.<tool>(...)' },
    ],
    usedBy: 'Hild (chore management, habit stacks), York (green light — reads task completion).',
  },
  {
    icon: '👁️',
    name: 'image-analysis',
    color: 'yellow',
    status: 'To Build',
    statusClass: 'medium',
    desc: 'Cheap vision model for image interpretation. Send a photo path + prompt, get a structured description. Pinned to the cheapest capable vision model (Gemini Flash/Nano) — not the agent\'s own model.',
    details: [
      { label: 'Type', value: 'MCP Server (LLM API call)' },
      { label: 'Input', value: 'Image path + prompt (e.g. "describe the state of this kitchen")' },
      { label: 'Output', value: 'Structured text description' },
      { label: 'Model', value: 'Gemini Flash or cheapest vision model available' },
    ],
    usedBy: 'York (green light camera checks), Hild (chore state from photos).',
  },
  {
    icon: '📹',
    name: 'camera-monitor',
    color: 'purple',
    status: 'To Build',
    statusClass: 'xl',
    desc: 'Non-LLM camera processing daemon. Frame-diff motion detection, state change tracking, structured output to york-data. Standalone process, not an MCP server — it runs continuously.',
    details: [
      { label: 'Type', value: 'Script / daemon (no LLM)' },
      { label: 'Detection', value: 'Frame-diff for motion, periodic snapshots for state' },
      { label: 'Output', value: 'Structured events to york-data' },
      { label: 'Open questions', value: 'Sampling frequency, event vs noise thresholds, night/IR handling' },
    ],
    usedBy: 'Bede (pattern analysis over time), Hild (chore state awareness), York (gate context).',
  },
  {
    icon: '📅',
    name: 'google-calendar',
    color: 'green',
    status: 'Built',
    statusClass: 'small',
    desc: 'Google Calendar MCP server. Already in mcporter. Lists events, creates events, manages calendars.',
    details: [
      { label: 'Type', value: 'MCP Server' },
      { label: 'Tools', value: '3 tools' },
      { label: 'Access', value: 'mcporter call google-calendar.<tool>(...)' },
    ],
    usedBy: 'Dagr (morning brief), York (schedule awareness).',
  },
  {
    icon: '📊',
    name: 'google-sheets',
    color: 'green',
    status: 'Built',
    statusClass: 'small',
    desc: 'Google Sheets MCP server. Health data migrated to york-data, but Sheets still used for home projects spreadsheet and ad-hoc data work.',
    details: [
      { label: 'Type', value: 'MCP Server' },
      { label: 'Tools', value: '17 tools' },
      { label: 'Access', value: 'mcporter call google-sheets.<tool>(...)' },
    ],
    usedBy: 'Hild (home projects spreadsheet), York (ad-hoc data work).',
  },
  {
    icon: '📝',
    name: 'google-docs',
    color: 'green',
    status: 'Built',
    statusClass: 'small',
    desc: 'Google Docs MCP server. Create, read, and edit documents. Used for collaborative editing with James.',
    details: [
      { label: 'Type', value: 'MCP Server' },
      { label: 'Tools', value: '6 tools' },
      { label: 'Access', value: 'mcporter call google-docs.<tool>(...)' },
    ],
    usedBy: 'York (ad-hoc collaborative docs with James).',
  },
  {
    icon: '🎲',
    name: 'dice-roller',
    color: 'green',
    status: 'Built',
    statusClass: 'small',
    desc: 'Dice rolling MCP server. Standard notation support.',
    details: [
      { label: 'Type', value: 'MCP Server' },
      { label: 'Tools', value: '1 tool' },
      { label: 'Access', value: 'mcporter call dice-roller.roll notation="2d6+5"' },
    ],
    usedBy: 'Caedmon (D&D).',
  },
]


</script>

<style scoped>
.tool-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.tool-header h3 {
  margin-bottom: 0;
}

.tool-desc {
  color: #c9d1d9;
  font-size: 0.9rem;
  line-height: 1.6;
  margin-bottom: 1rem;
}

.tool-details {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
  margin-bottom: 0.75rem;
}

.tool-detail {
  display: flex;
  gap: 0.75rem;
  padding: 0.4rem 0.65rem;
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 4px;
  font-size: 0.85rem;
}

.detail-label {
  color: #8b949e;
  min-width: 100px;
  font-weight: 500;
}

.detail-value {
  color: #c9d1d9;
}

.tool-used-by {
  font-size: 0.8rem;
  color: #8b949e;
  font-style: italic;
}

.used-by-label {
  font-weight: 600;
}

.border-green { border-color: #3fb950; }
.border-yellow { border-color: #d29922; }
.border-purple { border-color: #bc8cff; }
</style>
