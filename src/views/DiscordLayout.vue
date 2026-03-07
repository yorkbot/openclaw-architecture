<template>
  <div>
    <div class="card">
      <h2>Communication Channels</h2>
      <p style="color: #8b949e; margin-bottom: 1.5rem;">
        Currently Discord only. Channel routing determines which agent handles the conversation. 
        Most channels route to York who dispatches. Some route directly to domain agents for interactive sessions.
      </p>
    </div>

    <div class="server">
      <div class="server-header">
        <span class="server-icon">🦝</span>
        <span class="server-name">York's Server</span>
      </div>

      <div class="category" v-for="cat in categories" :key="cat.name">
        <div class="category-header">▾ {{ cat.name }}</div>
        <div
          class="channel"
          v-for="ch in cat.channels"
          :key="ch.name"
          :class="{ 'channel-private': ch.private }"
        >
          <div class="channel-left">
            <span class="channel-hash">#</span>
            <span class="channel-name">{{ ch.name }}</span>
            <span class="channel-lock" v-if="ch.private">🔒</span>
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
        <li><strong>#health is private.</strong> Weight, calories, body data never visible in #general. James can discuss health privately; the morning brief in #general only shows what's appropriate.</li>
        <li><strong>#dnd routes directly to the DnD agent.</strong> Interactive lore sessions don't burn York's expensive context. James talks to a specialist who knows the wiki cold.</li>
        <li><strong>#general stays as the main channel.</strong> York lives here. Quick questions, accountability, daily chat, morning briefs.</li>
        <li><strong>#reviews is York-only.</strong> Improvement proposals, self-initiated changes. James reviews async.</li>
        <li><strong>No #work channel.</strong> Work prep is ad-hoc, spawned by York when asked. Doesn't need a persistent channel.</li>
        <li><strong>No #mtg channel.</strong> MTG is a separate OpenClaw instance entirely.</li>
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
        agent: 'York',
        agentClass: 'large',
        desc: 'Main conversation, morning briefs, accountability',
        private: false,
      },
      {
        name: 'health',
        agent: 'York → Health',
        agentClass: 'small',
        desc: 'Private health data, food logging, weight discussion',
        private: true,
      },
    ],
  },
  {
    name: 'HOBBIES',
    channels: [
      {
        name: 'dnd',
        agent: 'DnD Agent',
        agentClass: 'large',
        desc: 'Campaign wiki, lore, worldbuilding, session prep',
        private: false,
      },
    ],
  },
  {
    name: 'SYSTEM',
    channels: [
      {
        name: 'reviews',
        agent: 'York',
        agentClass: 'large',
        desc: 'York-initiated proposals, improvement ideas',
        private: false,
      },
    ],
  },
]

const rules = [
  {
    channel: 'general',
    agent: 'York (Orchestrator)',
    desc: 'All messages route to York. York delegates to sub-agents as needed. Morning brief, weekly review, nudges post here.',
  },
  {
    channel: 'health',
    agent: 'York → Health Agent',
    desc: 'York receives, spawns Health agent for data operations. Private channel — no health data leaks to #general.',
  },
  {
    channel: 'dnd',
    agent: 'DnD Agent (Direct)',
    desc: 'Messages route directly to the DnD agent. No York proxy. Interactive wiki/lore sessions at Sonnet cost. York can still spawn DnD agent for #general lore questions.',
  },
  {
    channel: 'reviews',
    agent: 'York',
    desc: 'York posts proposals here. James approves/rejects. York tracks in improvement-queue.md.',
  },
]

const rules2 = []
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

.channel-lock {
  font-size: 0.7rem;
  margin-left: 0.25rem;
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

.channel-private {
  opacity: 0.8;
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
