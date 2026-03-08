<template>
  <div>
    <div class="card">
      <h2>Architecture: York as Silent Brain</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        York is the orchestrator. It does not talk to James directly. Specialized conversational agents
        handle all interaction. York makes decisions, maintains state, and routes work.
      </p>
    </div>

    <div class="card border-gold">
      <h3>The Model</h3>
      <div class="arch-diagram">
        <div class="arch-layer">
          <div class="arch-label">James</div>
          <div class="arch-desc">Discord messages in #general, #dnd, etc.</div>
        </div>
        <div class="arch-arrow">↕</div>
        <div class="arch-layer highlight">
          <div class="arch-label">Conversational Agents</div>
          <div class="arch-desc">
            One or more agents with personality, voice, and tone.
            Handle all user-facing interaction. Read daily state from york-data on activation.
            Escalate decisions to York when needed.
          </div>
        </div>
        <div class="arch-arrow">↕</div>
        <div class="arch-layer brain">
          <div class="arch-label">🦝 York (Silent Brain)</div>
          <div class="arch-desc">
            Orchestrator. Runs on heartbeat. Reads york-data, evaluates state, makes routing decisions,
            spawns specialists. Writes daily state snapshot every heartbeat. Never posts to Discord.
          </div>
        </div>
        <div class="arch-arrow">↕</div>
        <div class="arch-layer">
          <div class="arch-label">york-data</div>
          <div class="arch-desc">
            Central data layer. Daily state, observations, suggestions, consumption, metrics, chores.
            Every agent reads from here. Structured, typed, reliable.
          </div>
        </div>
        <div class="arch-arrow">↕</div>
        <div class="arch-layer">
          <div class="arch-label">Specialist Agents</div>
          <div class="arch-desc">
            Bede (analysis), Offa (building), Health, Chores, Lore, etc.
            Spawned on-demand or via cron. Read and write york-data. No direct user interaction.
          </div>
        </div>
      </div>
    </div>

    <div class="card">
      <h3>How Conversational Agents Get Context</h3>
      <p style="color: #c9d1d9; line-height: 1.6; margin-bottom: 1rem; font-size: 0.9rem;">
        Conversational agents don't ask York for context. They pull it themselves from york-data on every activation.
      </p>
      <div class="context-items">
        <div class="ctx-item" v-for="ctx in contextSources" :key="ctx.source">
          <div class="ctx-source"><code>{{ ctx.source }}</code></div>
          <div class="ctx-what">{{ ctx.what }}</div>
          <div class="ctx-who">Written by: {{ ctx.writtenBy }}</div>
        </div>
      </div>
    </div>

    <div class="card">
      <h3>When Does a Conversational Agent Escalate to York?</h3>
      <div class="escalation-list">
        <div class="esc-item" v-for="e in escalations" :key="e.trigger">
          <div class="esc-trigger">{{ e.trigger }}</div>
          <div class="esc-why">{{ e.why }}</div>
        </div>
      </div>
      <p style="color: #8b949e; font-size: 0.85rem; margin-top: 1rem;">
        Everything else — responding to questions, chatting, delivering information — the conversational agent handles directly using york-data context. No round-trip needed.
      </p>
    </div>

    <div class="card">
      <h3>The Self-Improvement Loop</h3>
      <div class="loop-diagram">
        <div class="loop-step" v-for="(step, i) in loopSteps" :key="i">
          <div class="loop-num">{{ i + 1 }}</div>
          <div class="loop-content">
            <div class="loop-agent">
              <span class="loop-icon">{{ step.icon }}</span>
              <span class="loop-name">{{ step.agent }}</span>
            </div>
            <div class="loop-skill">{{ step.skill }}</div>
            <div class="loop-action">{{ step.action }}</div>
          </div>
        </div>
        <div class="loop-return">↩ Back to Collect — Measure results feed the next cycle</div>
      </div>
    </div>

    <div class="card">
      <h3>Open Questions</h3>
      <div class="question-list">
        <div class="question" v-for="q in questions" :key="q">❓ {{ q }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
const contextSources = [
  { source: 'york-data.daily_state', what: 'Today\'s weight, meals logged, cannabis gate status, chore state, pending items, calendar events', writtenBy: 'York (every heartbeat)' },
  { source: 'york-data.observations', what: 'Bede\'s structured findings — patterns, failures, recurring issues', writtenBy: 'Bede (overnight)' },
  { source: 'york-data.suggestions', what: 'Bede\'s actionable improvement proposals with measurability criteria', writtenBy: 'Bede (overnight)' },
  { source: 'york-data.consumption', what: 'Today\'s food log, calorie/protein totals', writtenBy: 'Health agent' },
  { source: 'agent memory files', what: 'Conversation history, daily context, long-term patterns', writtenBy: 'Each agent writes its own' },
]

const escalations = [
  { trigger: 'Cannabis gate request', why: 'Requires cross-domain judgment: chores, calories, movement, time of day, patterns' },
  { trigger: 'Spawning a specialist', why: 'Conversational agent can\'t spawn other agents — York decides what to spawn and when' },
  { trigger: 'Decisions outside authority', why: 'Anything that changes system state, modifies agent behavior, or requires orchestration' },
]

const loopSteps = [
  { icon: '🦉', agent: 'Bede', skill: 'Collect', action: 'Gather data from session transcripts, agent memory, york-data (including past Measure results)' },
  { icon: '🦉', agent: 'Bede', skill: 'Analyze', action: 'Turn raw data into structured findings. Identify corrections, errors, retries, frustration signals.' },
  { icon: '🦉', agent: 'Bede', skill: 'Suggest', action: 'Propose specific changes with measurability criteria. Write to york-data.' },
  { icon: '🦫', agent: 'Offa', skill: 'Build', action: 'Read suggestion from york-data. Implement: new skills, edits, config changes, scripts.' },
  { icon: '🦫', agent: 'Offa', skill: 'Verify', action: 'Confirm changes are correct. Read back files, test where possible.' },
  { icon: '🦉', agent: 'Bede', skill: 'Measure', action: 'Check if implemented changes reduced recurrence. Write measurement results to york-data.' },
]

const questions = [
  'How many conversational agents? One for #general, one for #dnd? Or one that handles all channels?',
  'What model for conversational agents? This is the experimentation surface — can swap freely.',
  'Communication mechanism for escalation: sessions_send (synchronous wait) or york-data request queue (async)?',
  'Should conversational agents have their own personality, or all share York\'s voice?',
  'Does the conversational agent need USER.md, or does daily_state from york-data cover enough context?',
]
</script>

<style scoped>
.arch-diagram {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0;
}

.arch-layer {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1rem 1.5rem;
  width: 100%;
  max-width: 600px;
}

.arch-layer.highlight {
  border-color: #58a6ff;
  background: #0d1520;
}

.arch-layer.brain {
  border-color: #f0c040;
  border-width: 2px;
  background: #1a1810;
}

.arch-label {
  color: #f0f6fc;
  font-weight: 700;
  font-size: 1rem;
  margin-bottom: 0.25rem;
}

.arch-desc {
  color: #8b949e;
  font-size: 0.85rem;
  line-height: 1.5;
}

.arch-arrow {
  color: #484f58;
  font-size: 1.5rem;
  padding: 0.25rem 0;
}

.context-items {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.ctx-item {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 4px;
  padding: 0.75rem;
}

.ctx-source code {
  color: #d2a8ff;
  font-size: 0.85rem;
}

.ctx-what {
  color: #c9d1d9;
  font-size: 0.85rem;
  margin: 0.25rem 0;
}

.ctx-who {
  color: #8b949e;
  font-size: 0.75rem;
}

.escalation-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.esc-item {
  background: #0d1117;
  border: 1px solid #21262d;
  border-radius: 4px;
  padding: 0.75rem;
}

.esc-trigger {
  color: #f0f6fc;
  font-weight: 600;
  font-size: 0.9rem;
  margin-bottom: 0.25rem;
}

.esc-why {
  color: #8b949e;
  font-size: 0.85rem;
}

.loop-diagram {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.loop-step {
  display: flex;
  gap: 1rem;
  padding: 0.75rem 0;
  border-left: 2px solid #30363d;
  margin-left: 1rem;
  padding-left: 1rem;
}

.loop-num {
  background: #30363d;
  color: #f0f6fc;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.75rem;
  font-weight: 700;
  flex-shrink: 0;
}

.loop-content {
  flex: 1;
}

.loop-agent {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.25rem;
}

.loop-icon { font-size: 1.1rem; }

.loop-name {
  color: #f0f6fc;
  font-weight: 600;
  font-size: 0.9rem;
}

.loop-skill {
  color: #d29922;
  font-weight: 600;
  font-size: 0.85rem;
  margin-bottom: 0.25rem;
}

.loop-action {
  color: #8b949e;
  font-size: 0.85rem;
  line-height: 1.4;
}

.loop-return {
  color: #58a6ff;
  font-size: 0.85rem;
  margin-left: 1rem;
  padding-left: 1rem;
  padding-top: 0.75rem;
  font-style: italic;
}

.question-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.question {
  color: #c9d1d9;
  font-size: 0.85rem;
  padding: 0.5rem;
  background: #0d1117;
  border-radius: 4px;
  line-height: 1.5;
}

.border-gold { border-color: #f0c040; border-width: 2px; }
</style>
