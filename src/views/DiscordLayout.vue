<template>
  <div>
    <div class="card">
      <h2>Communication Channels</h2>
      <p style="color: #8b949e; margin-bottom: 1rem;">
        Discord is the primary interface. Channel routing determines which agent handles the conversation.
        Most channels route to York who dispatches. Some route directly to domain agents for interactive sessions.
      </p>
      <p style="color: #c9d1d9; margin-bottom: 0; line-height: 1.6; font-size: 0.9rem; border-left: 3px solid #f22f46; padding-left: 1rem;">
        <strong style="color: #f22f46;">Twilio SMS</strong> — planned for nudges and urgent alerts.
        York will send SMS via york-tools.twilio_send() when Discord isn't enough.
        Late in the build plan but accounted for in the architecture.
      </p>
    </div>

    <div class="server">
      <div class="server-header">
        <span class="server-icon">🦊</span>
        <span class="server-name">York's Server</span>
      </div>

      <div class="category" v-for="cat in categories" :key="cat.name">
        <div class="category-header">▾ {{ cat.name }}</div>
        <div
          class="channel"
          v-for="ch in cat.channels"
          :key="ch.name"
        >
          <div class="channel-left">
            <span class="channel-hash">#</span>
            <span class="channel-name">{{ ch.name }}</span>
          </div>
          <div class="channel-right">
            <span :class="['tag', ch.agentClass]">{{ ch.agent }}</span>
            <span class="channel-desc">{{ ch.desc }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="card" style="margin-top: 1rem;">
      <h3>Routing Rules</h3>
      <div class="rules">
        <div class="rule" v-for="rule in rules" :key="rule.channel">
          <div class="rule-channel">#{{ rule.channel }}</div>
          <div class="rule-arrow">→</div>
          <div class="rule-detail">
            <strong>{{ rule.agent }}</strong>
            <span class="rule-desc">{{ rule.desc }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="card" style="margin-top: 1rem;">
      <h3>Design Decisions</h3>
      <ul class="decisions">
        <li><strong>Every agent with a channel gets direct routing.</strong> Messages in an agent's channel go straight to that agent. No York proxy. Less context burn, faster responses.</li>
        <li><strong>#general stays as the main channel.</strong> York lives here. Quick questions, accountability, daily chat. York delegates to other agents as needed.</li>
        <li><strong>Agent channels use emoji prefixes.</strong> #🐉caedmon, #🦫offa, #🦌wynn, etc. #general does not get an emoji — only channels that match an agent name.</li>
        <li><strong>#🐻wiglaf is fully siloed.</strong> Private work agent. No cross-agent access, no Bede analysis. Separate by design.</li>
        <li><strong>#🐺guthlac is fully siloed.</strong> Private personal agent. No cross-agent access, no Bede analysis. xAI model (Grok).</li>
        <li><strong>Each conversational agent gets its own voice.</strong> Caedmon sounds different from York. Wiglaf is professional and direct. Personalities are defined in SOUL.md.</li>
      </ul>
    </div>
  </div>
</template>

<script setup>
const categories = [
  {
    name: 'DAILY',
    channels: [
      {
        name: 'general',
        agent: 'York 🦊',
        agentClass: 'large',
        desc: 'Main conversation, morning briefs, accountability',
      },
    ],
  },
  {
    name: 'AGENTS',
    channels: [
      {
        name: '🐓dagr',
        agent: 'Dagr 🐓',
        agentClass: 'sonnet',
        desc: 'Morning brief, daily summaries',
      },
      {
        name: '🐉caedmon',
        agent: 'Caedmon 🐉',
        agentClass: 'large',
        desc: 'D&D campaign, wiki, lore, worldbuilding, session prep',
      },
      {
        name: '🦫offa',
        agent: 'Offa 🦫',
        agentClass: 'large',
        desc: 'Direct build instructions, skill creation, debugging',
      },
      {
        name: '🦌wynn',
        agent: 'Wynn 🦌',
        agentClass: 'sonnet',
        desc: 'Health & fitness logging, coaching',
      },
      {
        name: '🐻wiglaf',
        agent: 'Wiglaf 🐻',
        agentClass: 'large',
        desc: 'Private work agent, prioritization, meeting prep',
      },
      {
        name: '🦡hild',
        agent: 'Hild 🦡',
        agentClass: 'sonnet',
        desc: 'Home management, chore tracking, habit stacks',
      },
      {
        name: '🐺guthlac',
        agent: 'Guthlac 🐺',
        agentClass: 'grok',
        desc: 'Private personal agent, conversation, personality shaping',
      },
    ],
  },
]

const rules = [
  {
    channel: 'general',
    agent: 'York 🦊',
    desc: 'All messages route to York. York delegates to Wynn, Hild, etc. as needed. Morning brief, nudges, daily chat.',
  },
  {
    channel: '🐓dagr',
    agent: 'Dagr 🐓 (Direct)',
    desc: 'Morning brief channel. Dagr posts daily summaries and handles morning-related conversation.',
  },
  {
    channel: '🐉caedmon',
    agent: 'Caedmon 🐉 (Direct)',
    desc: 'Messages route directly to Caedmon. No York proxy. Interactive wiki/lore sessions.',
  },
  {
    channel: '🦫offa',
    agent: 'Offa 🦫 (Direct)',
    desc: 'James gives Offa build instructions directly. Hands-on building and skill creation.',
  },
  {
    channel: '🦌wynn',
    agent: 'Wynn 🦌 (Direct)',
    desc: 'Health & fitness logging. Food, workouts, weight, coaching — all direct to Wynn.',
  },
  {
    channel: '🐻wiglaf',
    agent: 'Wiglaf 🐻 (Direct)',
    desc: 'Private work agent. Prioritization, meeting prep, work context. Fully siloed from other agents.',
  },
  {
    channel: '🦡hild',
    agent: 'Hild 🦡 (Direct)',
    desc: 'Home management. Chore status, habit stacks, completions. Evening gate evaluation.',
  },
  {
    channel: '🐺guthlac',
    agent: 'Guthlac 🐺 (Direct)',
    desc: 'Private personal agent. Conversation, venting, thinking. Fully siloed. xAI Grok model.',
  },
]
</script>

<style scoped>
.server {
  background: #2b2d31;
  border-radius: 8px;
  overflow: hidden;
}

.server-header {
  padding: 1rem 1.5rem;
  border-bottom: 1px solid #1e1f22;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.server-icon {
  font-size: 1.5rem;
}

.server-name {
  font-weight: 700;
  color: #f2f3f5;
  font-size: 1.1rem;
}

.category {
  padding: 0.5rem 0;
}

.category-header {
  padding: 0.5rem 1rem;
  font-size: 0.7rem;
  font-weight: 700;
  color: #949ba4;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.channel {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.4rem 1rem 0.4rem 1.5rem;
  cursor: pointer;
  transition: background 0.1s;
}

.channel:hover {
  background: #35373c;
}

.channel-left {
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.channel-hash {
  color: #949ba4;
  font-size: 1.1rem;
  margin-right: 0.25rem;
}

.channel-name {
  color: #949ba4;
  font-weight: 500;
}

.channel:hover .channel-name {
  color: #dbdee1;
}

.channel-right {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.channel-desc {
  color: #6d6f78;
  font-size: 0.75rem;
  max-width: 300px;
  text-align: right;
}

.rules {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.rule {
  display: flex;
  align-items: flex-start;
  gap: 0.75rem;
}

.rule-channel {
  color: #58a6ff;
  font-weight: 600;
  min-width: 100px;
  font-size: 0.9rem;
}

.rule-arrow {
  color: #484f58;
}

.rule-detail {
  font-size: 0.9rem;
  line-height: 1.5;
}

.rule-desc {
  color: #8b949e;
  margin-left: 0.5rem;
}

.decisions {
  list-style: none;
  padding: 0;
}

.decisions li {
  padding: 0.5rem 0;
  line-height: 1.5;
  border-bottom: 1px solid #21262d;
  font-size: 0.9rem;
}

.decisions li:last-child {
  border-bottom: none;
}
</style>
