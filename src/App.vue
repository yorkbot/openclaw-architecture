<template>
  <div class="app">
    <header>
      <h1>🦞 OpenClaw Architecture 🦝</h1>
      <p class="subtitle">Proposed multi-agent setup for James's RPi</p>
    </header>

    <div class="views">
      <button
        v-for="v in views"
        :key="v.id"
        :class="{ active: currentView === v.id }"
        @click="currentView = v.id"
      >{{ v.label }}</button>
    </div>

    <SystemDiagram v-if="currentView === 'system'" />
    <AgentDetail v-if="currentView === 'agents'" />
    <Tools v-if="currentView === 'tools'" />
    <DiscordLayout v-if="currentView === 'comms'" />
    <CronSchedule v-if="currentView === 'crons'" />


    <Migration v-if="currentView === 'migration'" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import SystemDiagram from './views/SystemDiagram.vue'
import AgentDetail from './views/AgentDetail.vue'
import DiscordLayout from './views/DiscordLayout.vue'

import Tools from './views/Tools.vue'

import CronSchedule from './views/CronSchedule.vue'
import Migration from './views/Migration.vue'

const currentView = ref('agents')

const views = [
  { id: 'system', label: 'System Overview' },
  { id: 'agents', label: 'Agent Details' },
  { id: 'tools', label: 'Tools' },
  { id: 'comms', label: 'Communication Channels' },
  { id: 'crons', label: 'Cron Schedule' },

  { id: 'migration', label: 'Migration' },

]
</script>

<style>
* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  background: #0d1117;
  color: #c9d1d9;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

.app {
  max-width: 1400px;
  margin: 0 auto;
  padding: 2rem;
}

header {
  text-align: center;
  margin-bottom: 2rem;
}

header h1 {
  font-size: 2rem;
  color: #f0f6fc;
}

.subtitle {
  color: #8b949e;
  margin-top: 0.5rem;
}

.views {
  display: flex;
  gap: 0.5rem;
  justify-content: center;
  margin-bottom: 2rem;
  flex-wrap: wrap;
}

.views button {
  background: #161b22;
  border: 1px solid #30363d;
  color: #c9d1d9;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.2s;
}

.views button:hover {
  border-color: #58a6ff;
  color: #58a6ff;
}

.views button.active {
  background: #1f6feb;
  border-color: #1f6feb;
  color: white;
}

.card {
  background: #161b22;
  border: 1px solid #30363d;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1rem;
}

.card h2 {
  color: #f0f6fc;
  margin-bottom: 1rem;
  font-size: 1.2rem;
}

.card h3 {
  color: #58a6ff;
  margin-bottom: 0.5rem;
  font-size: 1rem;
}

.tag {
  display: inline-block;
  padding: 0.15rem 0.5rem;
  border-radius: 12px;
  font-size: 0.75rem;
  font-weight: 600;
  margin-right: 0.25rem;
}

.tag.trivial { background: #1a3a1a; color: #3fb950; }
.tag.small { background: #1a2a3a; color: #58a6ff; }
.tag.medium { background: #3a2a1a; color: #d29922; }
.tag.large { background: #3a1a1a; color: #f85149; }
.tag.xl { background: #2a1a3a; color: #bc8cff; }
.tag.script { background: #1a2a1a; color: #56d364; border: 1px dashed #56d364; }
.tag.opus { background: #3a1a1a; color: #f85149; }
.tag.sonnet { background: #1a2a3a; color: #58a6ff; }
.tag.gpt { background: #1a3a2a; color: #3fb950; }
.tag.grok { background: #2a2a1a; color: #d29922; }
.tag.cron { background: #1a1a2a; color: #8b8bff; }
.tag.heartbeat { background: #2a1a2a; color: #bc8cff; }
</style>
