<template>
  <div>
    <div class="card" v-for="agent in agents" :key="agent.id" :class="'border-' + agent.tier">
      <div class="agent-header">
        <h2>{{ agent.icon }} {{ agent.name }}</h2>
        <div>
          <span :class="['tag', agent.tierClass]">{{ agent.tierLabel }}</span>
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

      <div class="agent-section" v-if="agent.whyThisTier">
        <h3>Why This Tier</h3>
        <p class="notes">{{ agent.whyThisTier }}</p>
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
    whyThisTier: 'The orchestrator makes nuanced judgment calls: when to push back on the cannabis gate, when to nudge vs stay quiet, how to read tone in a conversation. This is where model quality directly impacts the user experience. A smaller model would rubber-stamp the gate and miss conversational nuance.',
    notes: 'Most heartbeats are silent (NO_REPLY). The value is quality of judgment, not volume of calls. This agent runs the fewest tokens per day but they matter the most.',
  },
  {
    id: 'health',
    icon: '🍎',
    name: 'Health & Nutrition',
    tier: 'blue',
    tierClass: 'small',
    tierLabel: 'Small',
    scheduleClass: 'cron',
    schedule: 'Cron (caches)',
    purpose: 'All health data operations: food logging, calorie calculation, dashboard updates, nutrition summaries, consumption gap detection.',
    workspace: 'Own workspace with: spreadsheet reference, consumption logging skill, dashboard skill, nutrition-cache skill. Sees fitness spreadsheet only. No chores, no D&D, no work context.',
    triggers: [
      'Spawned by York when James reports food/drinks/weight',
      'Cron: nutrition-cache (4:15 AM), consumption-gap (5:15 AM)',
      'Spawned by York for cannabis gate calorie check',
    ],
    spawns: [],
    whyThisTier: 'Structured data operations: "log X to row Y with these calories." Follows skill file instructions. The hardest part is calorie estimation for ambiguous foods, which any reasonable model handles fine.',
    notes: 'Weekly trend analysis is handled by the Weekly Review agent, not this one. This agent does writes and simple reads.',
  },
  {
    id: 'chores',
    icon: '🏠',
    name: 'Chores & Home',
    tier: 'blue',
    tierClass: 'small',
    tierLabel: 'Small',
    scheduleClass: null,
    schedule: null,
    purpose: 'Manage chore state, analyze camera snapshots, prepare data for the cannabis gate. York makes the gate decision; this agent gathers the data.',
    workspace: 'Own workspace with: chore-state.md, camera skill, chore definitions. Sees cameras and chore spreadsheet only.',
    triggers: [
      'Spawned by York during cannabis gate flow',
      'Spawned by York for chore status checks',
    ],
    spawns: [],
    whyThisTier: 'Camera analysis (is the kitchen clean?) plus reading/writing chore state. Vision capability needed but the judgment is simple: dishes or no dishes, clutter or clean.',
    notes: 'The nuanced judgment ("is this clean enough given Audrey is visiting?") stays with York. This agent just reports what it sees.',
  },
  {
    id: 'dnd',
    icon: '🎲',
    name: 'D&D Campaign',
    tier: 'red',
    tierClass: 'large',
    tierLabel: 'Large',
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
    whyThisTier: 'The wiki is massive and deeply interconnected. A lore question about one character touches factions, timelines, locations, and other characters across dozens of articles. The model needs to hold all that context and reason about connections, contradictions, and implications. Session prep is even harder — cross-referencing player histories, NPC motivations, and plot threads. A smaller model would give shallow, disconnected answers.',
    notes: 'Having its own channel means interactive sessions don\'t burn York\'s context. Overnight research runs as cron (isolated). The combination of creative worldbuilding + large context + interconnected reasoning makes this genuinely Large.',
  },
  {
    id: 'morning',
    icon: '☀️',
    name: 'Morning Brief',
    tier: 'yellow',
    tierClass: 'medium',
    tierLabel: 'Medium',
    scheduleClass: 'cron',
    schedule: 'Cron 8AM',
    purpose: 'Compile morning brief from pre-cached data. Weather, calendar, nutrition summary, chore state, D&D questions, workout nudge. Posts to #general.',
    workspace: 'Reads from shared memory files written by cache scripts and other agents. No own data sources.',
    triggers: [
      'Cron: 8:00 AM daily',
    ],
    spawns: [],
    whyThisTier: 'The data is pre-cached, so no heavy lifting. But the brief needs editorial judgment: what to include, what tone to use, how to frame the weight number, when to skip a section. It needs to read well and adapt to context (Audrey visiting, bad calorie day, etc).',
    notes: 'All data is pre-cached by scripts and other agents\' cron jobs. This agent formats and editorializes.',
  },
  {
    id: 'research',
    icon: '🔍',
    name: 'Research',
    tier: 'yellow',
    tierClass: 'medium',
    tierLabel: 'Varies',
    scheduleClass: null,
    schedule: null,
    purpose: 'General research: web search, browsing, comparison, synthesis. Complexity varies per task.',
    workspace: 'Minimal workspace. Each research task is mostly self-contained.',
    triggers: [
      'Spawned by York when James asks to research something',
      'Spawned by overnight agent for improvement research',
    ],
    spawns: [],
    whyThisTier: 'Range is huge. Quick lookups (game prices, API docs) are Small. Deep comparisons (hardware options, architecture decisions) are Large. York decides the model at spawn time based on the request.',
    notes: 'The model is selected per-spawn, not fixed. This is the most variable agent in terms of what it needs.',
  },
  {
    id: 'builder',
    icon: '🔨',
    name: 'Builder',
    tier: 'purple',
    tierClass: 'xl',
    tierLabel: 'XL',
    scheduleClass: null,
    schedule: null,
    purpose: 'Implement features, create skills, debug complex issues, write non-trivial code. The heavy lifter.',
    workspace: 'Gets the relevant workspace for the task. Can operate on any agent\'s workspace when building/fixing.',
    triggers: [
      'Spawned by York for implementation tasks',
      'Spawned by overnight agent for complex improvements',
      'Spawned for work-prep (processing transcripts, rewriting priorities)',
    ],
    spawns: [],
    whyThisTier: 'This is Claude Code territory. Multi-file implementation, debugging from logs, architectural decisions, writing code that needs to work on the first try. Called infrequently but for the hardest tasks.',
    notes: 'A session might be expensive but it\'s doing work that smaller models would fail at or iterate on endlessly. The right model here saves money by getting it right the first time.',
  },
  {
    id: 'weekly',
    icon: '📊',
    name: 'Weekly Review',
    tier: 'red',
    tierClass: 'large',
    tierLabel: 'Large',
    scheduleClass: 'cron',
    schedule: 'Cron Sun 8PM',
    purpose: 'Pull full week of data, calculate trends, identify patterns, form opinions, deliver accountability review.',
    workspace: 'Reads fitness spreadsheet, daily memory files, chore history, overnight results.',
    triggers: [
      'Cron: Sunday 8:00 PM',
    ],
    spawns: [],
    whyThisTier: 'Needs to identify patterns across 7 days of data, correlate behaviors (post-smoke eating, A2 vs Cleveland habits), and form genuine opinions that James will actually listen to. Surface-level summary isn\'t useful — the value is in the analysis.',
    notes: 'Once per week. Worth investing in quality here because this is the primary feedback loop for behavior change.',
  },
  {
    id: 'overnight',
    icon: '🌙',
    name: 'Overnight Worker',
    tier: 'yellow',
    tierClass: 'medium',
    tierLabel: 'Medium',
    scheduleClass: 'cron',
    schedule: 'Cron every 30m, 1-6AM',
    purpose: 'Self-improvement dispatcher. Picks tasks from queue, executes simple ones, spawns Builder for complex ones.',
    workspace: 'Reads improvement-queue.md, overnight-results.md, changelog. Can read any workspace for context.',
    triggers: [
      'Cron: every 30 minutes, 1:00-6:00 AM',
    ],
    spawns: ['builder'],
    whyThisTier: 'The dispatcher itself is Medium: read the queue, assess task complexity, decide whether to do it inline or spawn Builder. The actual hard work is delegated to Builder.',
    notes: 'Runs 12 times per night. Most runs are quick (pick task, do simple fix or spawn Builder, log result). The medium model handles the routing judgment fine.',
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
.border-purple { border-color: #bc8cff; }
</style>
