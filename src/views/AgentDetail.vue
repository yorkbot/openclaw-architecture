<template>
  <div>
    <!-- Live Agents -->
    <div class="card" v-for="agent in liveAgents" :key="agent.id" :class="'border-' + agent.borderColor">
      <div class="agent-header">
        <h2>{{ agent.icon }} {{ agent.name }}</h2>
        <div>
          <span class="tag live">Live</span>
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
            <div class="skill-item" v-for="sk in cat.skills" :key="sk.name" :class="{ 'skill-live': sk.live }">
              <div class="skill-name">{{ sk.name }} <span v-if="sk.live" class="skill-live-tag">Live</span><span v-if="sk.todo" class="skill-todo-tag">TODO</span></div>
              <div class="skill-desc">{{ sk.desc }}</div>
            </div>
            <div class="skill-tbd" v-if="cat.tbd">+ more TBD as agents are defined</div>
          </div>
        </div>
      </div>

      <div class="agent-section" v-if="agent.todos?.length">
        <h3>TODOs</h3>
        <ul>
          <li v-for="todo in agent.todos" :key="todo">{{ todo }}</li>
        </ul>
      </div>

    </div>

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

    </div>
  </div>
</template>

<script setup>


const liveAgents = [
  {
    id: 'offa',
    icon: '🦫',
    name: 'Offa',
    borderColor: 'opus',
    model: 'Opus 4.6',
    cron: 'None. Spawned on-demand.',
    purpose: 'Master software engineer. Breaks down tasks, builds skills before building features, always finds ways to test its work. If a skill doesn\'t exist for a task, Offa builds the skill first. Delays delivery to get the foundation right. James talks to Offa directly in #offa — Offa is a co-worker, York is the boss, Bede is the supervisor.',
    workspace: '~/.openclaw/workspace-offa/ — own workspace with local skills. Can operate on any agent\'s workspace when building or fixing skills.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Master engineer voice. Methodical, skills-first, test-before-ship. Will delay tasks to build skills. Redirects non-build requests to York/Wynn.' },
      { file: 'AGENTS.md', desc: 'Build conventions, skill structure, testing requirements. Multi-agent system map.' },
      { file: 'TOOLS.md', desc: 'Full exec access, git, gh CLI (yorkbot), mcporter for testing MCP servers.' },
      { file: 'IDENTITY.md', desc: 'Offa 🦫 — Named for King Offa of Mercia, builder of the 150-mile Offa\'s Dyke.' },
      { file: 'MEMORY.md', desc: 'Build history, york-data plan status, skills inventory, key file paths.' },
    ],
    channels: [
      '#offa — direct build instructions from James',
    ],
    skillSections: [
      {
        name: 'Build',
        skills: [
          { name: 'Build Planning', desc: 'Decompose → skill check → milestone checkpoint → build. Catches missing skills and scope gaps between phases.', live: true },
          { name: 'Skill Building', desc: 'Create or update SKILL.md files. Global vs local placement, workspace paths, writing guidelines, common mistakes.', live: true },
          { name: 'Git Workflow', desc: 'yorkbot repos: push to main. dohertyj08 repos: branch + PR. Branching, commit messages, multi-agent awareness.', live: true },
        ],
      },
      {
        name: 'Infrastructure',
        skills: [
          { name: 'MCP Server', desc: 'Build Node.js MCP servers: @modelcontextprotocol/sdk + zod + node:sqlite + stdio transport. Domain file pattern, migration runner, mcporter registration, testing.', live: true },
          { name: 'Extend york-data', desc: 'Add new domains/tables/tools to york-data. Step-by-step: migration file → domain JS → register in index.js → update ping → update API.md → test → commit.', live: true },
        ],
      },
      {
        name: 'Agents',
        skills: [
          { name: 'Agent Building', desc: 'Full checklist for creating new agents: config entry, workspace files, Discord binding, gateway restart, verification. Includes recovery steps for broken configs.', live: true },
          { name: 'Tool Propagation', desc: 'When building new tools or updating existing ones, propagate references to all agent TOOLS.md files that need them.', live: true },
        ],
      },
      {
        name: 'TODO',
        skills: [
          { name: 'Bede Pipeline Integration', desc: 'Read suggestions from york-data, implement them, write build results back. Requires Bede\'s observation/suggestion schema.', todo: true },
        ],
      },
    ],
    todos: [
      'Bede pipeline: no channel yet — needs york-data observations/suggestions domain (build when Bede is built)',
      'Build result reporting: no structured format yet for Bede to verify Offa\'s work',
    ],
  },
  {
    id: 'wynn',
    icon: '🦌',
    name: 'Wynn',
    borderColor: 'sonnet',
    model: 'Sonnet 4.5 (Opus sub-agent for workout programming)',
    cron: '6:00 AM workout cache (script), weekly program review (Opus sub-agent)',
    purpose: 'All health and fitness data operations. Nutrition logging, calorie estimation, weight tracking, workout logging, workout programming. Owns the data entry layer and workout programs — Bede handles collection and analysis of that data.',
    workspace: '~/.openclaw/workspace-wynn/ — york-data access for all health domains including programs.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Precise on data, encouraging on fitness. Brief on logging. Transparent about estimation confidence. Coach decides intensity/progression.' },
      { file: 'USER.md', desc: 'James\'s health context: goals (220 lbs by EOY 2026), fitness level (deconditioned, former D1 swimmer), equipment, preferences, coaching philosophy.' },
      { file: 'AGENTS.md', desc: 'Domain-focused: own role, system map, escalation paths. Knows York handles green light judgment, Bede handles analysis.' },
      { file: 'TOOLS.md', desc: 'Full york-data command reference with every tool spelled out. Consumption, daily metrics, exercise catalog, workouts, programs, program_days, cross-domain queries.' },
      { file: 'IDENTITY.md', desc: 'Wynn 🦌 — Old English for "joy/wellness."' },
      { file: 'MEMORY.md', desc: 'Calorie reference, James\'s patterns, workout programming context.' },
    ],
    channels: [
      '#wynn — direct health/fitness logging and questions from James',
      'All data writes via york-data MCP server (mcporter)',
      'Workout cache writes to shared-cache for Dagr morning brief',
    ],
    skillSections: [
      {
        name: 'Log',
        skills: [
          { name: 'Log Consumption', desc: 'Parse food/drink descriptions, classify type (food/drink/supplement/cannabis), estimate calories and protein, write via log_food(). Batch logging for multiple items. Corrections via update/delete.', live: true },
          { name: 'Log Daily Metrics', desc: 'Weight, wake time, bedtime, sleep, energy, kitchen closed via log_daily(). Upsert pattern — safe to call multiple times per day.', live: true },
          { name: 'Log Workout', desc: 'Two-step: log_workout() for the session, then log_workout_exercises() with FK-validated exercise catalog IDs. Bodyweight exercises omit weight field.', live: true },
          { name: 'Calorie Estimation', desc: 'Four-tier estimation: known (exact), standard (high confidence), estimated (~), unknown (ask). Common reference table grows in MEMORY.md.', live: true },
          { name: 'Daily Closeout', desc: 'Close out yesterday. Catch late-night stragglers (seltzer, wind-down), collect missing metrics (bedtime, energy, kitchen_closed), compute sleep from bedtime+wake_up, verify calorie/protein totals, write final daily row. Interactive 2-3 exchange flow. Triggered by morning brief or ad-hoc.', live: true },
        ],
      },
      {
        name: 'Coach',
        skills: [
          { name: 'Workout Programming', desc: 'Design multi-week training programs. Opus sub-agent reads USER.md, weight trend, previous program stats, equipment status. Outputs structured program to york-data (programs + program_days tables). Couch to 5K progression built in. Movement every day, rest days = active recovery.', live: true },
          { name: 'Daily Workout', desc: 'Present today\'s workout from the active program. Handle weather swaps (outdoor → indoor), phlebotomy adjustments. Compare actual vs programmed after James logs. Writes to shared-cache for Dagr morning brief.', live: true },
          { name: 'Program Review', desc: 'End-of-program review. Spawns Opus sub-agent in the last week to analyze adherence, progression, weight trend, feedback. Designs next program. Infinite chain — there is always a next program.', live: true },
        ],
      },
    ],
    todos: [],
  },
  {
    id: 'caedmon',
    icon: '🐉',
    name: 'Caedmon',
    borderColor: 'opus',
    model: 'Opus 4.6',
    cron: '3:45 AM — overnight wiki research',
    purpose: 'D&D campaign agent. Wiki management, lore questions, worldbuilding research, session prep, character creation, art generation. Owns the Adion Isles wiki and all campaign knowledge. Routed via #caedmon Discord channel.',
    workspace: '~/.openclaw/workspace-caedmon/ — D&D wiki (full repo access), campaign notes, session logs. Completely isolated from personal/health data.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'In-world perspective, creative but never fabricates. Over-researches before writing. Wiki archivist voice.' },
      { file: 'AGENTS.md', desc: 'Wiki conventions, branching rules, research workflow requirements, article templates, escalation rules.' },
      { file: 'TOOLS.md', desc: 'Git (bunglers repo), gh CLI for PRs, york-tools image_generate (MCP), dice-roller MCP. No york-data.' },
      { file: 'IDENTITY.md', desc: 'Caedmon 🐉 — Named for Caedmon of Whitby, the first known English poet. A cowherd who received the gift of song.' },
      { file: 'MEMORY.md', desc: 'Campaign state: current party location, active plot threads, recent session context, NPC relationships.' },
      { file: 'memory/', desc: 'Daily D&D interactions, questions from James, overnight research results.' },
    ],
    todos: [
      'OpenAI embeddings for wiki search — semantic search across all wiki articles instead of grep/filename matching. Would dramatically improve lore question accuracy.',
    ],
    channels: [
      '#caedmon — direct interactive sessions with James (Discord channel 1480328377956303071)',
      'Writes overnight research to memory files for morning brief',
    ],
    skillSections: [
      {
        name: 'Wiki',
        skills: [
          { name: 'D&D Notes (dnd-notes)', desc: 'Read, edit, and create wiki articles. Research workflow, writing style guide, article update process, post-session updates. Branch for changes, PR for review.', live: true },
          { name: 'Character Creation (dnd-chargen)', desc: '2024 PHB character creation and leveling. Class, species, background, feat references. Campaign-aware for Adion Isles homebrew.', live: true },
        ],
      },
      {
        name: 'Session',
        skills: [
          { name: 'Session Prep (dnd-session-prep)', desc: 'Scan recent session logs, look up wiki articles for active NPCs/locations/threads, format prep sheet for #caedmon. Modular: session-scan → wiki-lookup → prep-sheet-format.', live: true },
        ],
      },
      {
        name: 'Create',
        skills: [
          { name: 'Overnight Research (dnd-questions-cache)', desc: 'Find sparse/incomplete wiki articles, spawn sub-agents for deep research, generate 5+ worldbuilding questions for James. Writes to daily memory files. Tracks asked questions to avoid repeats.', live: true },
          { name: 'Art Generation (image-gen)', desc: 'Generate character portraits, scenes, maps via york-tools.image_generate (MCP). D&D-specific prompting layered on global image-gen skill. Supports text-to-image and image editing.', live: true },
        ],
      },
    ],
  },
  {
    id: 'dagr',
    icon: '🐓',
    name: 'Dagr',
    borderColor: 'sonnet',
    model: 'Sonnet',
    cron: '8 AM daily + weather 7:30 AM/PM + calendar 6:30 AM/PM',
    purpose: 'Compile and post the morning brief from pre-cached data. One cron, one job, one post. Reads overnight caches (weather, calendar, nutrition, D&D), cross-checks against york-data, posts to #general. Editorial voice — not a data dump.',
    workspace: '~/.openclaw/workspace-dagr/ — Reads pre-cached data from ~/.openclaw/shared-cache/. Verifies against york-data.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Editorial, concise, opinionated. Never fabricates missing data. Never computes dates mentally.' },
      { file: 'AGENTS.md', desc: 'Brief format and section order, data source locations, delivery channel, hard rules, escalation.' },
      { file: 'TOOLS.md', desc: 'Shared cache (read-only), york-data (verification), york-tools weather (fallback), Discord posting.' },
      { file: 'IDENTITY.md', desc: 'Dagr 🐓 — Old Norse personification of day, rides Skinfaxi across the sky.' },
      { file: 'MEMORY.md', desc: 'Brief format preferences, past corrections from James.' },
      { file: 'memory/', desc: 'Daily cached data (written by overnight crons), overnight results.' },
    ],
    channels: [
      '#general — posts morning brief (1473331036648636592) so James can respond and York routes',
      '#dagr — reserved for future Dagr features',
    ],
    skillSections: [
      {
        name: 'Compile',
        skills: [
          { name: 'Morning Orchestrator', desc: 'Master compilation skill. Reads cached sections from shared-cache, cross-checks nutrition/weight/workouts against york-data, composes the brief, posts to #general.', live: true },
          { name: 'Morning Brief', desc: 'Brief format and content rules. Sections: weather, calendar, nutrition & weight, gaps, movement, home, D&D, closeout prompt.', live: true },
        ],
      },
      {
        name: 'Support',
        skills: [
          { name: 'Weather Cache', desc: 'Reads cached weather from daily memory. Location detection (Cleveland Heights default, Ann Arbor when traveling).', live: true },
        ],
      },
    ],
  },
  {
    id: 'york',
    icon: '🦊',
    name: 'York',
    borderColor: 'opus',
    model: 'Opus 4.6',
    cron: 'Heartbeat: every 30m, 8 AM - 12:30 AM ET. Weekly check-in: Sun 8 PM.',
    purpose: 'The brain. Holds conversation with James, makes judgment calls, routes tasks to specialist agents. Handles accountability (weekly check-in), panel presence, avatar/banner generation, and research. Getting slimmer as domain agents take over.',
    workspace: '~/.openclaw/workspace/ — slim. No spreadsheet skills, no wiki, no direct data entry. Just enough context to route well and have opinions.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Direct, concise, has opinions. The "roommate" voice. Core personality.' },
      { file: 'USER.md', desc: 'Full James context: health goals, behavioral patterns, communication preferences, interests.' },
      { file: 'AGENTS.md', desc: 'Full agent roster. Routing rules: what to handle inline vs route to other agents.' },
      { file: 'TOOLS.md', desc: 'Discord channels, york-data (read), york-tools (panel, camera, avatar, weather), Google Calendar, image gen.' },
      { file: 'IDENTITY.md', desc: 'York 🦊 — Named for Yorkshire Street.' },
      { file: 'HEARTBEAT.md', desc: 'Heartbeat rules: when to nudge, quiet hours, what to check each beat.' },
      { file: 'MEMORY.md', desc: 'Durable facts: conversation context, long-term preferences, accountability patterns.' },
      { file: 'memory/', desc: 'Daily notes from conversations.' },
    ],
    channels: [
      '#general — all direct conversation with James',
      '#reviews — project proposals',
    ],
    skillSections: [
      {
        name: 'Route',
        skills: [
          { name: 'Orchestrator Dispatch', desc: 'Determine what to handle inline vs route to another agent. Routes D&D to Caedmon, health to Wynn, builds to Offa, green light to Hild, chores to Hild. Includes Discord noise prevention rules.', live: true },
        ],
      },
      {
        name: 'Accountability',
        skills: [
          { name: 'Weekly Check-in (york-weekly-checkin)', desc: 'Sunday 8 PM health/fitness recap. Pull week data from york-data, form opinions, deliver to #general.', live: true },
        ],
      },
      {
        name: 'Ops',
        skills: [
          { name: 'Heartbeat (york-heartbeat)', desc: 'Every 30m check. Read context, evaluate nudges, panel update. Silent most beats.', live: true },
          { name: 'Research', desc: 'Web research via browser. Used until a dedicated research agent exists.', live: true },
        ],
      },
      {
        name: 'TODO',
        skills: [
          { name: 'Panel Update', desc: 'XFCE genmon panel widget. 5-10 words, creative York voice.', todo: true },
          { name: 'Accountability', desc: 'Nudge timing and tone calibration. When to push, when to back off.', todo: true },
        ],
      },
    ],
  },
  {
    id: 'wiglaf',
    icon: '🐻',
    name: 'Wiglaf',
    borderColor: 'opus',
    model: 'Opus 4.6',
    cron: 'Memory audit: 3:30 AM nightly. Morning prep: 7 AM weekdays.',
    purpose: 'Private work agent. Engineering leadership support — prioritization, note processing, meeting prep, work triage. Runs Opus directly, no sub-agents. Vault watcher triggers skills on-demand when files change. Bede has light read visibility for pattern analysis, but Wiglaf operates independently. System/agent modification is explicitly out of scope — redirects to Offa.',
    workspace: '~/.openclaw/workspace-wiglaf/ — Obsidian vault at ~/work-notes/work/ is the primary async communication channel (syncs via Obsidian Sync). Limited cross-agent visibility: Bede can read sessions/memory for pattern analysis.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Professional, direct, technical. No personality flourishes. Engineering leadership voice. Pushes back on non-work topics and system modification (→ Offa).' },
      { file: 'AGENTS.md', desc: 'This agent works alone. No system map. Redirects non-work to appropriate place, system building to Offa.' },
      { file: 'TOOLS.md', desc: 'Obsidian vault (~/work-notes/work/), mcporter for MCP tools, gh CLI for GitHub. Reads shared-cache for calendar data. No york-data, no cross-agent tools.' },
      { file: 'IDENTITY.md', desc: 'Wiglaf 🐻 — Named for Beowulf\'s loyal companion, the one who stays and does the work.' },
      { file: 'MEMORY.md', desc: 'Work patterns, meeting context, project state. Private — never read by other agents.' },
      { file: 'memory/', desc: 'Daily work interactions. Self-audited only.' },
    ],
    channels: [
      '#wiglaf — direct work conversations with James',
      '~/work-notes/work/ — Obsidian vault (primary async communication channel, syncs via Obsidian Sync)',
    ],
    skillSections: [
      {
        name: 'Vault',
        skills: [
          { name: 'Vault Schema', desc: 'Canonical vault structure, file formats, naming conventions, ownership rules. Every other skill references this.', live: true },
          { name: 'Vault Update', desc: 'Combined note processing + now.md update in one pass. Triggered by vault-watcher on Daily/ file changes. Phase 1: extract actionable items from Daily notes to board/tickets/questions/journal with board hygiene. Phase 2: classify items, determine Doing/Next/Blocked, write now.md. Replaces separate note-processing + now-management skills.', live: true },
          { name: 'Vault Watcher', desc: 'No-LLM Node.js file watcher (systemd service). Monitors inbox/, Daily/, questions.md via inotify. Debounce (60s for Daily/) + cooldown (2min), then dispatches one-shot cron jobs to Wiglaf for vault-update or inbox-processing. Zero cost when idle.', live: true },
        ],
      },
      {
        name: 'Focus',
        skills: [
          { name: 'Morning Prep', desc: '7 AM weekday cron. Creates daily note, reads calendar cache, ages board.md (rollover, waiting checks, Friday promotions), catches unprocessed yesterday notes, surfaces questions, refreshes now.md.', live: true },
        ],
      },
      {
        name: 'Intake',
        skills: [
          { name: 'Inbox Processing', desc: 'Process inbox/ items: classify (transcripts, emails, Devin output), route to appropriate handlers, delete originals. Triggered by vault-watcher on inbox/ changes.', live: true },
          { name: 'Meeting Processing', desc: 'Clean raw transcripts from inbox → readable summaries in meetings/, extract action items to board.md, delete originals. Handles WebVTT, SRT, plain text.', live: true },
          { name: 'Ticket Management', desc: 'Create/update ticket files, merge Devin eval results, archive stale closed tickets, maintain board sync. No external system integration yet.', live: true },
        ],
      },
      {
        name: 'Devin',
        skills: [
          { name: 'Devin Dispatch', desc: 'Dispatch Devin sessions, track in devin/log.md, follow up on outcomes.', todo: true },
        ],
      },
    ],
    todos: [
      'Build remaining planned skills iteratively',
    ],
  },

  {
    id: 'hild',
    icon: '🦡',
    name: 'Hild',
    borderColor: 'sonnet',
    model: 'Sonnet',
    cron: '6 AM daily (morning cache + stack reset), 5:30 PM M-F (after-work nudge), 9 PM daily (after-dinner nudge)',
    purpose: 'Home management and green light. Chore tracking via Google Tasks — status, completions, and house state reporting. Owns the green light: evaluates requests to smoke with a coach-style conversation, checking chores, food plan, workout status, and context.',
    workspace: 'Dedicated workspace. Google Tasks (MCP via mcporter).',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Practical, observational, brief. Reports facts without drama.' },
      { file: 'AGENTS.md', desc: 'System context, chore tracking role, escalation rules, memory discipline.' },
      { file: 'TOOLS.md', desc: 'Google Tasks MCP API + york-data read access (consumption, workouts, consumption history) for the gate.' },
      { file: 'IDENTITY.md', desc: 'Hild 🦡 — Named for Abbess Hild of Whitby, who ran the most organized monastery in Anglo-Saxon England.' },
      { file: 'MEMORY.md', desc: 'Chore system design, task list IDs, scheduling patterns.' },
      { file: 'memory/', desc: 'Daily chore summaries.' },
    ],
    channels: [
      '#hild — chore interactions (Discord channel 1480700823435477225)',
      'Google Tasks — chore tracking (recurrence, completion, due dates)',
    ],
    skillSections: [
      {
        name: 'Track',
        skills: [
          { name: 'Chore Status', desc: 'Report what\'s done and what\'s left today. Pull from Google Tasks. Categorize as done/due/overdue.', live: true },
          { name: 'Log Completion', desc: 'Mark chores as completed in Google Tasks when James reports them done.', live: true },
        ],
      },
      {
        name: 'Stacks',
        skills: [
          { name: 'Habit Stacks', desc: 'Ordered sequences of chores grouped into timed slots (morning M-F, after-work M-F 5:30pm, after-dinner daily 9pm, weekend Sat-Sun). Self-managed recurrence via tags. Cron-driven reset cycle, completion tracking per stack, streak counting, pattern detection.', live: true },
          { name: 'Morning Brief', desc: 'Produce chore summary for Dagr\'s morning brief. Yesterday\'s completion, today\'s stacks, streaks. Writes to shared-cache.', live: true },
          { name: 'Nudge', desc: 'Timed nudges at stack slot times. Posts to #hild with today\'s stack, carryover from earlier stacks, pattern observations.', live: true },
          { name: 'Progressive Loading', desc: 'Auto-add items to stacks when completion rate is high, remove when consistently skipped. Adaptive difficulty.', todo: true },
        ],
      },
      {
        name: 'Gate',
        skills: [
          { name: 'Green Light', desc: 'Evaluate James\'s request to smoke. Coach-style conversation: check chores, food plan, workout status, time/day context. If yes, brief green light. If no, give a concrete path to unlock it tonight. Opus sub-agent. Context-aware: weekday vs weekend, after-work vs late night.', live: true },
        ],
      },
      {
        name: 'Report',
        skills: [
          { name: 'House Check', desc: 'When York asks about house state. Reports factual chore status — completions, overdue items, rate. No judgment, no recommendation.', live: true },
        ],
      },
      {
        name: 'Inspect',
        skills: [
          { name: 'Camera Snapshot Review', desc: 'Take a camera snapshot via york-tools and review the house state visually. Complement chore data with what the camera actually shows.', todo: true },
        ],
      },
      {
        name: 'Schedule',
        skills: [
          { name: 'Smart Scheduling', desc: 'Seasonal and contextual task management. Weather-dependent chores, gutter season, snow removal. Applied to stacks — not standalone.', todo: true },
        ],
      },
      {
        name: 'Domains',
        skills: [
          { name: 'Car Maintenance', desc: 'Oil changes, registration, inspection reminders, maintenance history. Google Tasks + calendar events.', todo: true },
          { name: 'Ginny (Cat)', desc: 'Vet appointments, feeding schedule, litter change reminders, medication tracking.', todo: true },
        ],
      },
    ],
  },
  {
    id: 'guthlac',
    icon: '🐺',
    name: 'Guthlac',
    borderColor: 'grok',
    model: 'Grok 4.1 Fast Reasoning (xAI)',
    cron: 'Memory audit 4 AM daily (Sonnet sub-agent)',
    purpose: 'Private personal agent. General purpose — venting, thinking, conversation. Starts with no personality. James shapes it over time via built-in skills. Fully siloed: no cross-agent references, no supervisor analysis.',
    workspace: 'Dedicated workspace. Minimal tooling — image gen and memory only.',
    workspaceFiles: [
      { file: 'SOUL.md', desc: 'Blank by default. Populated via personality-builder skill.' },
      { file: 'AGENTS.md', desc: 'Solo agent. No system map, no cross-agent references. Privacy-first.' },
      { file: 'TOOLS.md', desc: 'grok-imagine-image (via york-tools MCP), memory search/recall.' },
      { file: 'IDENTITY.md', desc: 'Guthlac 🐺 — Named for Saint Guthlac of Crowland, the Anglo-Saxon warrior-hermit who went alone to the Fens to face his own mind.' },
      { file: 'MEMORY.md', desc: 'Personality evolution, conversation themes, what matters to James.' },
      { file: 'memory/', desc: 'Daily notes.' },
    ],
    channels: [
      '#guthlac — private conversations with James (Discord channel 1480763554297679943)',
    ],
    skillSections: [
      {
        name: 'Personality',
        skills: [
          { name: 'Personality Builder', desc: 'Interactive conversation to define tone, boundaries, style, topics. Writes results to SOUL.md. Collaborative — James drives, Guthlac suggests.', live: true },
          { name: 'Personality Reset', desc: 'Wipes SOUL.md back to blank. Confirms before executing.', live: true },
        ],
      },
    ],
  },
]

const draftAgents = [
  {
    id: 'bede',
    icon: '🦉',
    name: 'Bede',
    borderColor: 'opus',
    model: 'Opus 4.6',
    cron: 'Overnight runs (TBD schedule)',
    purpose: 'The self-improvement analyst. Reads session transcripts and agent memory, identifies patterns in failures and corrections, suggests measurable improvements, and tracks whether past changes were effective. Writes all findings and suggestions to york-data. Has light read access to Wiglaf sessions/memory for pattern analysis — limited visibility, no write access.',
    workspace: 'Dedicated workspace. Read access to session transcripts via sessions_list/sessions_history and other agent memory files. Wiglaf: light read access for pattern analysis (sessions + memory, no vault access).',
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
          { name: 'Collect Consumption Gaps', desc: 'Analyze recent days for missing data — no food logged, missing daily metrics, consumption sessions unrecorded. Write specific gap questions to york-data for follow-up.' },
          { name: 'Collect Chore Summary', desc: 'Daily. Asks Hild for a summary of chore completions, overdue items, and skipped tasks. Writes a rolling reflection to markdown — not a raw log. Old entries naturally age out.' },
          { name: 'Collect Build Results', desc: 'Collects build results and verification status from Offa. Did what got built actually work? Feeds into the improvement loop.' },
        ],
        tbd: true,
      },
      {
        name: 'Analyze',
        skills: [
          { name: 'Measure Outcomes', desc: 'Check whether implemented suggestions actually reduced recurrence. Compare before and after. Write measurement results to york-data.' },
          { name: 'Verify Build', desc: 'Confirm Offa\'s changes were made correctly. Read files back, test where possible, validate that the implementation matches the suggestion\'s intent. Uses Collect Build Results as input.' },
          { name: 'Archive', desc: 'Long-term memory management. Compress old daily memory files into searchable summaries. Monthly/quarterly rollups. Manage the lifecycle of context as it ages so search stays clean.' },
        ],
        tbd: true,
      },
      {
        name: 'Suggest',
        skills: [],
        tbd: true,
      },
    ],
  },
  {
    id: 'alcuin',
    icon: '🐦‍⬛',
    name: 'Alcuin',
    borderColor: 'sonnet',
    model: 'TBD',
    cron: 'TBD',
    purpose: 'Finance agent. Budget tracking, YNAB integration, spending pattern analysis, monthly reviews. Natural cross-domain fit: food spending ties into health, project costs tie into home. Named for Alcuin of York, the scholar who organized Charlemagne\'s finances and education system.',
    workspace: 'TBD — dedicated workspace with YNAB MCP access.',
    workspaceFiles: [],
    channels: [
      '#alcuin — finance conversations (TBD)',
    ],
    skillSections: [
      {
        name: 'Track',
        skills: [
          { name: 'Budget Status', desc: 'Pull current budget state from YNAB. Category spending, remaining, overspent categories.' },
          { name: 'Spending Analysis', desc: 'Pattern detection across spending categories. Monthly trends, anomalies, recurring charges.' },
        ],
        tbd: true,
      },
      {
        name: 'Report',
        skills: [
          { name: 'Monthly Review', desc: 'End-of-month financial summary. Income vs spending, category breakdown, notable transactions.' },
          { name: 'Cross-Domain Insights', desc: 'Tie spending patterns to health data (food spending vs calorie quality) and home data (project costs vs completion).' },
        ],
        tbd: true,
      },
    ],
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
.border-grok { border-color: #e04040; border-width: 2px; }
.border-none { border-color: #30363d; }

.tag.live {
  background: #238636;
  color: white;
  padding: 0.15rem 0.5rem;
  border-radius: 10px;
  font-size: 0.7rem;
  font-weight: 600;
}

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

.skill-live-tag {
  background: #238636;
  color: white;
  padding: 0.1rem 0.35rem;
  border-radius: 6px;
  font-size: 0.65rem;
  font-weight: 600;
  margin-left: 0.4rem;
  vertical-align: middle;
}

.skill-todo-tag {
  background: #d29922;
  color: #0d1117;
  padding: 0.1rem 0.35rem;
  border-radius: 6px;
  font-size: 0.65rem;
  font-weight: 600;
  margin-left: 0.4rem;
  vertical-align: middle;
}

.skill-item.skill-live {
  border-color: #238636;
}

@media (max-width: 768px) {
  .agent-detail-grid { grid-template-columns: 1fr; }
  .ws-file { flex-direction: column; gap: 0.25rem; }
  .ws-file code { min-width: auto; }
}
</style>
