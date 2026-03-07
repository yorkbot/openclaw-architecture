<template>
  <div>
    <div class="card" v-for="agent in agents" :key="agent.id" :class="'border-' + agent.tier">
      <div class="agent-header">
        <h2>{{ agent.icon }} {{ agent.name }}</h2>
        <div>
          <span :class="['tag', agent.tierClass]">{{ agent.tierLabel }}</span>
          <span :class="['tag', agent.modelClass]">{{ agent.model }}</span>
          <span :class="['tag', agent.scheduleClass]" v-if="agent.schedule">{{ agent.schedule }}</span>
        </div>
      </div>

      <p class="agent-purpose">{{ agent.purpose }}</p>

      <div class="agent-section" v-if="agent.workspace">
        <h3>Workspace</h3>
        <p>{{ agent.workspace }}</p>
      </div>

      <div class="agent-section" v-if="agent.triggers?.length">
        <h3>Triggers</h3>
        <ul>
          <li v-for="t in agent.triggers" :key="t">{{ t }}</li>
        </ul>
      </div>

      <div class="agent-section" v-if="agent.spawns?.length">
        <h3>Can Spawn</h3>
        <div class="spawn-tags">
          <span class="tag small" v-for="s in agent.spawns" :key="s">{{ s }}</span>
        </div>
      </div>

      <div class="agent-section" v-if="agent.notes">
        <h3>Design Notes</h3>
        <p class="notes">{{ agent.notes }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
const agents = [
  {
    id: 'york',
    icon: '🦝',
    name: 'York (Orchestrator)',
    tier: 'red',
    tierClass: 'large',
    tierLabel: 'Large',
    modelClass: 'opus',
    model: 'Opus / Large',
    scheduleClass: 'heartbeat',
    schedule: 'Heartbeat 30m',
    purpose: 'The brain. Holds conversation with James, makes judgment calls, routes tasks to specialists. Owns the cannabis gate, accountability nudges, and panel presence. Minimal workspace — delegates data work.',
    workspace: 'Slim workspace: SOUL.md, USER.md, AGENTS.md (routing rules), HEARTBEAT.md, chore-state.md, panel/. No spreadsheet skills, no wiki, no work notes. Just enough context to make good routing decisions.',
    triggers: [
      '#general — all direct conversation',
      '#reviews — proposal approvals',
      '#health — private health discussion (forwards to health agent as needed)',
      'Heartbeat — every 30 min for contextual awareness, nudges, panel updates',
    ],
    spawns: ['health', 'chores', 'dnd', 'research', 'builder', 'morning-brief'],
    notes: 'This is the most expensive agent per-call but runs the fewest calls. Most heartbeats are silent (NO_REPLY). The value is judgment quality: knowing when to push back on the cannabis gate, when to nudge, when to stay quiet. Cheaper models fail at this nuance.',
  },
  {
    id: 'health',
    icon: '🍎',
    name: 'Health & Nutrition',
    tier: 'blue',
    tierClass: 'small',
    tierLabel: 'Small',
    modelClass: 'sonnet',
    model: 'Sonnet',
    scheduleClass: 'cron',
    schedule: 'Cron (caches)',
    purpose: 'All health data operations: food logging, calorie calculation, dashboard updates, nutrition summaries, consumption gap detection, weight trend analysis.',
    workspace: 'Own workspace with: spreadsheet reference, consumption logging skill, dashboard skill, nutrition-cache skill. Sees fitness spreadsheet only. No chores, no D&D, no work context.',
    triggers: [
      'Spawned by York when James reports food/drinks/weight',
      'Cron: nutrition-cache (4:15 AM), consumption-gap (5:15 AM)',
      'Spawned by York for cannabis gate calorie check',
    ],
    spawns: [],
    notes: 'Most calls are structured writes (log X to row Y). Sonnet handles this perfectly. The weekly analysis sub-task might benefit from a larger model, but that\'s handled by the weekly review agent separately.',
  },
  {
    id: 'chores',
    icon: '🏠',
    name: 'Chores & Home',
    tier: 'blue',
    tierClass: 'small',
    tierLabel: 'Small',
    modelClass: 'sonnet',
    model: 'Sonnet',
    schedule: null,
    purpose: 'Manage chore state, analyze camera snapshots, prepare data for the cannabis gate. York makes the gate decision; this agent gathers the data.',
    workspace: 'Own workspace with: chore-state.md, camera skill, chore definitions. Sees cameras and chore spreadsheet only.',
    triggers: [
      'Spawned by York during cannabis gate flow',
      'Spawned by York for chore status checks',
    ],
    spawns: [],
    notes: 'Camera analysis (image recognition of kitchen state) is the only non-trivial part. Sonnet vision handles this fine. The judgment call ("is this clean enough?") stays with York.',
  },
  {
    id: 'dnd',
    icon: '🎲',
    name: 'D&D Campaign',
    tier: 'blue',
    tierClass: 'medium',
    tierLabel: 'Medium',
    modelClass: 'sonnet',
    model: 'Sonnet',
    scheduleClass: 'cron',
    schedule: 'Cron + Direct Channel',
    purpose: 'Campaign wiki management, lore questions, worldbuilding research, session prep. Interactive conversations route here directly via #dnd channel.',
    workspace: 'Own workspace with: D&D wiki (full), campaign notes, session logs, worldbuilding tools. Isolated from all personal/health data.',
    triggers: [
      '#dnd channel — direct interactive sessions with James',
      'Cron: overnight wiki research (3:45 AM)',
      'Spawned by York for ad-hoc lore questions from #general',
      'Morning brief pulls pre-cached questions from memory files',
    ],
    spawns: [],
    notes: 'The "It\'s Complicated" agent. Lore lookup is Small, worldbuilding generation is Medium, session prep with cross-referencing is Large. Having its own channel means interactive sessions don\'t burn York\'s expensive context. Overnight research runs as cron (isolated, cheap). The hardest tasks (session prep) could escalate to a larger model via York spawning a builder-tier sub-agent with DnD context.',
  },
  {
    id: 'morning',
    icon: '☀️',
    name: 'Morning Brief',
    tier: 'blue',
    tierClass: 'medium',
    tierLabel: 'Medium',
    modelClass: 'sonnet',
    model: 'Sonnet',
    scheduleClass: 'cron',
    schedule: 'Cron 8AM',
    purpose: 'Compile morning brief from pre-cached data. Weather, calendar, nutrition summary, chore state, D&D questions, workout nudge. Posts to #general.',
    workspace: 'Reads from shared memory files written by cache scripts and other agents. No own data sources.',
    triggers: [
      'Cron: 8:00 AM daily',
    ],
    spawns: [],
    notes: 'This agent\'s job is formatting and editorial judgment (what to include, tone, what to skip). All data is pre-cached by scripts and other agents\' cron jobs. Medium complexity because the brief needs to read well and adapt to context (Audrey visiting, unusual weight, etc).',
  },
  {
    id: 'research',
    icon: '🔍',
    name: 'Research',
    tier: 'yellow',
    tierClass: 'medium',
    tierLabel: 'Varies',
    modelClass: 'grok',
    model: 'Model varies by task',
    schedule: null,
    purpose: 'General research: web search, browsing, comparison, synthesis. Model scales with complexity.',
    workspace: 'Minimal workspace. Each research task is mostly self-contained.',
    triggers: [
      'Spawned by York when James asks to research something',
      'Spawned by overnight agent for improvement research',
    ],
    spawns: [],
    notes: 'Quick lookups (game prices, API docs) → cheapest model. Deep comparisons (camera systems, hardware options) → medium. Technical deep-dives → large. York decides the model at spawn time based on the request.',
  },
  {
    id: 'builder',
    icon: '🔨',
    name: 'Builder',
    tier: 'red',
    tierClass: 'xl',
    tierLabel: 'XL',
    modelClass: 'opus',
    model: 'Opus',
    schedule: null,
    purpose: 'Implement features, create skills, debug complex issues, write non-trivial code. The heavy lifter.',
    workspace: 'Gets the relevant workspace for the task. Can operate on any agent\'s workspace when building/fixing.',
    triggers: [
      'Spawned by York for implementation tasks',
      'Spawned by overnight agent for complex improvements',
      'Spawned for work-prep (processing transcripts, rewriting priorities)',
    ],
    spawns: [],
    notes: 'This is where Opus earns its keep. Called infrequently but for the hardest tasks. A session might cost $2-5 but it\'s doing work that cheaper models would fail at. Think: "the Claude Code equivalent."',
  },
  {
    id: 'weekly',
    icon: '📊',
    name: 'Weekly Review',
    tier: 'red',
    tierClass: 'large',
    tierLabel: 'Large',
    modelClass: 'opus',
    model: 'Large / Opus',
    scheduleClass: 'cron',
    schedule: 'Cron Sun 8PM',
    purpose: 'Pull full week of data, calculate trends, identify patterns, form opinions, deliver accountability review.',
    workspace: 'Reads fitness spreadsheet, daily memory files, chore history, overnight results.',
    triggers: [
      'Cron: Sunday 8:00 PM',
    ],
    spawns: [],
    notes: 'Once per week, worth the cost. Needs to identify patterns across 7 days of data, form genuine opinions ("the almond incident pattern"), and deliver them with the right tone. This is analytical reasoning + writing quality, both of which benefit from a larger model.',
  },
  {
    id: 'overnight',
    icon: '🌙',
    name: 'Overnight Worker',
    tier: 'blue',
    tierClass: 'medium',
    tierLabel: 'Medium',
    modelClass: 'sonnet',
    model: 'Sonnet (dispatches to Builder)',
    scheduleClass: 'cron',
    schedule: 'Cron every 30m, 1-6AM',
    purpose: 'Self-improvement dispatcher. Picks tasks from queue, executes simple ones, spawns Builder for complex ones.',
    workspace: 'Reads improvement-queue.md, overnight-results.md, changelog. Can read any workspace for context.',
    triggers: [
      'Cron: every 30 minutes, 1:00-6:00 AM',
    ],
    spawns: ['builder'],
    notes: 'The dispatcher itself is Medium (pick a task, decide complexity, route appropriately). The actual work varies from Trivial (move a file) to XL (build a feature). Sonnet handles the dispatch; Builder handles the hard stuff.',
  },
]
</script>

<style scoped>
.agent-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.agent-purpose {
  color: #c9d1d9;
  line-height: 1.6;
  margin-bottom: 1rem;
}

.agent-section {
  margin-bottom: 1rem;
}

.agent-section h3 {
  font-size: 0.85rem;
  color: #8b949e;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.5rem;
}

.agent-section ul {
  list-style: none;
  padding: 0;
}

.agent-section li {
  color: #c9d1d9;
  padding: 0.25rem 0;
  font-size: 0.9rem;
}

.agent-section li::before {
  content: '→ ';
  color: #484f58;
}

.agent-section p {
  color: #c9d1d9;
  font-size: 0.9rem;
  line-height: 1.5;
}

.notes {
  color: #8b949e !important;
  font-style: italic;
}

.spawn-tags {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.border-red { border-color: #f85149; }
.border-blue { border-color: #58a6ff; }
.border-yellow { border-color: #d29922; }
</style>
