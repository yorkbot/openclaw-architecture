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

      <div class="agent-section" v-if="agent.skillSections">
        <h3>Skills</h3>
        <div class="skill-category" v-for="cat in agent.skillSections" :key="cat.name">
          <h4 class="skill-cat-header">{{ cat.name }}</h4>
          <div class="skill-items">
            <div class="skill-item" v-for="sk in cat.skills" :key="sk.name">
              <div class="skill-name">{{ sk.name }}</div>
              <div class="skill-desc">{{ sk.desc }}</div>
            </div>
            <div class="skill-tbd" v-if="cat.tbd">+ more TBD as agents are defined</div>
          </div>
        </div>
      </div>

      <div class="agent-section" v-else-if="agent.skills">
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

      <div class="agent-section" v-if="agent.skillSections">
        <h3>Skills</h3>
        <div class="skill-category" v-for="cat in agent.skillSections" :key="cat.name">
          <h4 class="skill-cat-header">{{ cat.name }}</h4>
          <div class="skill-items">
            <div class="skill-item" v-for="sk in cat.skills" :key="sk.name">
              <div class="skill-name">{{ sk.name }}</div>
              <div class="skill-desc">{{ sk.desc }}</div>
            </div>
            <div class="skill-tbd" v-if="cat.tbd">+ more TBD as agents are defined</div>
          </div>
        </div>
      </div>

      <div class="agent-section" v-else-if="agent.skills">
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
    skillSections: [
      {
        name: 'Collect',
        skills: [
          { name: 'Collect Transcripts', desc: 'Read session transcripts from all agents. Extract corrections, errors, retries, frustration signals. The universal collection skill — every agent produces transcripts.' },
          { name: 'Collect Health Pulse', desc: 'Frequent, lightweight. Runs every few hours. Keeps a rolling tab of last ~2 days of health data from york-data (Wynn\'s domain). Can\'t be more than a handful of hours behind.' },
          { name: 'Collect Health Daily', desc: 'Daily compilation. Pulls yesterday\'s consumption, calculates calorie/protein totals, weight entry, workout activity. Structured summary to york-data. Used by morning brief and other agents.' },
          { name: 'Collect Health Deep', desc: 'Weekly, comprehensive. Bigger model. Pulls full week of health data, calculates trends, identifies patterns across domains. Writes detailed analysis to york-data.' },
          { name: 'Collect Consumption Gaps', desc: 'Analyze recent days for missing data — no food logged, missing daily metrics, cannabis sessions unrecorded. Write specific gap questions to york-data for follow-up.' },
          { name: 'Collect Build Results', desc: 'Collects build results and verification status from Offa. Did what got built actually work? Feeds into the improvement loop.' },
        ],
        tbd: true,
      },
      {
        name: 'Analyze',
        skills: [
          { name: 'Measure Outcomes', desc: 'Check whether implemented suggestions actually reduced recurrence. Compare before and after. Write measurement results to york-data.' },
          { name: 'Verify Build', desc: 'Confirm Offa\'s changes were made correctly. Read files back, test where possible, validate that the implementation matches the suggestion\'s intent. Uses Collect Build Results as input.' },
        ],
        tbd: true,
      },
      {
        name: 'Suggest',
        skills: [],
        tbd: true,
      },
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
    purpose: 'Implements changes. Reads suggestions from york-data, builds what\'s described: new skills, skill adjustments, config changes, scripts. Reports build results to york-data for Bede to verify.',
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
    skillSections: [
      {
        name: 'Build',
        skills: [
          { name: 'New Skills', desc: 'Create a new skill from scratch. SKILL.md, supporting scripts, reference data. Follows skill structure conventions.' },
          { name: 'Skill Edits', desc: 'Modify an existing skill. Read current state, apply changes, preserve what works. Handles prompt tuning, logic changes, data format updates.' },
        ],
        tbd: true,
      },
    ],
    spawns: [],
  },
  {
    id: 'wynn',
    icon: '🦌',
    name: 'Wynn',
    borderColor: 'sonnet',
    model: 'Sonnet (can spawn Opus sub-agent for complex tasks)',
    cron: 'On-demand for logging and coaching.',
    purpose: 'All health and fitness data operations. Nutrition logging, calorie estimation, weight tracking, workout logging, exercise guidance. Owns the data entry layer — Bede handles collection and analysis of that data.',
    workspace: 'Dedicated workspace. york-data access for all health domains (consumption, daily metrics, workouts, cannabis).',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Precise on data, encouraging on fitness. Logs exactly what\'s told, estimates clearly labeled. Exercise suggestions are practical and adaptive.' },
      { file: 'AGENTS.md', desc: 'Consumption logging rules, calorie estimation guidelines, Dashboard field mappings, workout programming methodology, progressive overload tracking.' },
      { file: 'TOOLS.md', desc: 'york-data functions for all health domains. Exercise library reference. No direct spreadsheet access.' },
      { file: 'IDENTITY.md', desc: 'Wynn 🦌 — Old English for "joy/wellness."' },
      { file: 'MEMORY.md', desc: 'Calorie estimation reference, common meal shortcuts, James\'s exercise history, equipment availability, workout preferences.' },
      { file: 'memory/', desc: 'Daily logs, workout history context, nutrition caches.' },
    ],
    channels: [
      'Writes all health/fitness data via york-data',
      'Writes nutrition and workout caches to shared memory for morning brief',
    ],
    skillSections: [
      {
        name: 'Log',
        skills: [
          { name: 'Log Consumption', desc: 'Parse food/drink descriptions, estimate calories and protein, write to york-data via log_food(). Handle corrections via update_consumption() and delete_consumption().' },
          { name: 'Log Daily Metrics', desc: 'Write weight, wake time, bedtime, sleep, energy, kitchen closed to york-data via log_daily(). Upsert pattern — partial updates are fine.' },
          { name: 'Log Workout', desc: 'Record workout sessions and individual exercises with sets, reps, weight via log_workout() and log_exercises().' },
          { name: 'Log Cannabis', desc: 'Record cannabis sessions with time via log_cannabis().' },
        ],
      },
      {
        name: 'Coach',
        skills: [
          { name: 'Workout Programming', desc: 'Suggest exercises based on equipment availability, recent workout history, and progressive overload goals. Adapt to schedule and energy. May spawn Opus sub-agent for complex program design.' },
          { name: 'Calorie Estimation', desc: 'Reference data for estimating calories and protein from natural language food descriptions. Restaurant lookup when possible.' },
        ],
        tbd: true,
      },
    ],
    spawns: [
      'Opus sub-agent (for complex workout programming or deep nutrition analysis)',
    ],
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
.border-sonnet { border-color: #58a6ff; border-width: 2px; }
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

.skill-category {
  margin-bottom: 1.25rem;
}

.skill-category:last-child {
  margin-bottom: 0;
}

.skill-cat-header {
  font-size: 0.8rem;
  color: #d29922;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  margin-bottom: 0.5rem;
  padding-bottom: 0.25rem;
  border-bottom: 1px solid #21262d;
}

.skill-items {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.skill-item {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 4px;
  padding: 0.65rem 0.75rem;
}

.skill-name {
  color: #d2a8ff;
  font-weight: 600;
  font-size: 0.85rem;
  margin-bottom: 0.2rem;
}

.skill-desc {
  color: #8b949e;
  font-size: 0.8rem;
  line-height: 1.4;
}

.skill-tbd {
  color: #484f58;
  font-size: 0.8rem;
  font-style: italic;
  padding: 0.4rem 0.75rem;
}

@media (max-width: 768px) {
  .agent-detail-grid { grid-template-columns: 1fr; }
  .ws-file { flex-direction: column; gap: 0.25rem; }
  .ws-file code { min-width: auto; }
}
</style>
