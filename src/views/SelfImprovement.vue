<template>
  <div>
    <div class="card">
      <h2>Self-Improvement System</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        If the orchestrator is good and self-improvement works, everything else follows.
        Skills get built, bugs get fixed, patterns get noticed, the system evolves.
        This isn't a feature — it's the core capability that makes everything else secondary.
      </p>
    </div>

    <!-- The Loop -->
    <div class="card">
      <h3>The Feedback Loop</h3>
      <p style="color: #8b949e; margin-bottom: 1.5rem;">
        Self-improvement is a continuous cycle, not a TODO list. Every agent participates in observation.
        Analysis finds patterns. Experiments test fixes. Verification confirms they worked.
      </p>
      <div class="loop">
        <div class="loop-step" v-for="(step, i) in loopSteps" :key="step.name">
          <div class="loop-number">{{ i + 1 }}</div>
          <div class="loop-content">
            <div class="loop-header">
              <span class="loop-name">{{ step.name }}</span>
              <span :class="['tag', step.tierClass]">{{ step.tier }}</span>
              <span :class="['tag', step.whenClass]" v-if="step.when">{{ step.when }}</span>
            </div>
            <div class="loop-desc">{{ step.desc }}</div>
            <div class="loop-examples" v-if="step.examples?.length">
              <div class="loop-example" v-for="ex in step.examples" :key="ex">{{ ex }}</div>
            </div>
          </div>
          <div class="loop-arrow" v-if="i < loopSteps.length - 1">↓</div>
        </div>
        <div class="loop-return">
          <span>↺ Cycle repeats — verification feeds back into observation</span>
        </div>
      </div>
    </div>

    <!-- Observation Detail -->
    <div class="card border-green">
      <h3>Observation Layer (Embedded in Every Agent)</h3>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        This is the foundation. Every agent writes friction events to a shared journal.
        Not a separate task — a reflex. Like distributed logging.
      </p>

      <div class="obs-grid">
        <div class="obs-card" v-for="obs in observations" :key="obs.agent">
          <div class="obs-agent">{{ obs.agent }}</div>
          <div class="obs-watches">
            <div class="obs-item" v-for="w in obs.watches" :key="w">{{ w }}</div>
          </div>
        </div>
      </div>

      <div class="obs-format">
        <h4>Journal Entry Format</h4>
        <div class="code-block">
          <div class="code-line"><span class="code-key">when:</span> 2026-03-07 11:53 AM</div>
          <div class="code-line"><span class="code-key">agent:</span> york</div>
          <div class="code-line"><span class="code-key">type:</span> correction</div>
          <div class="code-line"><span class="code-key">what:</span> James corrected calorie estimate for Chipotle bowl (said 800, was 1100)</div>
          <div class="code-line"><span class="code-key">category:</span> data-accuracy</div>
          <div class="code-line"><span class="code-key">domain:</span> health</div>
        </div>
        <p style="color: #8b949e; margin-top: 0.75rem; font-size: 0.85rem;">
          Types: correction, friction, slow-response, bad-routing, missed-opportunity, stale-data, user-frustration
        </p>
      </div>
    </div>

    <!-- Analysis Detail -->
    <div class="card border-blue">
      <h3>Pattern Analysis</h3>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Reads the journal. Finds patterns humans miss. Cross-domain connections.
        This is where the Large model earns its cost.
      </p>

      <div class="analysis-examples">
        <div class="analysis-item" v-for="a in analyses" :key="a.pattern">
          <div class="analysis-pattern">
            <span class="analysis-label">Pattern:</span> {{ a.pattern }}
          </div>
          <div class="analysis-insight">
            <span class="analysis-label">Insight:</span> {{ a.insight }}
          </div>
          <div class="analysis-action">
            <span class="analysis-label">Action:</span> {{ a.action }}
          </div>
        </div>
      </div>

      <div class="analysis-categories">
        <h4>Failure Categories</h4>
        <div class="cat-grid">
          <div class="cat-item" v-for="cat in categories" :key="cat.name">
            <span class="cat-name">{{ cat.name }}</span>
            <span class="cat-fix">{{ cat.fix }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Experimentation -->
    <div class="card border-yellow">
      <h3>Experimentation</h3>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Hypothesize → change → measure. Not "fix and forget."
        Every change has success criteria defined before implementation.
      </p>

      <div class="exp-flow">
        <div class="exp-step" v-for="step in expSteps" :key="step.label">
          <div class="exp-label">{{ step.label }}</div>
          <div class="exp-desc">{{ step.desc }}</div>
        </div>
      </div>

      <div class="exp-spawns">
        <h4>What Gets Spawned</h4>
        <div class="spawn-grid">
          <div class="spawn-item" v-for="s in spawns" :key="s.task">
            <span :class="['tag', s.tierClass]">{{ s.tier }}</span>
            <span class="spawn-task">{{ s.task }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Verification -->
    <div class="card border-purple">
      <h3>Verification</h3>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        The part the current system completely lacks. Did the fix actually help?
        If the same mistake recurs, the experiment failed and we need a different approach.
      </p>

      <div class="verify-methods">
        <div class="verify-item" v-for="v in verifications" :key="v.method">
          <div class="verify-method">{{ v.method }}</div>
          <div class="verify-how">{{ v.how }}</div>
        </div>
      </div>
    </div>

    <!-- Cross-Domain Span -->
    <div class="card">
      <h3>Why Self-Improvement Spans Everything</h3>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Self-improvement isn't a domain like Health or D&D. It's a cross-cutting concern.
        The observation layer is embedded in every agent. The analysis touches every domain.
        The only boundary is the OpenClaw instance — MTG (separate install) handles its own improvement.
      </p>

      <div class="span-diagram">
        <div class="span-header">
          <div class="span-label">SELF-IMPROVEMENT FEEDBACK LOOP</div>
        </div>
        <div class="span-domains">
          <div class="span-domain" v-for="d in domains" :key="d.name">
            <div class="span-icon">{{ d.icon }}</div>
            <div class="span-name">{{ d.name }}</div>
            <div class="span-examples">{{ d.examples }}</div>
          </div>
        </div>
        <div class="span-footer">
          <div class="span-note">
            Every domain generates observations. Analysis finds cross-domain patterns.
            Experiments can modify any skill, cron, routing rule, or agent behavior.
          </div>
        </div>
      </div>
    </div>

    <!-- What Changes vs Current -->
    <div class="card">
      <h3>Current System vs This</h3>
      <div class="compare-table">
        <div class="compare-row header">
          <div class="compare-col">Aspect</div>
          <div class="compare-col current">Current (improvement-queue.md)</div>
          <div class="compare-col proposed">Proposed (Feedback Loop)</div>
        </div>
        <div class="compare-row" v-for="c in comparisons" :key="c.aspect">
          <div class="compare-col aspect">{{ c.aspect }}</div>
          <div class="compare-col current">{{ c.current }}</div>
          <div class="compare-col proposed">{{ c.proposed }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const loopSteps = [
  {
    name: 'Observe',
    tier: 'Trivial',
    tierClass: 'script',
    when: 'Every session',
    whenClass: 'heartbeat',
    desc: 'Every agent writes friction events to a shared journal as they happen. Corrections, slow responses, bad data, user frustration, missed opportunities. This is a reflex, not a task.',
    examples: [
      'James corrected a calorie estimate → journal entry',
      'Sub-agent failed and had to retry → journal entry',
      'Morning brief was missing data that should have been cached → journal entry',
      'James said "I already told you this" → journal entry (high priority)',
    ],
  },
  {
    name: 'Analyze',
    tier: 'Large',
    tierClass: 'large',
    when: 'Overnight + on-demand',
    whenClass: 'cron',
    desc: 'Read the journal. Find patterns across days and domains. Categorize failures. Decide what\'s fixable, what needs research, and what\'s a model limitation. This is where the big model earns its cost.',
    examples: [
      '"Calorie estimates for restaurant meals are wrong 60% of the time" → skill file needs a restaurant lookup step',
      '"James gets frustrated every time York asks permission for obvious things" → SOUL.md or dispatch rules need updating',
      '"Camera snapshots fail every 3rd time at night" → infrastructure issue, not skill issue',
    ],
  },
  {
    name: 'Hypothesize & Experiment',
    tier: 'Varies',
    tierClass: 'medium',
    when: 'Overnight',
    whenClass: 'cron',
    desc: 'Form a hypothesis about why something fails. Implement a change with clear success criteria. The model tier depends on the fix: skill file edits are Small, new skills are XL.',
  },
  {
    name: 'Verify',
    tier: 'Small',
    tierClass: 'small',
    when: 'Days after change',
    whenClass: 'cron',
    desc: 'Check if the same pattern recurs in the journal after the fix. If it does, the experiment failed — try a different approach or escalate. If it stopped, log the successful fix in the changelog.',
  },
]

const observations = [
  {
    agent: '🦝 York',
    watches: [
      'James corrections in conversation',
      'Repeated questions (York forgot something)',
      'Gate decisions James disagreed with',
      'Times James said "I already told you"',
    ],
  },
  {
    agent: '🍎 Health',
    watches: [
      'Calorie estimates that get corrected',
      'Spreadsheet write errors',
      'Missing or duplicate entries',
      'Skill file instructions that were ambiguous',
    ],
  },
  {
    agent: '🎲 D&D',
    watches: [
      'Lore contradictions in wiki',
      'Questions James already answered',
      'Research that duplicated existing articles',
      'Session prep gaps',
    ],
  },
  {
    agent: '🏠 Chores',
    watches: [
      'Camera false positives/negatives',
      'Chore state drift from reality',
      'Gate data that was stale',
    ],
  },
  {
    agent: '☀️ Morning Brief',
    watches: [
      'Sections with stale or missing data',
      'Tone misreads (wrong framing for the day)',
      'Data that should have been cached but wasn\'t',
    ],
  },
  {
    agent: '🔧 Cron/Scripts',
    watches: [
      'Failed runs (non-zero exit)',
      'Timeout or slow execution',
      'Output format changes breaking consumers',
    ],
  },
]

const analyses = [
  {
    pattern: '4 calorie corrections on restaurant meals in 2 weeks',
    insight: 'The consumption skill file doesn\'t have a step for looking up restaurant-specific nutrition data',
    action: 'Add a restaurant lookup step to the consumption skill → spawn Builder',
  },
  {
    pattern: 'James says "I already told you" 3 times about house layout',
    insight: 'Important context is being stored in daily memory files that expire, not in a permanent reference',
    action: 'Create a permanent reference file for house context → simple fix, do inline',
  },
  {
    pattern: 'Camera snapshot fails 30% of the time between 11 PM and 6 AM',
    insight: 'Cameras may be entering a power-saving or IR mode that delays RTSP response',
    action: 'Research camera night mode behavior → spawn Research agent, then adjust timeout',
  },
  {
    pattern: 'Overnight worker re-suggests items James already rejected in #reviews',
    insight: 'No record of rejected proposals accessible to the overnight worker',
    action: 'Add a "rejected" section to the changelog, check before proposing → simple fix',
  },
]

const categories = [
  { name: 'Skill gap', fix: 'Update or create skill file' },
  { name: 'Stale data', fix: 'Fix cron timing or add cache step' },
  { name: 'Wrong routing', fix: 'Update dispatch rules' },
  { name: 'Bad judgment', fix: 'Update SOUL.md or agent instructions' },
  { name: 'Missing context', fix: 'Add to permanent reference files' },
  { name: 'Infrastructure', fix: 'Fix script, adjust timeout, add retry' },
  { name: 'Model limitation', fix: 'Document workaround, don\'t fight it' },
  { name: 'Architectural', fix: 'Needs redesign — spawn Builder or flag for James' },
]

const expSteps = [
  { label: 'Hypothesis', desc: 'Clear statement: "If I change X, then Y should stop happening"' },
  { label: 'Success criteria', desc: 'Measurable: "Zero calorie corrections on restaurant meals for 7 days"' },
  { label: 'Implementation', desc: 'Make the change. Log what was changed and why.' },
  { label: 'Observation window', desc: 'Wait. Don\'t declare victory immediately. Let the journal collect data.' },
  { label: 'Verdict', desc: 'Did it work? If yes → changelog. If no → try different approach or escalate.' },
]

const spawns = [
  { tier: 'Script', tierClass: 'script', task: 'Cron timing adjustment, config tweak, file move' },
  { tier: 'Small', tierClass: 'small', task: 'Skill file wording update, add a missing step, fix a template' },
  { tier: 'Medium', tierClass: 'medium', task: 'Restructure a skill, adjust routing rules, rewrite a cron job' },
  { tier: 'Large', tierClass: 'large', task: 'Cross-domain analysis, pattern diagnosis, architectural assessment' },
  { tier: 'XL', tierClass: 'xl', task: 'New skill creation, major refactor, new agent setup' },
]

const verifications = [
  {
    method: 'Journal recurrence check',
    how: 'Search journal for the same failure pattern after the fix. If it appears again, the fix failed.',
  },
  {
    method: 'Before/after data quality',
    how: 'Compare error rates in a domain (e.g., calorie corrections per week) before and after the change.',
  },
  {
    method: 'James feedback signal',
    how: 'If James stops correcting something, it\'s probably fixed. Don\'t ask explicitly — observe.',
  },
  {
    method: 'Automated checks',
    how: 'Scripts that validate data consistency (e.g., spreadsheet rows aren\'t duplicated, cache files are fresh).',
  },
]

const domains = [
  { icon: '🍎', name: 'Health', examples: 'Calorie accuracy, logging reliability, trend detection' },
  { icon: '🏠', name: 'Chores', examples: 'Camera reliability, gate accuracy, chore state drift' },
  { icon: '🎲', name: 'D&D', examples: 'Wiki consistency, research quality, lore accuracy' },
  { icon: '💬', name: 'Conversation', examples: 'Tone, memory, permission-seeking, verbosity' },
  { icon: '☀️', name: 'Morning Brief', examples: 'Data freshness, editorial quality, relevance' },
  { icon: '💰', name: 'Finance', examples: 'Transaction categorization, budget tracking accuracy' },
  { icon: '⚙️', name: 'Infrastructure', examples: 'Cron reliability, script failures, API timeouts' },
  { icon: '🦝', name: 'York Core', examples: 'Routing decisions, panel presence, heartbeat behavior' },
]

const comparisons = [
  {
    aspect: 'Input',
    current: 'Manual TODO items, sometimes from conversation, sometimes not',
    proposed: 'Automatic friction logging from every agent, every session',
  },
  {
    aspect: 'Memory',
    current: 'Flat file re-read from scratch each session',
    proposed: 'Timestamped journal with categories, searchable',
  },
  {
    aspect: 'Priority',
    current: 'P0/P1/P2 labels assigned once, never updated',
    proposed: 'Priority emerges from pattern frequency — recurring issues rise naturally',
  },
  {
    aspect: 'Execution',
    current: 'Overnight worker picks whatever looks interesting',
    proposed: 'Analysis identifies highest-impact patterns, spawns right-sized agents',
  },
  {
    aspect: 'Verification',
    current: 'None. "Done" means someone edited the file.',
    proposed: 'Journal recurrence check confirms fix worked. Failed fixes get retried.',
  },
  {
    aspect: 'Scope',
    current: 'Only what someone remembers to add',
    proposed: 'Every correction, every frustration, every failure — automatically captured',
  },
  {
    aspect: 'Cross-domain',
    current: 'Items are siloed by whoever wrote them',
    proposed: 'Analysis finds connections across domains (spending + eating, sleep + gate decisions)',
  },
]
</script>

<style scoped>
.loop {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.loop-step {
  display: flex;
  gap: 1rem;
  padding: 1rem 0;
}

.loop-number {
  background: #30363d;
  color: #f0f6fc;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  flex-shrink: 0;
}

.loop-content {
  flex: 1;
}

.loop-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
  flex-wrap: wrap;
}

.loop-name {
  font-weight: 700;
  color: #f0f6fc;
  font-size: 1.1rem;
}

.loop-desc {
  color: #c9d1d9;
  line-height: 1.6;
  font-size: 0.9rem;
  margin-bottom: 0.5rem;
}

.loop-examples {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.loop-example {
  color: #8b949e;
  font-size: 0.8rem;
  padding-left: 1rem;
  border-left: 2px solid #21262d;
}

.loop-arrow {
  text-align: center;
  color: #484f58;
  font-size: 1.2rem;
  padding: 0.25rem 0;
  margin-left: 14px;
}

.loop-return {
  text-align: center;
  color: #58a6ff;
  font-size: 0.85rem;
  padding: 1rem;
  border: 1px dashed #1f6feb;
  border-radius: 6px;
  margin-top: 0.5rem;
}

.obs-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.obs-card {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1rem;
}

.obs-agent {
  font-weight: 600;
  color: #f0f6fc;
  margin-bottom: 0.5rem;
}

.obs-item {
  color: #8b949e;
  font-size: 0.8rem;
  padding: 0.2rem 0;
}

.obs-item::before {
  content: '· ';
}

.obs-format {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1rem;
}

.obs-format h4 {
  color: #8b949e;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.75rem;
}

.code-block {
  font-family: monospace;
  font-size: 0.8rem;
  line-height: 1.8;
}

.code-key {
  color: #ff7b72;
}

.code-line {
  color: #c9d1d9;
}

.analysis-examples {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.analysis-item {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1rem;
}

.analysis-pattern, .analysis-insight, .analysis-action {
  font-size: 0.85rem;
  padding: 0.25rem 0;
  line-height: 1.5;
}

.analysis-pattern { color: #f85149; }
.analysis-insight { color: #d29922; }
.analysis-action { color: #3fb950; }
.analysis-label {
  font-weight: 600;
  font-size: 0.75rem;
  text-transform: uppercase;
}

.analysis-categories { margin-top: 1rem; }

.analysis-categories h4 {
  color: #8b949e;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.75rem;
}

.cat-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 0.5rem;
}

.cat-item {
  display: flex;
  justify-content: space-between;
  padding: 0.5rem;
  background: #0d1117;
  border-radius: 4px;
  font-size: 0.85rem;
}

.cat-name { color: #f0f6fc; font-weight: 500; }
.cat-fix { color: #8b949e; }

.exp-flow {
  display: flex;
  flex-direction: column;
  gap: 0;
  margin-bottom: 1.5rem;
}

.exp-step {
  display: flex;
  gap: 1rem;
  padding: 0.75rem 0;
  border-bottom: 1px solid #21262d;
}

.exp-step:last-child { border-bottom: none; }

.exp-label {
  color: #58a6ff;
  font-weight: 600;
  font-size: 0.85rem;
  min-width: 140px;
}

.exp-desc {
  color: #c9d1d9;
  font-size: 0.85rem;
}

.exp-spawns h4 {
  color: #8b949e;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.75rem;
}

.spawn-grid {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.spawn-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  font-size: 0.85rem;
}

.spawn-task { color: #c9d1d9; }

.verify-methods {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.verify-item {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1rem;
}

.verify-method {
  color: #bc8cff;
  font-weight: 600;
  font-size: 0.9rem;
  margin-bottom: 0.25rem;
}

.verify-how {
  color: #8b949e;
  font-size: 0.85rem;
  line-height: 1.5;
}

.span-diagram {
  background: #0d1117;
  border: 2px solid #58a6ff;
  border-radius: 8px;
  overflow: hidden;
}

.span-header {
  background: #1f6feb;
  padding: 0.75rem;
  text-align: center;
}

.span-label {
  color: white;
  font-weight: 700;
  font-size: 0.8rem;
  letter-spacing: 0.1em;
}

.span-domains {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 0.5rem;
  padding: 1rem;
}

.span-domain {
  text-align: center;
  padding: 0.75rem;
  border: 1px dashed #30363d;
  border-radius: 4px;
}

.span-icon { font-size: 1.5rem; margin-bottom: 0.25rem; }
.span-name { color: #f0f6fc; font-weight: 600; font-size: 0.85rem; display: block; }
.span-examples { color: #8b949e; font-size: 0.7rem; margin-top: 0.25rem; }

.span-footer {
  padding: 0.75rem 1rem;
  border-top: 1px solid #30363d;
}

.span-note {
  color: #8b949e;
  font-size: 0.8rem;
  text-align: center;
  line-height: 1.5;
}

.border-green { border-color: #3fb950; }
.border-blue { border-color: #58a6ff; }
.border-yellow { border-color: #d29922; }
.border-purple { border-color: #bc8cff; }

.compare-table {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.compare-row {
  display: grid;
  grid-template-columns: 120px 1fr 1fr;
  gap: 1rem;
  padding: 0.75rem;
  border-bottom: 1px solid #21262d;
}

.compare-row:last-child { border-bottom: none; }

.compare-row.header {
  font-weight: 700;
  color: #8b949e;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  border-bottom: 2px solid #30363d;
}

.compare-col {
  font-size: 0.85rem;
  line-height: 1.5;
}

.compare-col.aspect {
  color: #f0f6fc;
  font-weight: 600;
}

.compare-col.current {
  color: #f85149;
}

.compare-col.proposed {
  color: #3fb950;
}

@media (max-width: 768px) {
  .compare-row {
    grid-template-columns: 1fr;
    gap: 0.25rem;
  }
  .obs-grid {
    grid-template-columns: 1fr;
  }
}
</style>
