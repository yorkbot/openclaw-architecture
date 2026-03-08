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

    <!-- Animal Emoji Reference -->
    <div class="card">
      <h3>Agent Roster (Current + Ideas)</h3>
      <p style="color: #8b949e; margin-bottom: 1rem; font-size: 0.85rem;">
        Every agent gets a one-word name and an animal emoji. Names are descriptive to save tokens.
      </p>
      <div class="roster">
        <div class="roster-item" v-for="r in roster" :key="r.name" :class="r.status">
          <span class="roster-emoji">{{ r.icon }}</span>
          <span class="roster-name">{{ r.name }}</span>
          <span class="roster-role">{{ r.role }}</span>
          <span :class="['tag', r.tagClass]">{{ r.tagLabel }}</span>
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
  {
    icon: '🦑',
    name: 'Scraper',
    priority: 'someday',
    priorityClass: 'script',
    priorityLabel: 'Someday',
    desc: 'Web scraping, price monitoring, deal alerts. Watch for sales on specific items, track prices over time.',
    why: 'Could save money on recurring purchases. "Those dumbbells you wanted are 30% off." Script-tier for the actual scraping, Small for analysis.',
  },
  {
    icon: '🐺',
    name: 'Coach',
    priority: 'likely',
    priorityClass: 'medium',
    priorityLabel: 'Likely',
    desc: 'Workout programming, exercise suggestions, form tips. Goes beyond logging into actual guidance. Adapts to equipment availability and schedule.',
    why: 'Currently York suggests exercises inline. A dedicated agent could maintain a proper program, track progressive overload, and know the exercise library deeply.',
  },
  {
    icon: '🦜',
    name: 'Social',
    priority: 'someday',
    priorityClass: 'script',
    priorityLabel: 'Someday',
    desc: 'Gift tracking, birthday reminders, Audrey visit prep automation, social calendar awareness.',
    why: 'Low-key useful. "Audrey\'s birthday is in 2 weeks and you haven\'t gotten anything yet." Could be york-data + calendar, doesn\'t need its own agent.',
  },
  {
    icon: '🐢',
    name: 'Maintain',
    priority: 'likely',
    priorityClass: 'medium',
    priorityLabel: 'Likely',
    desc: 'Home maintenance calendar for a 1920s house. Seasonal tasks, project tracking, contractor contacts, cost history.',
    why: 'That house always has something. A dedicated agent could build a maintenance timeline and catch seasonal items before they become emergencies.',
  },
  {
    icon: '🐋',
    name: 'Archive',
    priority: 'someday',
    priorityClass: 'script',
    priorityLabel: 'Someday',
    desc: 'Long-term memory management. Compress old daily memory files into searchable summaries. Manage the lifecycle of context as it ages.',
    why: 'After 6 months of daily memory files, search gets noisy. An archiver could create monthly/quarterly summaries and manage retention.',
  },
]

const skillIdeas = [
  {
    agent: '🦝 York',
    name: 'Mood detection',
    desc: 'Read tone in James\'s messages and adjust response style. Not sentiment analysis — more like "is this a quick question or does he want to talk."',
  },
  {
    agent: '🦝 York',
    name: 'Proactive calendar awareness',
    desc: '"You have a meeting in 45 minutes, might want to wrap up that D&D conversation." Calendar-driven nudges via heartbeat.',
  },

  {
    agent: '🐝 Health',
    name: 'Meal pattern learning',
    desc: 'After 6 months of consumption data, suggest meals based on what James actually cooks vs what he orders. "You haven\'t made steak in 2 weeks and it\'s your best protein day."',
  },
  {
    agent: '🐝 Health',
    name: 'Restaurant nutrition lookup',
    desc: 'When James logs a restaurant meal, look up the actual menu nutrition instead of estimating. The #1 source of calorie corrections.',
  },
  {
    agent: '🐉 Lore',
    name: 'Session replay',
    desc: 'After a D&D session, generate a narrative recap from session notes. Could auto-update wiki with new developments.',
  },
  {
    agent: '🐓 Brief',
    name: 'Adaptive sections',
    desc: 'Learn which brief sections James reads vs skips. De-emphasize sections he consistently ignores. Promote sections he engages with.',
  },
  {
    agent: '🦫 Builder',
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
    icon: '⚖️',
    name: 'DIY Smart Scale',
    status: 'Planned',
    statusClass: 'small',
    desc: 'ESP32 + HX711 + load cell. Auto-log weight to york-data. ~$15 in parts. Eliminates manual weight logging entirely.',
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
    icon: '🎮',
    name: 'Game state awareness',
    status: 'Wild idea',
    statusClass: 'script',
    desc: 'Know when James is gaming (PZ, sim racing) and suppress non-urgent nudges. Could use process detection or focused window info.',
  },
  {
    icon: '📱',
    name: 'Phone presence',
    status: 'Deprioritized',
    statusClass: 'script',
    desc: 'SMS/push notifications for truly urgent things. James says Discord reach is fine for now.',
  },
]

const roster = [
  { icon: '🦉', name: 'Analyst', role: 'Self-improvement, pattern recognition, experiment design', tagClass: 'draft-tag', tagLabel: 'Draft', status: 'active' },
  { icon: '🦝', name: 'York', role: 'Orchestrator, conversation, judgment, cannabis gate', tagClass: 'suggested-tag', tagLabel: 'Suggested', status: 'active' },
  { icon: '🐝', name: 'Health', role: 'Nutrition logging, weight tracking, calorie analysis', tagClass: 'suggested-tag', tagLabel: 'Suggested', status: 'active' },
  { icon: '🐜', name: 'Chores', role: 'Home tasks, camera analysis, gate data prep', tagClass: 'suggested-tag', tagLabel: 'Suggested', status: 'active' },
  { icon: '🐉', name: 'Lore', role: 'D&D wiki, worldbuilding, session prep', tagClass: 'suggested-tag', tagLabel: 'Suggested', status: 'active' },
  { icon: '🐓', name: 'Brief', role: 'Morning brief compilation and editorial', tagClass: 'suggested-tag', tagLabel: 'Suggested', status: 'active' },
  { icon: '🦊', name: 'Research', role: 'Web search, comparison, synthesis', tagClass: 'suggested-tag', tagLabel: 'Suggested', status: 'active' },
  { icon: '🦫', name: 'Builder', role: 'Implementation, debugging, skill creation', tagClass: 'suggested-tag', tagLabel: 'Suggested', status: 'active' },
  { icon: '🦅', name: 'Review', role: 'Weekly accountability, trend analysis', tagClass: 'suggested-tag', tagLabel: 'Suggested', status: 'active' },

  { icon: '🦎', name: 'Finance', role: 'Budget, YNAB, spending patterns', tagClass: 'idea-tag', tagLabel: 'Idea', status: 'idea' },
  { icon: '🐺', name: 'Coach', role: 'Workout programming, exercise guidance', tagClass: 'idea-tag', tagLabel: 'Idea', status: 'idea' },
  { icon: '🐢', name: 'Maintain', role: 'Home maintenance calendar, seasonal tasks', tagClass: 'idea-tag', tagLabel: 'Idea', status: 'idea' },
  { icon: '🐋', name: 'Archive', role: 'Long-term memory compression and management', tagClass: 'idea-tag', tagLabel: 'Idea', status: 'idea' },
]

const thinks = [
  'Should Coach and Health be the same agent? They share data but have different complexity needs (logging is Small, programming is Medium+).',
  'Is Brief really its own agent or just a cron skill that York runs? It doesn\'t have persistent state or interactive sessions.',
  'Review and Analyst both do pattern analysis. Should Review be a skill of Analyst, or do they have different enough concerns?',
  'Finance ties into Health (spending on food), Home (project costs), and Social (gift budgets). How much cross-domain access does it need?',

  'Should agents be able to flag things for the Analyst directly? "Hey, something weird happened in this session." Or is transcript-only observation cleaner?',
  'How much personality should non-York agents have? Should Lore sound different from Health? Or should York\'s voice be the only personality?',
  'Maintain (home maintenance) on a 1920s house could get complex. Plumbing, electrical, seasonal. Does it need Large tier for reasoning about old-house quirks?',
  'Should there be a dedicated Avatar/Identity agent? Daily avatar generation, banner changes, panel voice — currently spread across York + crons.',
  'What happens when two agents disagree? E.g., Health says "you need protein" but Chores says "the kitchen is a mess." York resolves, but how formally?',
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

.roster {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.roster-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.5rem 0.75rem;
  background: #0d1117;
  border-radius: 4px;
  font-size: 0.85rem;
}

.roster-item.idea {
  opacity: 0.7;
}

.roster-emoji { font-size: 1.2rem; flex-shrink: 0; }
.roster-name { color: #f0f6fc; font-weight: 600; min-width: 80px; }
.roster-role { color: #8b949e; flex: 1; }

.tag.draft-tag { background: #1f6feb; color: white; }
.tag.suggested-tag { background: #30363d; color: #8b949e; }
.tag.idea-tag { background: #21262d; color: #484f58; }

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
