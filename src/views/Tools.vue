<template>
  <div>
    <div class="card">
      <h2>🔧 Tools</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Things that need to be built. Services, scripts, and infrastructure that agents depend on.
        Not agents themselves — these are the tools agents call.
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
    color: 'yellow',
    status: 'To Build',
    statusClass: 'medium',
    desc: 'Custom MCP server. The central data layer between agents and storage. Typed functions with validation, schema enforcement, and no raw DB access for LLMs. Replaces direct Google Sheets calls.',
    details: [
      { label: 'Type', value: 'MCP Server (Node.js)' },
      { label: 'Domains', value: 'Consumption, Daily Metrics, Workouts, Cannabis, Chores & Home, Observations' },
      { label: 'DB Engine', value: 'TBD — likely Node built-in SQLite (node:sqlite)' },
      { label: 'Access', value: 'All agents via mcporter call york-data.<function>(...)' },
    ],
    usedBy: 'Every agent. This is the backbone.',
  },
  {
    icon: '🔧',
    name: 'york-tools',
    color: 'green',
    status: 'To Build',
    statusClass: 'medium',
    desc: 'MCP server bundling all utility operations. Replaces standalone bash scripts with typed, validated tool calls. Same calling pattern as york-data: mcporter call york-tools.<function>(...)',
    details: [
      { label: 'Type', value: 'MCP Server (Node.js)' },
      { label: 'Functions', value: 'camera_snapshot, weather_fetch, calendar_fetch, panel_update, avatar_set, twilio_send, wiki_search, transcript_export' },
      { label: 'Why MCP', value: 'Typed params, validation, consistent calling pattern. Agents call tools the same way regardless of what\'s under the hood.' },
    ],
    usedBy: 'All agents. Replaces the scripts layer.',
  },
  {
    icon: '📷',
    name: 'camera-snapshot',
    color: 'green',
    status: 'Exists',
    statusClass: 'small',
    desc: 'Simple ffmpeg script. Grabs a single frame from an RTSP camera feed and writes it to /tmp/. That\'s it. No analysis, no state, no LLM.',
    details: [
      { label: 'Type', value: 'Bash script' },
      { label: 'How', value: 'ffmpeg -i rtsp://camera/stream -frames:v 1 /tmp/snapshot.jpg' },
      { label: 'Cameras', value: 'Kitchen (Tapo C103). More planned.' },
    ],
    usedBy: 'Hild (on-demand snapshots for chore/gate context), camera-monitor (as a building block).',
  },
  {
    icon: '📹',
    name: 'camera-monitor',
    color: 'purple',
    status: 'To Build',
    statusClass: 'xl',
    desc: 'Non-LLM camera processing pipeline. Frame-diff motion detection, state change tracking, structured output to york-data. The LLM only gets involved when an agent needs to interpret what\'s in an image — the monitoring itself is script-tier.',
    details: [
      { label: 'Type', value: 'Script / daemon (no LLM)' },
      { label: 'Detection', value: 'Frame-diff for motion, periodic snapshots for state' },
      { label: 'Output', value: 'Structured events to york-data: "kitchen_state_changed", "motion_detected"' },
      { label: 'Open questions', value: 'How often to sample? What triggers an event vs noise? How to handle night/IR mode?' },
    ],
    usedBy: 'Bede (pattern analysis over time), Hild (chore state awareness), York (gate context).',
  },
  {
    icon: '✅',
    name: 'google-tasks',
    color: 'yellow',
    status: 'To Build',
    statusClass: 'medium',
    desc: 'MCP server or york-data integration for Google Tasks API. Chore tracking with built-in recurrence for standard frequencies (daily/weekly/monthly). Hild manages the intelligence layer on top: seasonal adjustments, weather-dependent task creation. Completion data mirrors to york-data for Bede.',
    details: [
      { label: 'Type', value: 'MCP Server or york-data extension' },
      { label: 'API', value: 'Google Tasks API (requires permission extension in Google Console)' },
      { label: 'Recurrence', value: 'Daily, weekly, monthly handled by Google. Seasonal/weather by Hild.' },
      { label: 'Visibility', value: 'James sees chores in Google Calendar UI for free' },
    ],
    usedBy: 'Hild (chore management), Bede (completion pattern analysis via york-data mirror).',
  },
  {
    icon: '👁️',
    name: 'image-analysis',
    color: 'yellow',
    status: 'To Build',
    statusClass: 'medium',
    desc: 'LLM-as-a-tool for image interpretation. Send a photo, get a structured description back. Not an agent skill — a tool call that happens to hit an LLM API. Could use Gemini Flash, Nano, or whatever\'s cheapest for "describe this kitchen."',
    details: [
      { label: 'Type', value: 'Tool (LLM API call, not an agent)' },
      { label: 'Input', value: 'Image path + prompt (e.g. "describe the state of this kitchen")' },
      { label: 'Output', value: 'Structured text description' },
      { label: 'Model', value: 'TBD — Gemini Flash/Nano, or cheapest vision model available' },
    ],
    usedBy: 'York (cannabis gate camera checks), potentially Bede (pattern analysis over time).',
  },
  {
    icon: '🎨',
    name: 'image-gen',
    color: 'green',
    status: 'Exists',
    statusClass: 'small',
    desc: 'Image generation via Gemini Nano Banana Pro. Takes a text prompt, returns a generated image. Currently used for D&D character art and fantasy scenes. The calling agent crafts the prompt — the tool just generates.',
    details: [
      { label: 'Type', value: 'Tool (LLM API call)' },
      { label: 'Model', value: 'Gemini Nano Banana Pro (gemini-2.0-flash-exp)' },
      { label: 'Input', value: 'Text prompt describing the image' },
      { label: 'Output', value: 'Generated image file' },
    ],
    usedBy: 'Caedmon (D&D art — characters, locations, items, scenes). Potentially other agents for visual content.',
  },
  {
    icon: '🌤️',
    name: 'weather-cache',
    color: 'green',
    status: 'Exists',
    statusClass: 'script',
    desc: 'Curl wttr.in or Open-Meteo, write result to a memory file. Runs on cron before morning brief.',
    details: [
      { label: 'Type', value: 'Bash script / cron job' },
    ],
    usedBy: 'Brief (morning brief weather section).',
  },
  {
    icon: '📅',
    name: 'calendar-cache',
    color: 'green',
    status: 'Exists',
    statusClass: 'script',
    desc: 'mcporter call to Google Calendar, write events to a memory file. Runs on cron before morning brief.',
    details: [
      { label: 'Type', value: 'mcporter call / cron job' },
    ],
    usedBy: 'Brief (morning brief calendar section), York (schedule awareness).',
  },
  {
    icon: '🖼️',
    name: 'avatar-set',
    color: 'green',
    status: 'Exists',
    statusClass: 'script',
    desc: 'Discord API call to update bot avatar. Simple curl script.',
    details: [
      { label: 'Type', value: 'Bash script' },
    ],
    usedBy: 'York (identity management).',
  },
  {
    icon: '🖥️',
    name: 'panel-update',
    color: 'green',
    status: 'Exists',
    statusClass: 'script',
    desc: 'Write status.json for the XFCE genmon panel widget. York\'s face on the desktop.',
    details: [
      { label: 'Type', value: 'Bash script' },
    ],
    usedBy: 'York (panel presence).',
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
