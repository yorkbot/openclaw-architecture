<template>
  <div>
    <!-- Draft Agents (Bede, Offa) -->
    <div class="card" v-for="agent in draftAgents" :key="agent.id" :class="'border-' + agent.borderColor">
      <div class="agent-header">
        <h2>{{ agent.icon }} {{ agent.name }}</h2>
        <div>
          <span class="tag draft">Draft</span>
        </div>
      </div>
      <p class="agent-purpose">{{ agent.purpose }}</p>

      <div class="agent-detail-grid">
        <div class="detail-item">
          <h3>Default Model</h3>
          <p>{{ agent.model }}</p>
        </div>
        <div class="detail-item">
          <h3>Cron Schedule</h3>
          <p>{{ agent.cron }}</p>
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
          <li v-for="sk in agent.skills" :key="sk.name">
            <strong>{{ sk.name }}</strong> — {{ sk.desc }}
          </li>
        </ul>
      </div>

      <div class="agent-section" v-if="agent.spawns?.length">
        <h3>Expected Sub-Agents</h3>
        <div class="spawn-tags">
          <span class="tag small" v-for="s in agent.spawns" :key="s">{{ s }}</span>
        </div>
      </div>
    </div>

    <!-- Suggested Agents (everything else) -->
    <div class="card" v-for="agent in suggestedAgents" :key="agent.id" :class="'border-' + agent.borderColor">
      <div class="agent-header">
        <h2>{{ agent.icon }} {{ agent.name }}</h2>
        <div>
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
          <h3>Cron Schedule</h3>
          <p>{{ agent.cron }}</p>
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
    </div>
  </div>
</template>

<script setup>
const draftAgents = [
  {
    id: 'bede',
    icon: '🦉',
    name: 'Bede',
    borderColor: 'opus',
    model: 'Opus 4.6',
    cron: 'Overnight runs (TBD schedule)',
    purpose: 'The self-improvement analyst. Reads session transcripts and agent memory, identifies patterns in failures and corrections, suggests measurable improvements, and tracks whether past changes were effective. Writes all findings and suggestions to york-data.',
    workspace: 'Dedicated workspace. Read access to session transcripts via sessions_list/sessions_history and other agent memory files.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Analytical, pattern-obsessed, evidence-driven. Speaks in observations not opinions.' },
      { file: 'AGENTS.md', desc: 'Analysis methodology: what to look for, how to categorize findings, observation and suggestion schemas.' },
      { file: 'TOOLS.md', desc: 'sessions_list, sessions_history for transcript access. york-data for reading and writing observations/suggestions. memory_search for agent memory files.' },
      { file: 'IDENTITY.md', desc: 'Bede 🦉 — Named for the Venerable Bede, greatest scholar of Anglo-Saxon England.' },
      { file: 'MEMORY.md', desc: 'Durable patterns: recurring failure categories, known-good baselines, what\'s already been observed.' },
      { file: 'memory/', desc: 'Daily analysis logs: what transcripts were read, what patterns were found.' },
    ],
    channels: [
      'Writes observations and suggestions to york-data',
    ],
    skills: [
      { name: 'Collect', desc: 'Gather data from session transcripts (sessions_list/sessions_history), agent memory files (memory_search), and york-data (past observations, measurement results). Assembles raw material for analysis.' },
      { name: 'Analyze', desc: 'Turn collected data into structured findings. Extract corrections, errors, frustration signals, retries. Correlate failures across days and domains. Report findings objectively.' },
      { name: 'Suggest', desc: 'Propose specific, actionable improvements based on findings. Each suggestion must include: what to change, why, and how to measure whether it worked. If it\'s not measurable, it\'s not a suggestion.' },
      { name: 'Measure', desc: 'Check whether past suggestions that were implemented actually improved things. Compare recurrence rates before and after. Results feed back into york-data and inform future Collect/Analyze cycles.' },
    ],
    spawns: [
      'transcript-preprocessor (Small tier — summarize high-volume transcript batches before deep analysis)',
    ],
  },
  {
    id: 'offa',
    icon: '🦫',
    name: 'Offa',
    borderColor: 'opus',
    model: 'Opus 4.6',
    cron: 'None. Spawned on-demand.',
    purpose: 'Implements changes. Reads suggestions from york-data, builds what\'s described: new skills, skill adjustments, config changes, scripts. Verifies its own work before reporting completion.',
    workspace: 'Task-specific. Can operate on any agent\'s workspace when building or fixing skills.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Methodical, test-before-ship. Reads existing code before writing. Verification gate.' },
      { file: 'AGENTS.md', desc: 'Build conventions, skill structure, testing requirements, PR workflow.' },
      { file: 'TOOLS.md', desc: 'Full exec access, git, gh CLI, mcporter for testing. york-data for reading suggestions and marking them as implemented.' },
      { file: 'IDENTITY.md', desc: 'Offa 🦫 — Named for King Offa of Mercia, builder of the 150-mile Offa\'s Dyke.' },
      { file: 'MEMORY.md', desc: 'Build patterns, common gotchas, architecture decisions from past builds.' },
      { file: 'memory/', desc: 'Build logs per task. Read by Bede for quality tracking.' },
    ],
    channels: [
      'Reads suggestions from york-data',
      'Writes build results and verification status back to york-data',
    ],
    skills: [
      { name: 'Build', desc: 'Implement the changes described in a suggestion. New skills, skill edits, config changes, scripts. Receives specific context about what and why.' },
      { name: 'Verify', desc: 'Confirm the changes were made correctly. Read files back, test where possible, validate that the implementation matches the suggestion\'s intent.' },
    ],
    spawns: [],
  },
]

const suggestedAgents = [
  {
    id: 'york',
    icon: '🦝',
    name: 'York',
    borderColor: 'none',
    model: 'TBD',
    cron: 'Heartbeat: every 30m, 8 AM - 12:30 AM ET',
    purpose: 'Holds conversation with James, makes judgment calls, routes tasks to specialists. Owns the cannabis gate, accountability nudges, and panel presence.',
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
    spawns: ['health', 'chores', 'dnd', 'research', 'morning-brief'],
  },
  {
    id: 'health',
    icon: '🐝',
    name: 'Health',
    borderColor: 'none',
    model: 'TBD',
    cron: 'TBD — caches run overnight',
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
      'Writes data via york-data service',
    ],
    skills: [
      'TBD — consumption-logging (log food/drinks via york-data)',
      'TBD — dashboard-update (daily metrics via york-data)',
      'TBD — nutrition-cache (pre-cache for morning brief)',
      'TBD — consumption-gap (identify missing data)',
    ],
    spawns: [],
  },
  {
    id: 'chores',
    icon: '🐜',
    name: 'Chores',
    borderColor: 'none',
    model: 'TBD',
    cron: 'None. Spawned on-demand.',
    purpose: 'Manage chore state, analyze camera snapshots, prepare data for the cannabis gate.',
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
      'Receives camera images as context',
    ],
    skills: [
      'TBD — chore-status (read/write chore state via york-data)',
      'TBD — camera-analysis (interpret kitchen/room snapshots)',
      'TBD — gate-data-prep (compile chore status for cannabis gate)',
    ],
    spawns: [],
  },
  {
    id: 'dnd',
    icon: '🐉',
    name: 'Lore',
    borderColor: 'none',
    model: 'TBD',
    cron: 'Overnight wiki research (TBD schedule)',
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
  },
  {
    id: 'morning',
    icon: '🐓',
    name: 'Brief',
    borderColor: 'none',
    model: 'TBD',
    cron: '8 AM daily',
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
  },
  {
    id: 'research',
    icon: '🦊',
    name: 'Research',
    borderColor: 'none',
    model: 'TBD',
    cron: 'None. Spawned on-demand.',
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
      'Returns results to spawning agent',
    ],
    skills: [
      'TBD — web-research (search, browse, synthesize)',
      'TBD — comparison (structured pros/cons analysis)',
    ],
    spawns: [],
  },
  {
    id: 'weekly',
    icon: '🦅',
    name: 'Review',
    borderColor: 'none',
    model: 'TBD',
    cron: 'Sunday 8 PM',
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

.spawn-tags {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.border-opus { border-color: #f0c040; border-width: 2px; }
.border-none { border-color: #30363d; }

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
