<template>
  <div>
    <div class="card">
      <h2>🧠 Brainstorm</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Ideas for agents, skills, and capabilities. Some obvious, some speculative, some fun.
        Nothing here is committed. This is the "what if" page.
      </p>
    </div>

    <!-- Agent Ideas -->
    <div class="card">
      <h3>Agent Ideas Beyond Current Plan</h3>
      <div class="idea-grid">
        <div class="idea-card" v-for="idea in agentIdeas" :key="idea.name" :class="'priority-' + idea.priority">
          <div class="idea-header">
            <span class="idea-emoji">{{ idea.icon }}</span>
            <span class="idea-name">{{ idea.name }}</span>
            <span :class="['tag', idea.priorityClass]">{{ idea.priorityLabel }}</span>
          </div>
          <p class="idea-desc">{{ idea.desc }}</p>
          <p class="idea-why" v-if="idea.why">{{ idea.why }}</p>
        </div>
      </div>
    </div>

    <!-- Skill Ideas -->
    <div class="card">
      <h3>Skill Ideas for Existing Agents</h3>
      <div class="idea-grid">
        <div class="idea-card" v-for="skill in skillIdeas" :key="skill.name">
          <div class="idea-header">
            <span class="idea-name">{{ skill.agent }} → {{ skill.name }}</span>
          </div>
          <p class="idea-desc">{{ skill.desc }}</p>
        </div>
      </div>
    </div>

    <!-- Integration Ideas -->
    <div class="card">
      <h3>Integration Ideas</h3>
      <div class="idea-grid">
        <div class="idea-card" v-for="int in integrations" :key="int.name">
          <div class="idea-header">
            <span class="idea-emoji">{{ int.icon }}</span>
            <span class="idea-name">{{ int.name }}</span>
            <span :class="['tag', int.statusClass]">{{ int.status }}</span>
          </div>
          <p class="idea-desc">{{ int.desc }}</p>
        </div>
      </div>
    </div>



    <!-- Open Questions -->
    <div class="card">
      <h3>Things to Think About</h3>
      <div class="think-list">
        <div class="think-item" v-for="t in thinks" :key="t">💭 {{ t }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
const agentIdeas = [
  {
    icon: '🦎',
    name: 'Finance',
    priority: 'likely',
    priorityClass: 'medium',
    priorityLabel: 'Likely',
    desc: 'Budget tracking, YNAB integration, spending pattern analysis, monthly reviews. James already has the YNAB MCP built.',
    why: 'Natural fit with weekly review: weight + spending + consumption in one analysis. Cross-domain pattern recognition is the killer feature.',
  },

  {
    icon: '🐕',
    name: 'Car',
    priority: 'someday',
    priorityClass: 'script',
    priorityLabel: 'Someday',
    desc: 'Maintenance schedules, oil changes, registration, inspection reminders. James has a handwritten note to share.',
    why: 'Probably doesn\'t need its own agent — could be a york-data domain + calendar events. But if maintenance history gets deep, it might earn one.',
  },
  {
    icon: '🐈',
    name: 'Ginny',
    priority: 'fun',
    priorityClass: 'heartbeat',
    priorityLabel: 'Fun',
    desc: 'Vet appointment tracking, feeding schedule, litter change reminders, medication if needed. Named after the cat.',
    why: 'Probably overkill as a separate agent. But the name is perfect and it could live as a york-data domain that York manages.',
  },


]

const skillIdeas = [

  {
    agent: '🦌 Wynn',
    name: 'Meal pattern learning',
    desc: 'After 6 months of consumption data, suggest meals based on what James actually cooks vs what he orders. "You haven\'t made steak in 2 weeks and it\'s your best protein day."',
  },
  {
    agent: '🦌 Wynn',
    name: 'Restaurant nutrition lookup',
    desc: 'When James logs a restaurant meal, look up the actual menu nutrition instead of estimating. The #1 source of calorie corrections.',
  },
  {
    agent: '🐓 Dagr',
    name: 'Adaptive sections',
    desc: 'Learn which brief sections James reads vs skips. De-emphasize sections he consistently ignores. Promote sections he engages with.',
  },
  {
    agent: '🦫 Offa',
    name: 'Self-test',
    desc: 'After building a new skill or feature, generate test cases and verify them before declaring done.',
  },
]

const integrations = [
  {
    icon: '⌚',
    name: 'Bangle.js 2',
    status: 'Researched',
    statusClass: 'medium',
    desc: 'Open source smartwatch. Sleep tracking, step count, heart rate. $90, no subscription. Data feeds into Health agent.',
  },

  {
    icon: '💰',
    name: 'YNAB MCP',
    status: 'Exists',
    statusClass: 'large',
    desc: 'James already built this. Transaction categorization, budget tracking, spending analysis. Feeds into Finance agent.',
  },
  {
    icon: '📷',
    name: 'More cameras',
    status: 'Planned',
    statusClass: 'small',
    desc: 'Living room and den (smoking room) cameras for chore context. Same Tapo C103 + RTSP pattern. Frame-diff motion detection.',
  },

  {
    icon: '📱',
    name: 'SMS (Twilio)',
    status: 'Planned',
    statusClass: 'medium',
    desc: 'SMS nudges and alerts via Twilio. york-tools.twilio_send(). Late in build plan but architecturally accounted for.',
  },
]

const thinks = [
  'Session compaction UX: when agents compact, the Discord typing indicator goes blank and all models go silent. Feels like a network drop from the user\'s side. Need a way to signal "I\'m still here, just reorganizing" — maybe a brief status message or typing indicator during compaction.',
  'Dagr (Brief) will grow — alarm integration, schedule-aware wake times. Future plans confirmed.',
  'Finance ties into Health (spending on food), Home (project costs), and Social (gift budgets). How much cross-domain access does it need? Future scope.',
  'Should there be a dedicated Avatar/Identity agent? Daily avatar generation, banner changes, panel voice. Bookmarked for later.',
  'Agent conflict resolution (Health says "you need protein" but Chores says "kitchen is a mess") — will be a York skill.',
  'Conversational agents: James will build multiple over time. York is two-purpose (orchestrator + conversational) at first. Each agent gets its own voice.',
]
</script>

<style scoped>
.idea-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1rem;
}

.idea-card {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1rem;
}

.idea-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
  flex-wrap: wrap;
}

.idea-emoji { font-size: 1.2rem; }

.idea-name {
  color: #f0f6fc;
  font-weight: 600;
  font-size: 0.95rem;
}

.idea-desc {
  color: #c9d1d9;
  font-size: 0.85rem;
  line-height: 1.5;
  margin-bottom: 0.5rem;
}

.idea-why {
  color: #8b949e;
  font-size: 0.8rem;
  line-height: 1.5;
  font-style: italic;
  border-left: 2px solid #21262d;
  padding-left: 0.75rem;
}


.think-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.think-item {
  color: #c9d1d9;
  font-size: 0.85rem;
  padding: 0.75rem;
  background: #0d1117;
  border-radius: 4px;
  line-height: 1.5;
}

@media (max-width: 768px) {
  .idea-grid { grid-template-columns: 1fr; }
}
</style>
