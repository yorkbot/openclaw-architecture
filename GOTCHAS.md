# Multi-Agent Architecture — Gotchas & Failure Modes

**Author:** York (overnight research, 2026-03-08)
**Purpose:** Things that will bite us if we don't address them. Derived from real patterns in the current system + known multi-agent failure modes.

---

## 1. Context Handoff Loss

**The problem:** When York spawns a sub-agent, context travels via the `task` string. Details get lost or distorted in translation. We already see this: sub-agents ignore constraints, miss nuance, produce output that contradicts what James said.

**Current evidence:**
- Meal system sub-agent ignored "1 suggestion only, no parenthetical stats" despite SKILL.md saying exactly that (Mar 5 mistake journal)
- Consumption sub-agent hardcoded all timestamps to noon (Mar 1)
- Consumption sub-agent cleared existing rows while fixing one entry (Mar 4)

**Why it gets worse with multi-agent:** More agents = more handoffs = more loss. Today York does some work inline where it has full context. Splitting into agents forces EVERYTHING through handoffs.

**Mitigations to design:**
- Standardized task description templates per agent (not free-form prose)
- Agent-specific SKILL.md files that carry the constraints (sub-agent reads its own skill, doesn't rely on the spawn message for rules)
- York should pass minimal, structured context — not dump the conversation
- The Improvement Analyst should specifically watch for "sub-agent ignored a constraint" patterns

---

## 2. Routing Ambiguity

**The problem:** "Is this a health question or a chore question?" doesn't exist in a monolithic system. With 15-20 agents, every user message needs a routing decision. Wrong routing = wrong context = wrong answer.

**Examples that would be ambiguous:**
- "I did some push-ups before cleaning the kitchen" — Health agent? Chores agent? Both?
- "What should I make for dinner?" — Meal agent? But cannabis gate needs to know about dinner plans too.
- "Audrey's coming this weekend" — York context (schedule), Chores (house prep), Meal (cooking for two)

**Why it matters:** Routing errors are a NEW failure mode. Currently, York handles everything and has all context. Bad routing means the right agent never sees the message, or the wrong agent tries to handle it without the needed context.

**Mitigations to design:**
- York handles all ambiguous messages directly (don't route when uncertain)
- Define clear "trigger phrases" per agent (explicit routing rules, not vibes-based)
- Allow York to route to multiple agents when needed (fan-out pattern)
- Keep the number of agents small initially — every agent adds routing complexity
- Routing errors should be a tracked category in the Improvement Analyst

---

## 3. Cascading Service Failures

**The problem:** york-data becomes a single point of failure. If it's down, every agent that touches data is broken. Currently, each agent talks to Google Sheets independently — one failure doesn't cascade.

**Failure scenarios:**
- york-data MCP server crashes → all consumption logging, weight logging, chore tracking stops
- DB file gets corrupted → all historical data gone (SQLite single-file risk)
- york-data is slow/hanging → every agent blocks waiting for responses
- Bug in a york-data function → bad data written by every agent that calls it

**Mitigations to design:**
- Health checks for york-data (agent should detect "service unavailable" and say so, not hallucinate data)
- DB backups (daily automated backup of the SQLite/whatever file)
- Graceful degradation: agents should be able to say "data service is unavailable, I'll note this for later" instead of failing silently
- Error handling in the MCP server: bad parameters → clear error message, not silent data corruption
- Consider: does york-data need to be an always-running server, or can it be spawned on-demand per call? On-demand = no persistent failure state

---

## 4. Cold Start Token Overhead

**The problem:** Each agent spawn loads system prompt + workspace files + skill files. With 15-20 agents, the total token cost of cold starts adds up fast. A simple "log my food" could cost 5-10K input tokens just for the agent to boot.

**Current state:** York's context is ~20K+ tokens (SOUL.md, USER.md, AGENTS.md, HEARTBEAT.md, daily memory x2, skill files). Sub-agents inherit some of this.

**Why it matters for cost:** On API billing (not flat rate), each spawn is a cost event. If James says 5 things in a row, and each routes to a different agent, that's 5 cold starts. On Claude Max this doesn't matter, but James is moving to API billing.

**Mitigations to design:**
- Small-tier agents should have MINIMAL context. A nutrition logger needs: schema, function signatures, and the task. NOT James's full bio, D&D info, house layout.
- Persistent agent sessions for high-frequency domains (Health might stay warm during the day)
- Batch related operations: if James logs 3 food items, send all 3 to one agent spawn, not three separate spawns
- Track cost-per-agent to find the expensive ones and optimize their context

---

## 5. Memory Fragmentation

**The problem:** If each agent has its own workspace/memory, the system loses the unified "what happened today" view that makes York useful. Today, everything writes to `memory/YYYY-MM-DD.md`. With 15 agents, does each write to its own file? Does the morning brief read 15 files?

**Specific concerns:**
- How does the weekly review agent know what the chores agent did this week?
- How does York know that the health agent logged a workout when James asks "what did I do today?"
- If agents have isolated workspaces, how does cross-domain reasoning work?

**Mitigations to design:**
- york-data IS the shared memory for structured data. Any agent can query it.
- For unstructured context (conversations, decisions, corrections): either a shared daily memory file that all agents append to, OR agents write to york-data as structured events (`log_event(agent, type, summary)`)
- Morning brief agent queries york-data for yesterday's aggregated data, doesn't read files from other agents
- York (orchestrator) maintains the narrative memory — daily notes about what James said, decisions made, context. Other agents don't write narrative memory.

---

## 6. Sub-Agent Quality is the #1 Risk

**The problem:** This isn't hypothetical. Sub-agent quality failures are already our biggest issue. The multi-agent architecture makes EVERYTHING a sub-agent call. If sub-agent quality doesn't improve, we're scaling our worst failure mode.

**Current evidence (from mistake journal):**
- Sub-agents ignore explicit skill file instructions (meal system)
- Sub-agents cause collateral damage (spreadsheet corruption Mar 4)
- Sub-agents hardcode values instead of using parameters (noon timestamps Mar 1)
- Sub-agents claim data is missing without checking (false absence claims, recurring)

**Why multi-agent amplifies this:** Today, York handles ~40% of work inline (direct conversation, quick lookups). In multi-agent, ~90%+ of work goes through agent spawns. Every failure mode we see in sub-agents today becomes the default mode of operation.

**Mitigations to design:**
- york-data service layer is the biggest single mitigation — sub-agents can't mess up column offsets if they call typed functions
- Each agent's SKILL.md must include concrete anti-examples ("DO NOT do this, here's what it looks like when you mess up")
- Verification gates: York should spot-check sub-agent output before presenting to James (at least for high-stakes operations like data writes)
- Small-tier agents should have extremely narrow scope — one or two functions, crystal clear instructions, minimal room for "creative" interpretation

---

## 7. The Improvement Analyst Bootstrapping Problem

**The problem:** The Improvement Analyst is supposed to fix the system. But it's also part of the system. If it has bugs, who fixes those? If it misses a pattern, who notices?

**Specific risks:**
- Analyst reads session transcripts but misses conversational corrections (they're natural language, not structured)
- Analyst identifies a pattern but the experiment it runs doesn't actually test the hypothesis
- Analyst "verifies" a fix by checking the wrong metric
- Analyst gets stuck in a loop (identifies same pattern repeatedly, applies same ineffective fix)

**Mitigations to design:**
- James IS the ultimate verifier. The weekly review should surface what the Analyst did and whether it helped. James provides the ground truth.
- Track fix effectiveness quantitatively: "This fix was applied on date X. In the 7 days before, this error occurred N times. In the 7 days after, it occurred M times."
- Analyst should have a recurrence check: if the same pattern appears 3+ times after a fix was applied, the fix didn't work. Try something different.
- Lock the Analyst to Large tier (Opus). This is the one place you absolutely cannot use a cheap model — it's reasoning about system behavior across sessions.

---

## 8. Premature Agent Splitting

**The problem:** Starting with 15-20 agents when 5-8 would cover everything. More agents = more routing decisions, more context handoffs, more configuration, more things to go wrong. Each agent has setup/maintenance overhead (SKILL.md, SOUL.md, workspace config, heartbeat config).

**James's own framing supports this:** He said some agents are "It's Complicated" because they span multiple complexity levels. But "splitting them into the right pieces" is a design problem that needs careful thought, not just "make more agents."

**Mitigations to design:**
- Start with the minimum viable agent count. York + Health + Chores + DnD + Analyst = 5. Add more only when a domain demonstrably needs isolation.
- The first migration should be york-data (service layer), not agent splitting. Data reliability is the #1 issue. Agent splitting is #2.
- Measure before splitting: which domains actually cause context pollution? If D&D context never interferes with nutrition logging in practice, splitting them is premature.
- Each new agent should be justified by a specific failure mode it prevents, not by architectural elegance.

---

## 9. Testing & Verification Gap

**The problem:** How do we know the new system works? There's no test harness, no smoke tests, no way to verify a migration phase worked before moving to the next one.

**Specific gaps:**
- How do you test that york-data correctly logs food? Manual verification? Automated tests?
- How do you test that routing works? Send test messages and check they reach the right agent?
- How do you verify data migration didn't lose anything? Row-by-row comparison?
- How do you test that the Analyst's observation loop actually fires and produces output?

**Mitigations to design:**
- york-data should have a test mode: `york-data.test_connection()`, `york-data.validate_schema()`
- Each migration phase needs acceptance criteria defined before starting, not after
- "James doesn't have to correct me for a week" is a valid acceptance criterion
- Data migration: write a comparison script that reads both Sheets and local DB, diffs them
- Consider: nightly smoke test that exercises each agent's basic function and reports failures

---

## 10. The "Build Spam" Problem Persists

**The problem:** When agents do multi-step work in a Discord conversation session, every intermediate thought becomes a visible message. This is already our longest-unfixed issue (identified Feb 21, still happening Mar 7 — 15 messages of narration during architecture work).

**Why it gets worse:** Multi-agent means MORE sub-agent activity, potentially more narration. If the Health agent logs food and narrates "Now checking existing entries... Now appending to the sheet... Now updating the dashboard...", that's 3 messages for what should be silent.

**Mitigations to design:**
- Sub-agents should NEVER post to Discord directly. They return results to York. York posts the summary.
- This is already an architectural rule but needs to be enforced: sub-agents don't have Discord write access.
- Exception: agents that own a channel (DnD in #dnd) can post directly to their channel.
- York should post ONE message per user interaction, not stream its work.

---

## Summary: Priority Order for Design

Based on risk and impact:

1. **york-data service layer** — eliminates the #1 failure category (data operations). Must be designed and tested before any agent splitting.
2. **Sub-agent quality patterns** — anti-examples, narrow scope, verification gates. Applicable to current system AND future agents.
3. **Routing rules** — clear, explicit, testable. Not vibes-based.
4. **Memory model** — how agents share context without fragmenting the daily narrative.
5. **Agent count** — start small, split when justified by data.
6. **Testing** — acceptance criteria for each phase, smoke tests for critical paths.
7. **Cascading failure handling** — health checks, backups, graceful degradation.

---

*Written during overnight work. For James's review during the March 8 architecture session.*
