<template>
  <div>
    <div class="card">
      <h2>Migration Plan</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Phased migration from monolithic York to multi-agent architecture.
        Each phase is independently valuable — no need to complete all phases.
      </p>
    </div>

    <div class="card" v-for="phase in phases" :key="phase.num" :class="'border-' + phase.color">
      <div class="phase-header">
        <div class="phase-badge" :class="'bg-' + phase.color">Phase {{ phase.num }}</div>
        <h3>{{ phase.title }}</h3>
        <span class="phase-effort">{{ phase.effort }}</span>
      </div>

      <p class="phase-desc">{{ phase.description }}</p>

      <div class="phase-section">
        <h4>Changes</h4>
        <ul>
          <li v-for="change in phase.changes" :key="change">{{ change }}</li>
        </ul>
      </div>

      <div class="phase-section" v-if="phase.risks?.length">
        <h4>Risks</h4>
        <ul class="risks">
          <li v-for="risk in phase.risks" :key="risk">{{ risk }}</li>
        </ul>
      </div>

      <div class="phase-section">
        <h4>Validation</h4>
        <p>{{ phase.validation }}</p>
      </div>
    </div>

    <div class="card">
      <h3>What NOT to Migrate</h3>
      <ul class="dont">
        <li><strong>Cannabis gate judgment stays in York.</strong> The gate decision requires cross-domain awareness (chores + calories + time + patterns). Agents gather data; York decides.</li>
        <li><strong>Panel presence stays in York's heartbeat.</strong> It's self-expression, tied to York's personality and session context.</li>
        <li><strong>Accountability tone stays in York.</strong> The pushback, the conversational nudges, the weekly review opinions — these need the orchestrator's personality and the large model's nuance.</li>
        <li><strong>Don't over-split.</strong> If two things always run together and share context, they're one agent. Don't create agents for the sake of architecture.</li>
      </ul>
    </div>
  </div>
</template>

<script setup>
const phases = [
  {
    num: 1,
    title: 'Scripts Replace Trivial LLM Tasks',
    color: 'green',
    effort: '~2 hours',
    description: 'Replace LLM cron jobs with bash scripts for tasks that don\'t need reasoning. Immediate cost reduction, zero quality impact.',
    changes: [
      'Convert weather-cache cron from LLM agent to bash script (curl + sed + file write)',
      'Convert calendar-cache cron from LLM agent to bash script (mcporter call + jq + file write)',
      'Build weight-log script (mcporter call to append spreadsheet row)',
      'Build workout-log script (mcporter call to append spreadsheet row)',
      'Convert queue-audit to a bash script (pattern matching, no judgment needed)',
    ],
    risks: [
      'Scripts are brittle if data format changes. Add error handling.',
      'Calendar time zone conversion in bash is annoying. Use date command carefully.',
    ],
    validation: 'Run scripts manually, compare output to current LLM-generated versions. Should be identical.',
  },
  {
    num: 2,
    title: 'Sub-Agent Model Routing',
    color: 'blue',
    effort: '~1 hour',
    description: 'Keep the monolithic York agent but start passing model parameters to sessions_spawn based on task complexity. No structural changes, just smarter model selection.',
    changes: [
      'Update orchestrator-dispatch skill with model routing rules',
      'Consumption logging sub-agents spawn with model: "anthropic/claude-sonnet-4-6"',
      'Research sub-agents spawn with model based on task complexity (York decides)',
      'Builder tasks continue spawning with Opus',
      'Set up API keys for chosen providers (Anthropic API, OpenAI, xAI via OpenRouter)',
    ],
    risks: [
      'Sonnet may handle some consumption edge cases worse (odd food descriptions, unit conversions)',
      'Need to validate Sonnet follows skill files as reliably as Opus',
    ],
    validation: 'Run a week with mixed models. Compare sub-agent success rate and output quality. Check for any regressions in data accuracy.',
  },
  {
    num: 3,
    title: 'DnD Agent Separation',
    color: 'blue',
    effort: '~4 hours',
    description: 'Create a dedicated DnD agent with its own workspace and Discord channel. First real multi-agent split. Low risk because DnD is the most isolated domain.',
    changes: [
      'Create #dnd Discord channel',
      'Create DnD agent in agents.list with own workspace containing wiki, session logs, worldbuilding tools',
      'Write DnD SOUL.md (lore expert persona, knowledge of the campaign)',
      'Configure Discord routing: #dnd messages → DnD agent',
      'Move DnD cron (overnight research) to target DnD agent',
      'York keeps ability to spawn DnD agent for ad-hoc lore questions from #general',
      'Morning brief reads DnD research output from shared memory files',
    ],
    risks: [
      'DnD conversations that reference personal context (schedule, mood) lose that context',
      'Need to handle "James starts in #general, topic drifts to DnD" gracefully',
    ],
    validation: 'Have a full DnD conversation in #dnd. Verify lore accuracy, response quality, and that overnight research still works. Verify morning brief still includes DnD questions.',
  },
  {
    num: 4,
    title: 'Health Agent Separation',
    color: 'yellow',
    effort: '~3 hours',
    description: 'Create a dedicated Health agent. Higher stakes because this is the main purpose of the system. Careful validation needed.',
    changes: [
      'Create #health Discord channel (private)',
      'Create Health agent in agents.list with own workspace containing spreadsheet skills, nutrition tools',
      'Write Health SOUL.md (data-focused, conservative on estimates)',
      'Move consumption-related crons to target Health agent',
      'York spawns Health agent for food logging, calorie checks, gate data',
      'Weekly review agent reads from same spreadsheet',
    ],
    risks: [
      'Data accuracy regression — this is the highest-stakes domain',
      'Cannabis gate flow becomes multi-agent (York → Chores + Health → York decides)',
      'Edge cases in food logging (ambiguous items, corrections) need testing',
    ],
    validation: 'Log a full day of consumption through the new agent. Compare spreadsheet output to what the current monolith produces. Run the cannabis gate flow end-to-end.',
  },
  {
    num: 5,
    title: 'Full Multi-Agent Architecture',
    color: 'red',
    effort: '~6 hours',
    description: 'Complete the separation. Slim down York\'s workspace to orchestrator-only. All remaining agents split out.',
    changes: [
      'Create remaining agents: Chores, Morning Brief, Overnight, Weekly Review, Builder, Research',
      'Slim York workspace: remove all skill files except orchestrator-dispatch and cannabis-gate',
      'Define cross-agent memory patterns (shared memory dir or mediated reads)',
      'Full cron migration: each cron targets its specific agent',
      'Update all skill files to reference their agent\'s workspace paths',
      'Test every flow end-to-end',
    ],
    risks: [
      'Complexity spike — many moving parts',
      'Cross-agent memory sharing patterns need to be solid before this phase',
      'Debugging issues becomes harder with distributed agents',
    ],
    validation: 'Run for a full week. Every morning brief, every gate check, every food log, every heartbeat. Compare to the monolith week\'s output quality.',
  },
  {
    num: 6,
    title: 'MTG Standalone Instance',
    color: 'yellow',
    effort: '~2 hours',
    description: 'Move MTG to a completely separate OpenClaw instance. On-demand start/stop.',
    changes: [
      'Set up separate OpenClaw install (separate config, workspace, credentials)',
      'Own Discord server or channel for MTG',
      'Startup/shutdown script for when James wants to brew/draft',
      'Zero cost when not running',
    ],
    risks: [
      'Minor operational overhead of managing two installs',
    ],
    validation: 'Start MTG instance, run a draft analysis, shut it down. Verify York instance is unaffected.',
  },
]
</script>

<style scoped>
.phase-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
  flex-wrap: wrap;
}

.phase-badge {
  padding: 0.25rem 0.75rem;
  border-radius: 12px;
  font-size: 0.75rem;
  font-weight: 700;
  color: white;
}

.bg-green { background: #238636; }
.bg-blue { background: #1f6feb; }
.bg-yellow { background: #9e6a03; }
.bg-red { background: #da3633; }

.phase-effort {
  color: #8b949e;
  font-size: 0.85rem;
  margin-left: auto;
}

.phase-desc {
  color: #c9d1d9;
  line-height: 1.6;
  margin-bottom: 1rem;
}

.phase-section {
  margin-bottom: 1rem;
}

.phase-section h4 {
  color: #8b949e;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.5rem;
}

.phase-section ul {
  list-style: none;
  padding: 0;
}

.phase-section li {
  padding: 0.3rem 0;
  font-size: 0.9rem;
  color: #c9d1d9;
}

.phase-section li::before {
  content: '→ ';
  color: #484f58;
}

.risks li::before {
  content: '⚠️ ';
}

.phase-section p {
  color: #c9d1d9;
  font-size: 0.9rem;
}

.border-green { border-color: #238636; }
.border-blue { border-color: #1f6feb; }
.border-yellow { border-color: #9e6a03; }
.border-red { border-color: #da3633; }

.dont {
  list-style: none;
  padding: 0;
}

.dont li {
  padding: 0.5rem 0;
  border-bottom: 1px solid #21262d;
  line-height: 1.5;
  font-size: 0.9rem;
}

.dont li:last-child {
  border-bottom: none;
}
</style>
