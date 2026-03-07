<template>
  <div>
    <div class="card">
      <h2>Cost Estimate</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Estimated monthly cost with right-sized models vs current $200/mo flat plan.
        The goal is performance, not penny-pinching. Waste elimination, not cost cutting.
      </p>
    </div>

    <div class="card">
      <h3>Per-Agent Monthly Estimate</h3>
      <p style="color: #8b949e; margin-bottom: 1rem; font-size: 0.85rem;">
        Costs depend heavily on which specific models fill each tier. These estimates use
        mid-range pricing for each tier. Actual costs will vary by provider choice.
      </p>
      <table>
        <thead>
          <tr>
            <th>Agent</th>
            <th>Tier</th>
            <th>Calls/Day</th>
            <th>Why This Tier</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="row in costRows" :key="row.agent" :class="{ 'row-total': row.total }">
            <td>{{ row.agent }}</td>
            <td><span :class="['tag', row.tierClass]" v-if="row.tier">{{ row.tier }}</span></td>
            <td>{{ row.calls }}</td>
            <td class="why-cell">{{ row.why }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="card">
      <h3>Where the Current Setup Wastes</h3>
      <div class="waste-grid">
        <div class="waste-item" v-for="w in wastes" :key="w.task">
          <div class="waste-header">
            <span class="waste-task">{{ w.task }}</span>
            <span class="waste-arrow">→</span>
          </div>
          <div class="waste-detail">
            <div class="waste-current">
              <span class="waste-label">Now:</span> {{ w.current }}
            </div>
            <div class="waste-proposed">
              <span class="waste-label">Should be:</span> {{ w.proposed }}
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="card">
      <h3>Key Principle</h3>
      <div class="principle">
        <p>
          The current $200/mo flat plan was the right call for learning the system.
          The migration isn't about being cheaper — it's about being <strong>intentional</strong>.
          Every agent gets the model that matches its task complexity. No more, no less.
        </p>
        <p>
          The savings are a side effect of not being wasteful, not the goal.
          If a task needs a large model, it gets one. The point is that logging
          "2 lattes, a bagel" to a spreadsheet doesn't.
        </p>
      </div>
    </div>
  </div>
</template>

<script setup>
const costRows = [
  { agent: 'York (Orchestrator)', tier: 'Large', tierClass: 'large', calls: '~15', why: 'Judgment calls, conversation quality, gate decisions' },
  { agent: 'Health & Nutrition', tier: 'Small', tierClass: 'small', calls: '~3', why: 'Structured spreadsheet writes following skill instructions' },
  { agent: 'D&D Campaign', tier: 'Large', tierClass: 'large', calls: '~2', why: 'Interconnected wiki, creative reasoning, large context' },
  { agent: 'Morning Brief', tier: 'Medium', tierClass: 'medium', calls: '1', why: 'Editorial judgment on pre-cached data' },
  { agent: 'Weekly Review', tier: 'Large', tierClass: 'large', calls: '0.14', why: 'Week-long pattern analysis, opinion forming' },
  { agent: 'Overnight Worker', tier: 'Medium', tierClass: 'medium', calls: '~10', why: 'Task routing, simple fixes inline' },
  { agent: 'Chores', tier: 'Small', tierClass: 'small', calls: '~2', why: 'Camera vision + state tracking' },
  { agent: 'Research', tier: 'Varies', tierClass: 'medium', calls: '~1', why: 'Model chosen per-spawn based on task' },
  { agent: 'Builder', tier: 'XL', tierClass: 'xl', calls: '~0.5', why: 'Implementation, debugging, architecture. Claude Code territory.' },
  { agent: 'Memory/Compile/Other', tier: 'Small', tierClass: 'small', calls: '~3', why: 'Summarization, session review' },
  { agent: 'Scripts (no LLM)', tier: 'Script', tierClass: 'script', calls: '~8', why: 'Weather, calendar, weight, workout — no model needed' },
]

const wastes = [
  {
    task: 'Logging "258.2 lbs" to a spreadsheet',
    current: 'Spawns a full LLM sub-agent with Opus-level context',
    proposed: 'A bash script calling mcporter. Zero LLM tokens.',
  },
  {
    task: 'Fetching weather from wttr.in',
    current: 'Spawns an LLM agent that runs curl, then writes to a file',
    proposed: 'A 3-line bash script on cron.',
  },
  {
    task: 'Logging "2 lattes and a bagel with cream cheese"',
    current: 'Full Opus sub-agent loads workspace context, reads skill files, writes to spreadsheet',
    proposed: 'Small model with minimal context follows a structured skill file.',
  },
  {
    task: 'DnD lore conversation (20 back-and-forth messages)',
    current: 'Runs through York orchestrator — burns expensive orchestrator context on wiki lookups',
    proposed: 'Routes directly to DnD agent via #dnd channel. Orchestrator never involved.',
  },
  {
    task: 'Cannabis gate check',
    current: 'York does everything inline — reads chores, checks calories, makes decision',
    proposed: 'York spawns Small agents for data gathering, keeps only the judgment call.',
  },
]
</script>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
}

th {
  text-align: left;
  padding: 0.75rem;
  border-bottom: 2px solid #30363d;
  color: #8b949e;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.5rem 0.75rem;
  border-bottom: 1px solid #21262d;
  font-size: 0.9rem;
}

.why-cell {
  color: #8b949e;
  font-size: 0.85rem;
}

.row-total td {
  border-top: 2px solid #58a6ff;
  font-weight: 700;
  color: #f0f6fc;
  font-size: 1rem;
}

.waste-grid {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.waste-item {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1rem;
}

.waste-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.75rem;
}

.waste-task {
  color: #f0f6fc;
  font-weight: 600;
  font-size: 0.9rem;
}

.waste-arrow {
  color: #484f58;
}

.waste-detail {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

.waste-label {
  font-weight: 600;
  font-size: 0.8rem;
}

.waste-current {
  color: #f85149;
  font-size: 0.85rem;
  line-height: 1.5;
}

.waste-proposed {
  color: #3fb950;
  font-size: 0.85rem;
  line-height: 1.5;
}

.principle {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1.5rem;
}

.principle p {
  color: #c9d1d9;
  line-height: 1.7;
  font-size: 0.95rem;
  margin-bottom: 1rem;
}

.principle p:last-child {
  margin-bottom: 0;
}

.principle strong {
  color: #58a6ff;
}

@media (max-width: 768px) {
  .waste-detail {
    grid-template-columns: 1fr;
  }
}
</style>
