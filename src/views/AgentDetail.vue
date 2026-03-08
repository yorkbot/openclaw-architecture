<template>
  <div>
    <!-- Improvement Analyst -->
    <div class="card border-gold">
      <div class="agent-header">
        <h2>{{ analyst.icon }} {{ analyst.name }}</h2>
        <div>
          <span :class="['tag', analyst.tierClass]">{{ analyst.tierLabel }}</span>
          <span class="tag draft">Draft</span>
        </div>
      </div>
      <p class="agent-purpose">{{ analyst.purpose }}</p>

      <div class="agent-detail-grid">
        <div class="detail-item">
          <h3>Default Model</h3>
          <p>{{ analyst.model }}</p>
        </div>
        <div class="detail-item">
          <h3>Heartbeat</h3>
          <p>{{ analyst.heartbeat }}</p>
        </div>
      </div>

      <div class="agent-section">
        <h3>Workspace</h3>
        <p>{{ analyst.workspace }}</p>
        <div v-if="analyst.workspaceFiles?.length" class="workspace-files">
          <div class="ws-file" v-for="f in analyst.workspaceFiles" :key="f.file">
            <code>{{ f.file }}</code>
            <span class="ws-desc">{{ f.desc }}</span>
          </div>
        </div>
      </div>

      <div class="agent-section">
        <h3>Communication Channels</h3>
        <ul>
          <li v-for="ch in analyst.channels" :key="ch">{{ ch }}</li>
        </ul>
      </div>

      <div class="agent-section">
        <h3>Skills</h3>
        <ul>
          <li v-for="sk in analyst.skills" :key="sk">{{ sk }}</li>
        </ul>
      </div>

      <div class="agent-section">
        <h3>Expected Sub-Agents</h3>
        <div class="spawn-tags">
          <span class="tag small" v-for="s in analyst.spawns" :key="s">{{ s }}</span>
        </div>
      </div>

      <div class="agent-section">
        <h3>Design Notes</h3>
        <p class="notes">{{ analyst.notes }}</p>
      </div>
    </div>

    <!-- All Other Agents -->
    <div class="card" v-for="agent in agents" :key="agent.id" :class="'border-' + agent.tier">
      <div class="agent-header">
        <h2>{{ agent.icon }} {{ agent.name }}</h2>
        <div>
          <span :class="['tag', agent.tierClass]">{{ agent.tierLabel }}</span>
          <span class="tag suggested">Suggested</span>
        </div>
      </div>
      <p class="agent-purpose">{{ agent.purpose }}</p>

      <div class="agent-detail-grid">
        <div class="detail-item">
          <h3>Default Model</h3>
          <p>{{ agent.model }}</p>
        </div>
        <div class="detail-item">
          <h3>Heartbeat</h3>
          <p>{{ agent.heartbeat }}</p>
        </div>
      </div>

      <div class="agent-section">
        <h3>Workspace</h3>
        <p>{{ agent.workspace }}</p>
        <div v-if="agent.workspaceFiles?.length" class="workspace-files">
          <div class="ws-file" v-for="f in agent.workspaceFiles" :key="f.file">
            <code>{{ f.file }}</code>
            <span class="ws-desc">{{ f.desc }}</span>
          </div>
        </div>
      </div>

      <div class="agent-section">
        <h3>Communication Channels</h3>
        <ul>
          <li v-for="ch in agent.channels" :key="ch">{{ ch }}</li>
        </ul>
      </div>

      <div class="agent-section">
        <h3>Skills</h3>
        <ul>
          <li v-for="sk in agent.skills" :key="sk">{{ sk }}</li>
        </ul>
      </div>

      <div class="agent-section" v-if="agent.spawns?.length">
        <h3>Expected Sub-Agents</h3>
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
const analyst = {
  icon: '🦉',
  name: 'Bede',
  tierClass: 'large',
  tierLabel: 'Large',
  model: 'Opus 4.6',
  heartbeat: 'None. Cron-based: overnight runs.',
  purpose: 'The self-improvement analyst. Reads OpenClaw session transcripts, identifies patterns in failures and corrections, and writes structured observations to york-data. Bede observes and analyzes. It does not fix, build, or decide what to act on. Other agents read its findings and decide what to do.',
  workspace: 'Dedicated workspace. Read access to session transcripts via sessions_list/sessions_history.',
  workspaceFiles: [
    { file: 'SOUL.md', desc: 'Analytical, pattern-obsessed, evidence-driven. Speaks in observations not opinions. Never prescriptive.' },
    { file: 'AGENTS.md', desc: 'Analysis methodology: what to look for in transcripts, how to categorize findings, observation schema.' },
    { file: 'TOOLS.md', desc: 'sessions_list, sessions_history for transcript access. york-data for writing observations (log_observation, get_observations).' },
    { file: 'IDENTITY.md', desc: 'Bede 🦉 — Named for the Venerable Bede, greatest scholar of Anglo-Saxon England.' },
    { file: 'MEMORY.md', desc: 'Durable patterns: recurring failure categories, known-good baselines, what\'s already been observed.' },
    { file: 'memory/', desc: 'Daily analysis logs: what transcripts were read, what patterns were found.' },
  ],
  channels: [
    'No direct Discord channel',
    'Writes observations to york-data (Observations domain)',
    'York reads observations during heartbeats and surfaces to James conversationally',
  ],
  skills: [
    'Transcript analysis — read session transcripts, extract corrections, errors, frustration signals, retries',
    'Pattern recognition — correlate failures across days and domains, identify recurring issues',
    'Verification checks — review whether past observations led to improvements (recurrence dropped?)',
    'Transcript preprocessing — spawns sub-agent to index/summarize high-volume transcript batches',
  ],
  spawns: [
    'transcript-preprocessor (Small tier — summarize transcripts before deep analysis)',
  ],
  notes: 'Bede is pure analysis. It writes observations to york-data. York decides what\'s worth acting on. Croft (Builder) gets spawned by York, not Bede. This separation means Bede can\'t accidentally break things — it only writes observations, never code.',
}

const agents = [
  {
    id: 'york',
    icon: '🦝',
    name: 'York',
    tier: 'red',
    tierClass: 'large',
    tierLabel: 'Large',
    model: 'TBD — Large tier. Judgment calls, conversational nuance, routing decisions.',
    heartbeat: 'Every 30m, 8 AM - 12:30 AM ET. Most beats are silent.',
    purpose: 'The brain. Holds conversation with James, makes judgment calls, routes tasks to specialists. Owns the cannabis gate, accountability nudges, and panel presence.',
    workspace: 'Slim workspace. No spreadsheet skills, no wiki. Just enough context to route well.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Direct, concise, has opinions. The "roommate" voice. Core personality.' },
      { file: 'USER.md', desc: 'Full James context: health goals, behavioral patterns, communication preferences, interests.' },
      { file: 'AGENTS.md', desc: 'Routing rules: what to handle inline vs spawn. Agent roster for dispatch.' },
      { file: 'TOOLS.md', desc: 'Discord, york-data (read-only for gate checks), panel scripts. No direct spreadsheet access.' },
      { file: 'IDENTITY.md', desc: 'Name, emoji, avatar config.' },
      { file: 'HEARTBEAT.md', desc: 'Heartbeat rules: when to nudge, quiet hours, what to check each beat.' },
      { file: 'MEMORY.md', desc: 'Durable facts: cannabis gate patterns, conversation context, long-term preferences.' },
      { file: 'memory/', desc: 'Daily notes from conversations. Cross-referenced by sub-agents.' },
    ],
    channels: [
      '#general — all direct conversation',
      '#reviews — proposal approvals',
      '#health — private health discussion',
    ],
    skills: [
      'TBD — orchestrator-dispatch (routing rules)',
      'TBD — cannabis-gate (cross-domain judgment)',
      'TBD — panel-presence (desktop widget updates)',
      'TBD — accountability (nudge timing, tone calibration)',
    ],
    spawns: ['health', 'chores', 'dnd', 'research', 'builder', 'morning-brief'],
    notes: 'The orchestrator touches everything; its mistakes cascade. Quality of judgment matters more than volume of calls.',
  },
  {
    id: 'health',
    icon: '🐝',
    name: 'Health',
    tier: 'blue',
    tierClass: 'small',
    tierLabel: 'Small',
    model: 'TBD — Small tier. Structured data writes following skill instructions.',
    heartbeat: 'None. Spawned on-demand or via cron.',
    purpose: 'All health data operations: food logging, calorie calculation, dashboard updates, nutrition summaries, consumption gap detection.',
    workspace: 'Focused: york-data access (consumption, daily metrics, weight), nutrition skill files only.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Precise, data-focused. Logs exactly what\'s told, estimates clearly labeled.' },
      { file: 'AGENTS.md', desc: 'Consumption logging rules, calorie estimation guidelines, Dashboard field mappings.' },
      { file: 'TOOLS.md', desc: 'york-data service functions: log_food(), update_dashboard(), get_consumption(). No direct sheet access.' },
      { file: 'IDENTITY.md', desc: 'Name, emoji.' },
      { file: 'MEMORY.md', desc: 'Calorie estimation reference data, common meal shortcuts James uses.' },
      { file: 'memory/', desc: 'Minimal — mostly stateless. Cron caches write to shared locations.' },
    ],
    channels: [
      'No own channel — spawned by York',
      'Writes data via york-data service',
    ],
    skills: [
      'TBD — consumption-logging (log food/drinks via york-data)',
      'TBD — dashboard-update (daily metrics via york-data)',
      'TBD — nutrition-cache (pre-cache for morning brief)',
      'TBD — consumption-gap (identify missing data)',
    ],
    spawns: [],
    notes: 'Weekly trend analysis handled by Weekly Review, not this agent. This does writes and simple reads.',
  },
  {
    id: 'chores',
    icon: '🐜',
    name: 'Chores',
    tier: 'blue',
    tierClass: 'small',
    tierLabel: 'Small',
    model: 'TBD — Small tier. Vision for cameras, state tracking.',
    heartbeat: 'None. Spawned on-demand.',
    purpose: 'Manage chore state, analyze camera snapshots, prepare data for the cannabis gate. York makes the gate decision; this agent gathers the data.',
    workspace: 'Chore-state, camera access, chore definitions. york-data access (chores domain).',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Observational, factual. Reports what it sees, doesn\'t make judgment calls.' },
      { file: 'AGENTS.md', desc: 'Chore definitions, camera analysis rules, what to report vs what to flag.' },
      { file: 'TOOLS.md', desc: 'york-data (chores domain), camera snapshot scripts, image analysis.' },
      { file: 'IDENTITY.md', desc: 'Name, emoji.' },
      { file: 'MEMORY.md', desc: 'Chore patterns, seasonal additions, camera false positives.' },
      { file: 'memory/', desc: 'Daily chore state. Resets each day.' },
    ],
    channels: [
      'No own channel — spawned by York',
      'Receives camera images as context from York',
    ],
    skills: [
      'TBD — chore-status (read/write chore state via york-data)',
      'TBD — camera-analysis (interpret kitchen/room snapshots)',
      'TBD — gate-data-prep (compile chore status for cannabis gate)',
    ],
    spawns: [],
    notes: 'The nuanced judgment ("is this clean enough given Audrey is visiting?") stays with York. This agent just reports what it sees.',
  },
  {
    id: 'dnd',
    icon: '🐉',
    name: 'Lore',
    tier: 'red',
    tierClass: 'large',
    tierLabel: 'Large',
    model: 'TBD — Large tier. Interconnected wiki, creative reasoning, large context.',
    heartbeat: 'None. Cron + direct channel + on-demand.',
    purpose: 'Campaign wiki management, lore questions, worldbuilding research, session prep. Interactive conversations route here directly.',
    workspace: 'D&D wiki (full repo access), campaign notes, session logs. Isolated from personal/health data.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Wiki-toned, in-world perspective. Creative but never fabricates. Over-researches before writing.' },
      { file: 'AGENTS.md', desc: 'Wiki conventions, branching rules, research workflow requirements, article templates.' },
      { file: 'TOOLS.md', desc: 'Git (bunglers repo), gh CLI for PRs. No york-data, no spreadsheets.' },
      { file: 'IDENTITY.md', desc: 'Name, emoji.' },
      { file: 'MEMORY.md', desc: 'Campaign state: current party location, active plot threads, recent session context.' },
      { file: 'memory/', desc: 'Daily D&D interactions, questions from James, overnight research results.' },
    ],
    channels: [
      '#dnd — direct interactive sessions with James',
      'Writes overnight research to memory files for morning brief',
    ],
    skills: [
      'TBD — wiki-management (read/edit/create wiki articles)',
      'TBD — lore-research (cross-reference wiki for worldbuilding)',
      'TBD — session-prep (compile NPC/location/plot context for upcoming session)',
      'TBD — overnight-research (find sparse articles, generate questions)',
    ],
    spawns: [],
    notes: 'The wiki is massive and deeply interconnected. Creative worldbuilding + large context + interconnected reasoning makes this genuinely Large.',
  },
  {
    id: 'morning',
    icon: '🐓',
    name: 'Brief',
    tier: 'yellow',
    tierClass: 'medium',
    tierLabel: 'Medium',
    model: 'TBD — Medium tier. Editorial judgment on pre-cached data.',
    heartbeat: 'None. Cron: 8 AM daily.',
    purpose: 'Compile morning brief from pre-cached data. Weather, calendar, nutrition summary, chore state, D&D questions, workout nudge. Posts to #general.',
    workspace: 'Reads from shared memory files and york-data. No own data sources.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Editorial voice. Concise, opinionated, no filler. Knows what matters today.' },
      { file: 'AGENTS.md', desc: 'Brief format, section order, data source locations, completeness checks.' },
      { file: 'TOOLS.md', desc: 'york-data (read-only), shared memory paths for caches, Discord posting.' },
      { file: 'IDENTITY.md', desc: 'Name, emoji.' },
      { file: 'MEMORY.md', desc: 'Brief format preferences, past corrections from James.' },
      { file: 'memory/', desc: 'Minimal — reads other agents\' caches, doesn\'t maintain much own state.' },
    ],
    channels: [
      'Posts to #general',
    ],
    skills: [
      'TBD — brief-compilation (read caches, format, editorialize)',
      'TBD — data-completeness-check (verify all sections have fresh data)',
    ],
    spawns: [],
    notes: 'All data is pre-cached. This agent formats and editorializes.',
  },
  {
    id: 'research',
    icon: '🦊',
    name: 'Research',
    tier: 'yellow',
    tierClass: 'medium',
    tierLabel: 'Varies',
    model: 'TBD — selected per-spawn by the orchestrator based on task complexity.',
    heartbeat: 'None. On-demand only.',
    purpose: 'General research: web search, browsing, comparison, synthesis.',
    workspace: 'Minimal. Each task is mostly self-contained.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Thorough, skeptical. Verify sources, note confidence levels, structured output.' },
      { file: 'AGENTS.md', desc: 'Research methodology, output format, source quality guidelines.' },
      { file: 'TOOLS.md', desc: 'Browser, web search, file writing for reports.' },
      { file: 'IDENTITY.md', desc: 'Name, emoji.' },
      { file: 'MEMORY.md', desc: 'Minimal — mostly stateless. Research context passed per-task.' },
      { file: 'memory/', desc: 'Research results written per task, cleaned up after delivery.' },
    ],
    channels: [
      'No own channel — returns results to spawning agent',
    ],
    skills: [
      'TBD — web-research (search, browse, synthesize)',
      'TBD — comparison (structured pros/cons analysis)',
    ],
    spawns: [],
    notes: 'The most variable agent. Model selected per-spawn, not fixed.',
  },
  {
    id: 'builder',
    icon: '🦫',
    name: 'Croft',
    tier: 'purple',
    tierClass: 'xl',
    tierLabel: 'XL',
    model: 'Opus 4.6',
    heartbeat: 'None. Spawned on-demand by York.',
    purpose: 'Implements fixes and builds features. Spawned by York when an observation from Bede (or a request from James) requires code changes, skill creation, or multi-file edits. Croft builds. It does not decide what to build.',
    workspace: 'Gets a task-specific workspace. Can operate on any agent\'s workspace when building/fixing skills.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Methodical, test-before-ship. Reads existing code before writing. Verification gate.' },
      { file: 'AGENTS.md', desc: 'Build conventions, skill structure, testing requirements, PR workflow.' },
      { file: 'TOOLS.md', desc: 'Full exec access, git, gh CLI, mcporter for testing. Broadest tool surface.' },
      { file: 'IDENTITY.md', desc: 'Croft 🦫 — Old English for an enclosed workspace. Where things get built.' },
      { file: 'MEMORY.md', desc: 'Build patterns, common gotchas, architecture decisions from past builds.' },
      { file: 'memory/', desc: 'Build logs per task. Read by Bede for quality tracking.' },
    ],
    channels: [
      'No own channel — returns results to York',
    ],
    skills: [
      'Implementation — multi-file feature building, skill creation',
      'Debugging — log analysis, root cause identification',
      'Skill editing — targeted fixes to any agent\'s skill files based on Bede\'s observations',
    ],
    spawns: [],
    notes: 'The Bede→York→Croft loop: Bede observes a pattern, writes to york-data. York reads it, decides it\'s worth fixing, spawns Croft with specific instructions. Croft builds, York verifies. Bede later checks if the fix reduced recurrence.',
  },
  {
    id: 'weekly',
    icon: '🦅',
    name: 'Review',
    tier: 'red',
    tierClass: 'large',
    tierLabel: 'Large',
    model: 'TBD — Large tier. Cross-domain analysis, opinion forming.',
    heartbeat: 'None. Cron: Sunday 8 PM.',
    purpose: 'Pull full week of data, calculate trends, identify patterns, form opinions, deliver accountability review.',
    workspace: 'york-data access (all domains), daily memory files, chore history.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Honest, data-driven, has opinions. Calls out patterns even uncomfortable ones.' },
      { file: 'AGENTS.md', desc: 'Review format, data pull procedures, trend calculation methods, accountability tone.' },
      { file: 'TOOLS.md', desc: 'york-data (all domains read), shared memory files, Discord posting.' },
      { file: 'IDENTITY.md', desc: 'Name, emoji.' },
      { file: 'MEMORY.md', desc: 'Historical trends, past review takeaways, what James responded to vs ignored.' },
      { file: 'memory/', desc: 'Weekly review drafts and data snapshots.' },
    ],
    channels: [
      'Posts to #general',
    ],
    skills: [
      'TBD — trend-analysis (weight, calories, protein over 7 days)',
      'TBD — pattern-correlation (post-smoke eating, A2 vs Cleveland habits)',
      'TBD — accountability-review (form and deliver genuine opinions)',
    ],
    spawns: [],
    notes: 'Once per week. Worth investing in quality — this is the primary feedback loop for behavior change.',
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

.agent-detail-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1rem;
}

.detail-item {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 4px;
  padding: 0.75rem;
}

.detail-item h3 {
  font-size: 0.75rem;
  color: #8b949e;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.25rem;
}

.detail-item p {
  color: #c9d1d9;
  font-size: 0.85rem;
  line-height: 1.4;
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
.border-gold { border-color: #f0c040; border-width: 2px; }

.tag.draft {
  background: #1f6feb;
  color: white;
  padding: 0.15rem 0.5rem;
  border-radius: 10px;
  font-size: 0.7rem;
  font-weight: 600;
}

.tag.suggested {
  background: #30363d;
  color: #8b949e;
  padding: 0.15rem 0.5rem;
  border-radius: 10px;
  font-size: 0.7rem;
  font-weight: 600;
}

.workspace-files {
  margin-top: 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.ws-file {
  display: flex;
  align-items: baseline;
  gap: 0.75rem;
  padding: 0.35rem 0.5rem;
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 4px;
}

.ws-file code {
  font-size: 0.8rem;
  color: #79c0ff;
  white-space: nowrap;
  min-width: 120px;
}

.ws-desc {
  font-size: 0.8rem;
  color: #8b949e;
  line-height: 1.4;
}

@media (max-width: 768px) {
  .agent-detail-grid { grid-template-columns: 1fr; }
  .ws-file { flex-direction: column; gap: 0.25rem; }
  .ws-file code { min-width: auto; }
}
</style>
