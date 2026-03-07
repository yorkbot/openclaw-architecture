<template>
  <div>
    <div class="card">
      <h2>Cost Estimate</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Estimated monthly cost with mixed API providers vs current $200/mo flat plan.
        Based on observed usage patterns from the last week.
      </p>
    </div>

    <div class="card">
      <h3>Per-Agent Monthly Estimate</h3>
      <table>
        <thead>
          <tr>
            <th>Agent</th>
            <th>Model</th>
            <th>Calls/Day</th>
            <th>Avg Tokens</th>
            <th>$/Month</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="row in costRows" :key="row.agent" :class="{ 'row-total': row.total }">
            <td>{{ row.agent }}</td>
            <td><span :class="['tag', row.modelClass]" v-if="row.model">{{ row.model }}</span></td>
            <td>{{ row.calls }}</td>
            <td>{{ row.tokens }}</td>
            <td class="cost-cell">{{ row.cost }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="card">
      <h3>Comparison</h3>
      <div class="compare-grid">
        <div class="compare-card current">
          <div class="compare-label">Current</div>
          <div class="compare-price">$200/mo</div>
          <div class="compare-desc">Claude Max flat plan</div>
          <ul>
            <li>✓ Unlimited Opus calls</li>
            <li>✗ Rate limited on heavy use</li>
            <li>✗ Single provider only</li>
            <li>✗ Paying for Opus on trivial tasks</li>
          </ul>
        </div>
        <div class="compare-card proposed">
          <div class="compare-label">Proposed</div>
          <div class="compare-price">~$75-120/mo</div>
          <div class="compare-desc">Mixed API pricing</div>
          <ul>
            <li>✓ Right model for each task</li>
            <li>✓ No rate limits</li>
            <li>✓ Multi-provider flexibility</li>
            <li>✓ Scripts eliminate LLM costs for trivial work</li>
            <li>✗ Heavy building days could spike</li>
          </ul>
        </div>
        <div class="compare-card savings">
          <div class="compare-label">Savings</div>
          <div class="compare-price">$80-125/mo</div>
          <div class="compare-desc">40-60% reduction</div>
          <ul>
            <li>Biggest savings: replacing Opus sub-agents with Sonnet</li>
            <li>Second biggest: scripts replacing LLM for trivial tasks</li>
            <li>Variance: heavy building weeks cost more</li>
          </ul>
        </div>
      </div>
    </div>

    <div class="card">
      <h3>Assumptions & Caveats</h3>
      <ul class="caveats">
        <li>Token estimates based on observed session sizes from this week</li>
        <li>Cached input pricing assumed where OpenClaw supports it (Anthropic, OpenAI)</li>
        <li>Builder agent is highly variable — one big project day could be $10-20 alone</li>
        <li>"Calls/Day" for weekly agents are averaged (e.g., 1 call/week = 0.14/day)</li>
        <li>OpenRouter adds ~10-20% markup over direct API pricing</li>
        <li>Image generation costs not included (Gemini free tier or separate budget)</li>
        <li>These are estimates. First month of API usage will give real numbers.</li>
      </ul>
    </div>
  </div>
</template>

<script setup>
const costRows = [
  { agent: 'York (Orchestrator)', model: 'Opus', modelClass: 'opus', calls: '~15', tokens: '50k in / 2k out', cost: '$25-40' },
  { agent: 'Health & Nutrition', model: 'Sonnet', modelClass: 'sonnet', calls: '~3', tokens: '20k in / 1k out', cost: '$3-5' },
  { agent: 'Morning Brief', model: 'Sonnet', modelClass: 'sonnet', calls: '1', tokens: '30k in / 2k out', cost: '$2-3' },
  { agent: 'DnD Campaign', model: 'Sonnet', modelClass: 'sonnet', calls: '~2', tokens: '25k in / 2k out', cost: '$3-5' },
  { agent: 'Weekly Review', model: 'Opus', modelClass: 'opus', calls: '0.14', tokens: '40k in / 3k out', cost: '$3-5' },
  { agent: 'Overnight Worker', model: 'Sonnet', modelClass: 'sonnet', calls: '~10', tokens: '15k in / 1k out', cost: '$5-8' },
  { agent: 'DnD Research (cron)', model: 'Sonnet', modelClass: 'sonnet', calls: '1', tokens: '30k in / 3k out', cost: '$2-3' },
  { agent: 'Chores', model: 'Sonnet', modelClass: 'sonnet', calls: '~2', tokens: '15k in / 1k out', cost: '$1-2' },
  { agent: 'Research', model: 'Varies', modelClass: 'grok', calls: '~1', tokens: '20k in / 2k out', cost: '$2-5' },
  { agent: 'Builder', model: 'Opus', modelClass: 'opus', calls: '~0.5', tokens: '80k in / 5k out', cost: '$8-15' },
  { agent: 'Memory/Compile/Other', model: 'Sonnet', modelClass: 'sonnet', calls: '~3', tokens: '15k in / 1k out', cost: '$3-5' },
  { agent: 'Scripts (no LLM)', model: null, modelClass: '', calls: '~8', tokens: 'N/A', cost: '$0' },
  { agent: '', model: null, modelClass: '', calls: '', tokens: '', cost: '', total: false },
  { agent: 'TOTAL (estimated)', model: null, modelClass: '', calls: '~45', tokens: '', cost: '$57-96', total: true },
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

.cost-cell {
  color: #3fb950;
  font-weight: 600;
}

.row-total td {
  border-top: 2px solid #58a6ff;
  font-weight: 700;
  color: #f0f6fc;
  font-size: 1rem;
}

.row-total .cost-cell {
  color: #58a6ff;
  font-size: 1.1rem;
}

.compare-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.compare-card {
  background: #0d1117;
  border: 1px solid #30363d;
  border-radius: 6px;
  padding: 1.5rem;
  text-align: center;
}

.compare-label {
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: #8b949e;
  margin-bottom: 0.5rem;
}

.compare-price {
  font-size: 2rem;
  font-weight: 700;
  margin-bottom: 0.25rem;
}

.current .compare-price { color: #f85149; }
.proposed .compare-price { color: #3fb950; }
.savings .compare-price { color: #58a6ff; }

.compare-desc {
  color: #8b949e;
  font-size: 0.85rem;
  margin-bottom: 1rem;
}

.compare-card ul {
  list-style: none;
  padding: 0;
  text-align: left;
}

.compare-card li {
  padding: 0.25rem 0;
  font-size: 0.8rem;
  color: #c9d1d9;
}

.caveats {
  list-style: none;
  padding: 0;
}

.caveats li {
  padding: 0.4rem 0;
  font-size: 0.85rem;
  color: #8b949e;
  border-bottom: 1px solid #21262d;
}

.caveats li::before {
  content: '⚠️ ';
}

@media (max-width: 768px) {
  .compare-grid {
    grid-template-columns: 1fr;
  }
}
</style>
