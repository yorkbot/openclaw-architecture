# OpenClaw Multi-Agent Architecture — Design Brief

**Author:** York
**Date:** 2026-03-07
**Deadline:** March 20, 2026
**Status:** Design phase — active feedback loop with James

---

## 1. Project Goal

Migrate from a monolithic single-agent York (one fat context, one model, every skill loaded) to a multi-agent architecture where 15-20 specialized agents run with focused contexts, appropriate model tiers, and a shared service layer for data access.

**Why now:** James is leaving the $200/mo Claude Max flat plan. The switch to API billing makes waste visible, but the real driver is performance. The current setup loads every skill into every session. A chore check carries D&D wiki context. A nutrition log carries work prep context. Focused agents do better work with less noise.

**Target hardware:** Raspberry Pi 5 (8GB). Everything is API-bound orchestration, not local compute. A year of capability growth (finance, deeper health tracking, cooking patterns, home maintenance) won't change this. Fallback: mini PC (~$150) if local inference or heavy camera processing ever becomes needed.

**What success looks like:**
- Each agent has only the context it needs
- Self-improvement is embedded in every agent, not bolted on
- Data access goes through typed functions, never raw DB/API
- James never needs to look at spreadsheets again
- The system can grow new domains (finance, car, cooking) without redesigning the core

---

## 2. Decisions Made

### 2.1 York as Large-tier orchestrator

York stays as the main agent but sheds execution weight. Its job:
- **Conversation and judgment** — James talks to York, York routes to specialists or handles directly
- **Cannabis gate** — requires cross-domain awareness (chores, diet, movement, time of day). This is inherently orchestrator-level because it synthesizes data from multiple domains before making a judgment call
- **Memory and continuity** — session context, daily memory files, knowing what happened today
- **Routing** — deciding which agent handles a request, spawning sub-agents with appropriate model tiers

**Why Large:** York makes judgment calls that require understanding James's patterns, priorities, and current state. A smaller model would make "crazy choices" (James's words) on routing and gate decisions. The orchestrator touches everything; its mistakes cascade.

### 2.2 D&D as Large-tier agent

The D&D wiki is deeply interconnected. A question about one NPC can require understanding faction relationships, historical events, geographic context, and prior session logs. The wiki has dozens of articles that cross-reference each other.

**Why Large, not Medium:** James flagged this directly: "D&D might be opus... just the sheer size of the wiki." It's not just retrieval — it's reasoning about connections between lore elements, maintaining consistency across worldbuilding decisions, and generating content that fits an established world. Medium-tier models lose coherence across that much context.

**Workspace:** TBD (see Open Questions). At minimum: the wiki, session logs, and worldbuilding tools. May need its own channel routing (messages in a D&D context go directly to the D&D agent rather than proxying through York).

### 2.3 Self-improvement as first-class system

This is not a feature. It's THE core capability.

James: *"If the base architecture is correct and the self improvement works well, literally all other skills are secondary."*

The current improvement queue is broken. It forgets what's done and what's not. It doesn't write ideas when asked. It's a flat markdown file read by amnesiac sessions. The replacement:

**Observation layer** — embedded in every agent, not a separate process. Every agent logs friction (corrections from James, failed operations, unexpected behavior) as a reflex, not as a separate task. This is distributed data collection.

**Improvement Analyst** — Large-tier agent that replaces the overnight worker. Scans friction logs across all agents, identifies patterns, runs experiments, and verifies fixes actually worked. The verification step is the critical missing piece in the current system.

**Feedback loop:** Observe → Analyze → Experiment → Verify → Observe. If a fix doesn't reduce recurrence of the pattern that triggered it, the analyst knows and tries something different.

### 2.4 Service layer (york-data MCP)

The most important architectural decision. Without this, multi-agent is "the same problem as before but with extra steps" (James).

**What it is:** A custom MCP server that sits between agents and data. Agents call typed functions like `log_food(date, item, calories, protein)`. The service validates types, enforces schema, and writes to the DB. The LLM never sees columns, SQL, JSON blobs, or raw API parameters.

**Why it matters:**
- Current failure mode: LLMs construct Google Sheets API calls with wrong column offsets, wrong parameter names (`data` not `values`, underscores not camelCase), wrong ranges. Each skill re-documents the same gotchas. James only looks at spreadsheets "because you fuck up all the time."
- With typed functions: the LLM says "log this food." The service knows exactly which columns, which sheet, which format. Schema enforcement happens at the service level, not in the prompt.
- JSON in LLM prompts is "the worst" (James). Named parameters in, structured responses out. The service handles serialization.

**Implementation:** Custom MCP server replacing google-sheets MCP. Same mcporter call flow, completely different reliability. Agents use `mcporter call york-data.log_food ...` instead of constructing sheet ranges.

### 2.5 Local DB replaces Google Sheets for owned data

Data York owns (consumption, weight, workouts, chores, projects) moves to a local database. Google Sheets is demoted to an ad-hoc tool, not a primary data store.

**DB engine:** TBD. James explicitly said to leave this open. SQLite is the obvious boring choice (single file, no server process, JSON column support, already on Pi), but the service layer makes the engine swappable. The service defines the API; the storage backend is an implementation detail.

**What stays external:**
- Google Calendar — it's the source of truth for schedule data, shared with Audrey, syncs with work. No reason to replicate.
- Memory files and wiki — markdown files work. They're read by agents, not queried relationally.
- Camera feeds — hardware integration, not a DB concern.

### 2.6 Communication channels: Discord only (for now)

Currently all interaction is through Discord server channels. No DMs. The architecture supports future channels but doesn't design for them yet.

Channel routing in multi-agent:
- `#general` — York (orchestrator). Most conversation starts here.
- Domain-specific channels (e.g., `#health`, `#dnd`) — TBD whether these route directly to domain agents or York mediates. See Open Questions.

### 2.7 MTG as separate OpenClaw instance

MTG is self-contained. It doesn't feed into morning briefs, weekly reviews, or health tracking. It's a hobby tool James toggles on when he wants it. Separate instance = separate config, separate model, zero context bleed.

Already running this way (main + mtg as proof of concept).

### 2.8 Tiers, not model names

Agents are assigned complexity tiers, not model names:
- **Script** — no LLM, just code (weather cache, calendar cache)
- **Small** — simplest model available (nutrition logging, workout logging, daily avatar)
- **Medium** — everyday tasks (morning brief, meal suggestions)
- **Large** — complex reasoning (York orchestrator, D&D, weekly review, improvement analyst)
- **XL** — building/analytics/coding agent level (on-demand only)

James: *"Remove Opus and Sonnet references, use sizes."* Model names are implementation details that change. Tiers express what the task needs. The mapping from tier to model/provider is a config decision made once and updated as pricing and capabilities shift.

### 2.9 Performance motivated, not cost motivated

James: *"I'm more performance motivated than cost motivated."*

The multi-agent split isn't about saving money. It's about giving each agent only the context it needs so it performs better. A nutrition logger doesn't need D&D wiki context. A chore checker doesn't need work prep history.

The cost savings are a side effect. Focused contexts mean smaller prompts, which means cheaper calls AND better outputs. But if a task needs Large tier to perform well, it gets Large tier. Don't penny-pinch on model selection.

### 2.10 Work prep likely removed

James's company is providing general-purpose agents (beyond Devin). The work domain will probably disappear from York entirely. Don't design around it. If it stays, it's a separate concern routed through York on demand.

---

## 3. James's Explicit Feedback

Verbatim concerns and directives from the March 7 design session. These are constraints, not suggestions.

### On model references
> "Remove Opus and Sonnet references, use sizes."

### On D&D complexity
> "D&D might be opus... just the sheer size of the wiki."

### On motivation
> "I'm more performance motivated than cost motivated."

### On self-improvement priority
> "If the base architecture is correct and the self improvement works well, literally all other skills are secondary."

Self-improvement needs to be "first class" and span everything.

### On the improvement queue
> Improvement queue is "a big miss" — forgets done/not-done, doesn't write ideas when asked. "I wanted to give you a path for self improvement, but it is a big miss so far."

### On removing assumptions
> "Go a step deeper for most things because I've seen you make some crazy choices so I want to comprehensively remove assumptions."

### On JSON and LLMs
> JSON is "the worst in combination of an LLM."

### On the service layer
> "Otherwise we just have the same problem as before but with extra steps."

This is why the service layer exists. Swapping Google Sheets for a local DB without changing how agents interact with data solves nothing. The typed function interface is the fix.

### On DB choice
> "Why SQLite over literally anything else? Leave it an open question."

### On spreadsheet access
> Only looks at spreadsheets "because you fuck up all the time."

This is the clearest validation of the service layer approach. If data operations become reliable, James doesn't need direct spreadsheet access at all.

### On planning depth
> "Every time I go into a project without planning with you we hit a wall. This one I'm going to over-plan since it's so fundamental."
> "I believe it's the right thing to do and will work."

---

## 4. Open Questions

These need resolution before implementation. Ordered roughly by dependency (earlier questions block later ones).

### 4.1 DB engine choice

**Options:** SQLite, LevelDB, PouchDB, other.
**Context:** Service layer makes this swappable. The engine needs to run on a Pi 5 with no external server process. Relational is likely better than document for this data (consumption, weight, workouts all have fixed schemas), but the service layer enforces schema regardless.
**James's position:** Leave it open. Don't assume SQLite.
**To resolve:** Research lightweight DB options, benchmark on Pi, pick based on actual requirements not assumptions.

### 4.2 york-data function inventory

**Question:** What typed functions does the service layer need?
**Approach:** Audit every current Google Sheets operation across all skills. Map each to a typed function signature. This gives the exact API surface.
**Known operations:** log_food, log_weight, log_workout, get_dashboard, update_dashboard, get_consumption(date_range), get_weight_history, get_exercises, log_chore, get_chore_status, log_project, get_projects.
**Unknown:** What operations are we missing? What does the weekly review actually query? What does the morning brief cache need?

### 4.3 Data migration (Sheets → local DB)

**Question:** How does historical data move from Google Sheets to the local DB?
**Sub-questions:**
- Do we migrate everything or start fresh with local DB?
- Is historical data even needed? (Weight trend yes, individual meal logs from February maybe not)
- One-time script or gradual migration?
- What happens during the transition? Dual-write?

### 4.4 Agent discovery

**Question:** How do agents know about each other?
**Context:** OpenClaw's agents.list config defines all agents. But does each agent need to know what other agents exist and what they handle? Or does only the orchestrator (York) know the routing table?
**Likely answer:** Only York needs the full map. Other agents don't route to each other; they return results to York, who routes onward if needed. But this needs to be explicit.

### 4.5 D&D agent workspace boundary

**Question:** What does the D&D agent own?
**Candidates:** Wiki articles, session logs, worldbuilding tools, character sheets, encounter prep, loot tables, random generators.
**Sub-questions:**
- Does it have its own memory files or share York's?
- Does it write to the morning brief cache (D&D questions for James)?
- Does it handle image generation for D&D art or does it route to an image-gen agent?
- Channel routing: does `#general` D&D discussion go to D&D agent directly or through York?

### 4.6 Health channel routing

**Question:** Does `#health` need to exist as a Discord channel, or does York route health sub-agents internally?
**Context:** Health data is private (never in group contexts). A dedicated channel gives a clean routing target but adds channel sprawl. York mediating keeps the channel count low but adds a proxy hop.
**James's privacy rule:** Health data never in server channels. If `#health` exists, it needs to be truly private.

### 4.7 Self-improvement observation embedding

**Question:** How does the observation layer actually get embedded in every agent?
**Options:**
- Shared prompt fragment injected into every agent's system prompt
- Post-session hook that scans conversation for corrections/friction
- Agent-specific SOUL.md section that each agent's prompt references
- Combination: prompt fragment for awareness + post-session hook for capture
**Constraint:** Must not add significant context overhead to small-tier agents. A nutrition logger shouldn't carry 500 tokens of self-improvement instructions.

### 4.8 Cross-agent data access

**Question:** What happens when an agent needs data from another agent's domain?
**Example:** Weekly review agent needs nutrition data (health domain), chore completion (home domain), and cannabis session count (gate domain). Does it query york-data directly, or does York aggregate and pass context?
**Likely answer:** york-data is shared infrastructure. Any agent with appropriate permissions can call its functions. Domain boundaries are about workspace/prompt context, not data access.

### 4.9 Morning brief data access

**Question:** How does the morning brief agent access cached data?
**Current:** Cron jobs write to daily memory files. Morning orchestrator reads those files.
**Future:** If cached data lives in the DB, the morning brief agent calls york-data functions instead of reading files. But some data (weather, calendar) is ephemeral and might not belong in the DB.
**Decision needed:** What's cached in DB vs. what stays in memory files?

### 4.10 Camera/chore integration

**Question:** Does the chores agent need its own camera access, or does York mediate?
**Context:** Cannabis gate sometimes requests room photos. Currently York handles this directly. If chores becomes a separate agent, does it need node camera access, or does York take the photo and pass it to the chores agent?
**Likely answer:** York mediates hardware. Chores agent receives images as context, doesn't control cameras directly.

### 4.11 Panel presence ownership

**Question:** Does the desktop panel stay in York's heartbeat or become its own thing?
**Current:** York updates `panel/update-status.sh` in every heartbeat and after meaningful conversations.
**Options:** Keep in York (it's self-expression, inherently tied to the orchestrator's awareness), or split to a dedicated micro-agent that reflects system state.
**Likely answer:** Keep in York. The panel IS York's voice.

### 4.12 Finance/YNAB integration

**Question:** When does finance get added, and how?
**Context:** James used Claude Opus pre-OpenClaw for 2025 transaction audit + 2026 budget plan via YNAB MCP. He'll need it again. Natural integration with weekly review (spending + health cross-domain patterns).
**Decision needed:** Is finance a domain agent or a york-data service? Probably a Medium-tier agent that calls YNAB MCP + york-data for cross-domain analysis.

### 4.13 Car maintenance

**Question:** James has a handwritten note to share about car maintenance schedule. Where does this live?
**Options:** york-data service (structured maintenance schedule), calendar events, home domain agent, or a dedicated vehicle agent.
**Blocked on:** James sharing the note.

### 4.14 Transition period

**Question:** How do we handle running both systems during migration?
**Options:**
- Hard cutover (risky, everything breaks at once)
- Parallel run (dual-write to Sheets + DB, compare results)
- Phased migration (one domain at a time, starting with lowest-risk)
**Likely answer:** Phased. Migrate nutrition logging first (high frequency, simple schema, most error-prone currently). Verify it works. Then expand.

---

## 5. Current State of Diagram

Live at: https://yorkbot.github.io/openclaw-architecture/

The Vue3 app has 5 tabs built during the March 7 design session:

### Tab 1: System Overview
- Shows the full agent topology: York orchestrator at center, specialized agents radiating out
- Service layer (york-data MCP) between agents and data sources
- Callout: "Intentional, not cheap" — focused contexts improve both performance and cost
- Scripts (no LLM): weather-cache, calendar-cache
- External services: Google Calendar, camera/nodes
- Local DB (engine TBD) for owned data
- Google Sheets shown as demoted/ad-hoc

### Tab 2: Agent Details
- Table of all agents with complexity tiers, responsibilities, and notes
- Grouped by tier (Script → Small → Medium → Large)
- Includes current status (exists, needs splitting, new)

### Tab 3: Self-Improvement
- Full feedback loop: Observe → Analyze → Experiment → Verify
- Observation layer shown spanning all agents
- Improvement Analyst as dedicated Large-tier agent
- Friction journal replacing improvement queue

### Tab 4: Communication Channels
- Discord channel routing map
- Which agent owns which channel
- Privacy rules for health data

### Tab 5: Data Flow
- Shows data paths: agent → york-data service → local DB
- Cannabis gate flow (multi-domain synthesis)
- Nutrition cache flow (cron → DB → morning brief)
- Weekly review flow (cross-domain aggregation)
- External service flows (Google Calendar, cameras)

---

## 6. What's NOT in the Diagram Yet

### 6.1 Service layer detail

The diagram shows york-data as a box. It doesn't show:
- The full function inventory (what typed functions exist)
- Request/response schemas for each function
- Error handling (what happens when a function call has bad parameters)
- Authentication/authorization (do all agents get all functions?)
- Versioning (how do function signatures evolve?)

### 6.2 Data migration plan

No visualization of:
- Which data moves when
- The dual-write/verification period
- Rollback strategy if migration goes wrong
- Historical data handling (migrate vs. archive vs. ignore)

### 6.3 Future domains

Not represented yet:
- **Finance** — YNAB MCP integration, spending patterns, budget tracking
- **Car maintenance** — scheduled maintenance, repair history, cost tracking
- **Cooking patterns** — 6+ months of consumption data could feed recipe suggestions, ingredient frequency analysis, meal planning
- **Deeper health** — sleep tracking (Bangle.js), activity/HR correlation, phlebotomy scheduling
- **Life dashboard** — cross-domain weekly/monthly synthesis (weight + spending + sleep + work stress + consumption patterns)

### 6.4 Self-improvement observation in practice

The diagram shows the feedback loop conceptually. It doesn't show:
- Exactly what the observation prompt fragment looks like
- How friction gets logged (file? DB? structured format?)
- How the improvement analyst discovers friction logs from all agents
- What "verify a fix worked" means concretely (recurrence rate? James satisfaction?)
- How improvements propagate (change a skill file? update a prompt? modify a cron?)

### 6.5 Agent workspace boundaries

Not defined for any agent:
- What files does each agent read/write?
- Do agents share memory files or have isolated workspaces?
- Where does each agent's skill config live?
- What happens if two agents try to write the same file?

### 6.6 Error handling

No coverage of:
- What happens when york-data is unreachable
- What happens when an agent fails mid-task
- How the orchestrator knows a sub-agent failed vs. is still working
- Retry logic, fallback behavior, graceful degradation
- Alerting (does York notify James of failures? When? How?)

---

## 7. Implementation Principles

Derived from James's feedback and the design session. These aren't open questions; they're constraints.

1. **No assumptions.** Go a step deeper on every design choice. If the reasoning isn't documented, the choice isn't made.
2. **Typed functions, never raw access.** LLMs call named functions with named parameters. They never construct SQL, JSON payloads, or API calls directly.
3. **Tiers, not models.** Design documents reference Small/Medium/Large/XL. Model mapping is a separate config concern.
4. **Self-improvement is the product.** Everything else is a feature. If the improvement system works, features fix themselves over time.
5. **Service layer is the fix.** Swapping backends without changing the interaction pattern solves nothing.
6. **Verify, don't assume.** Every fix gets checked. Every migration gets validated. Every claim gets tested.
7. **Phase the migration.** No big bang. Each phase is independently valuable. If we stop after phase 1, things are still better than before.
8. **Hardware isn't the bottleneck.** Everything is API orchestration. Don't design around compute constraints that don't exist.

---

## 8. Next Steps

1. **James reviews this brief** — confirms decisions, adds missing context, resolves accessible open questions
2. **Audit current operations** — pull real usage data: sub-agent spawn frequency, cron success/failure rates, skill invocation patterns, friction points
3. **Define york-data function inventory** — map every spreadsheet operation to a typed function
4. **Resolve DB engine** — research, benchmark on Pi, pick one
5. **Design the observation layer** — concrete specification for how agents log friction and how the analyst consumes it
6. **Phase 1 implementation plan** — likely nutrition logging through york-data as the first migration target
7. **Update diagram** — add service layer detail, workspace boundaries, error handling

---

## 9. york-data Function Inventory (Audited from Current Usage)

Derived from auditing every `mcporter call google-sheets.*` across all current skills. These are the actual operations York performs today, mapped to typed functions.

### 9.1 Current Spreadsheet Schemas

**Fitness Spreadsheet** (`1_3GtsFhX3xmFjTZau4ZXVZcMlKRqgjZGR9kszQn1ZSw`):

| Sheet | Columns | Used By |
|-------|---------|---------|
| Dashboard (summary) | Start Weight, Goal Weight, Target Date, Current Weight, Lost So Far, To Go | nutrition-weight-cache, morning-brief |
| Dashboard (daily log) | Date, Weight, Calories, Protein (g), Kitchen Closed, Wake Up, Bedtime, Sleep (hrs), Energy (1-5) | york-consumption, nutrition-weight-cache, morning-orchestrator, weekly-checkin |
| [Month] Consumption | DateTime, Item, Quantity, Calories, Protein, Notes | york-consumption, nutrition-summary, consumption-gap, meal-system, morning-brief |
| Workouts | Date, Type, Location, Duration, Notes, (col F) | york-workout, weekly-checkin |
| Exercises | Date, Exercise, Sets, Reps, Weight, Notes | york-workout |

**Chores Spreadsheet** (`1syEgiOncEgoxJf6DHROe0lmU3sT_hbeeGBREeAnuWfQ`):

| Sheet | Columns | Used By |
|-------|---------|---------|
| Sheet1 | Task, Frequency, Last Completed, ... | york-chores, morning-orchestrator |

**Home Projects Spreadsheet** (`1kKg_Esyo12tSmSaa5ZUzDU4GtWQYfso1Dug2ShcdXbg`):

| Sheet | Columns | Used By |
|-------|---------|---------|
| Sheet1 | Task, Zone(s), Priority, ... | morning-brief |

### 9.2 Proposed Typed Functions

**Consumption (highest frequency, most error-prone)**
- `log_food(date, time, item, quantity, calories, protein, notes?)` — append to consumption log
- `get_consumption(date)` → all items for a date
- `get_consumption_range(start_date, end_date)` → items across date range
- `delete_consumption(id)` — remove a mis-logged entry
- `update_consumption(id, fields...)` — fix a specific entry

**Dashboard / Daily Metrics**
- `log_daily(date, weight?, calories?, protein?, kitchen_closed?, wake_up?, bedtime?, sleep_hrs?, energy?)` — upsert daily row
- `get_daily(date)` → single day's metrics
- `get_daily_range(start_date, end_date)` → range of daily metrics
- `get_weight_trend(days)` → weight entries with moving average

**Workouts**
- `log_workout(date, type, location, duration, notes?)` — add workout summary
- `log_exercises(date, exercises[{name, sets, reps, weight, notes?}])` — add exercise details
- `get_workouts(start_date?, end_date?)` → workout history
- `get_exercises(start_date?, end_date?)` → exercise details
- `get_last_workout()` → most recent workout date and details

**Chores**
- `get_chores()` → all chores with frequency and last-completed
- `log_chore(task, completed_date)` — mark a chore done
- `get_chore_status()` → what's due, what's overdue

**Home Projects**
- `get_projects()` → all projects with priority and status
- `update_project(task, fields...)` — update status/priority

**Cannabis (currently embedded in consumption, could be its own domain)**
- `log_cannabis(date, time, session_number)` — record a session
- `get_cannabis_sessions(date?)` → sessions for a date
- `get_cannabis_history(days)` → frequency and pattern data

**Cross-Domain Queries (for weekly review, pattern analysis)**
- `get_trends(days, domains[])` → combined view of weight, calories, protein, workouts, cannabis over N days
- `get_day_summary(date)` → everything that happened on a specific day

### 9.3 Operations That DON'T Move to york-data

These stay as separate tools/services:
- **Google Calendar** — `mcporter call google-calendar.list-events` stays as-is. Calendar is external source of truth.
- **Memory files** — still markdown, still read/written directly by agents.
- **D&D wiki** — markdown files, managed by DnD agent.
- **Camera snapshots** — ffmpeg scripts, hardware integration.
- **Discord** — message tool, stays as-is.
- **Panel/genmon** — status.json, stays as file write.

### 9.4 Gotchas the Service Layer Eliminates

Problems observed in the current system that typed functions would prevent:
1. **Wrong column offset** — LLM writes calories to protein column or vice versa
2. **Month sheet name** — LLM uses "Feb Consumption" instead of "February Consumption"
3. **Range calculation** — LLM guesses next empty row, sometimes overwrites existing data
4. **Parameter naming** — `data` vs `values`, `spreadsheet_id` vs `spreadsheetId`
5. **Duplicate entries** — LLM doesn't check if entry already exists before writing
6. **Dashboard row drift** — LLM counts rows wrong when dashboard has gaps
7. **Date format inconsistency** — "Mar 5" vs "March 5" vs "2026-03-05" vs "Mar 5, 2026"
8. **Calorie/protein in wrong units** — service can enforce reasonable bounds (0-5000 cal, 0-500g protein)

---

## 10. Friction Audit (Known Failure Patterns)

Patterns observed in the current system that the new architecture should eliminate:

### 10.1 Data Operations
- Consumption logging is the #1 source of corrections from James
- Dashboard updates frequently write to wrong row
- Month name construction fails (hardcoded "February" when it's March)
- Sub-agents clear existing data and rewrite instead of appending
- Calorie estimates for restaurant meals wrong ~60% of the time (model issue, not architecture)

### 10.2 Context Overload
- Every session loads SOUL.md, USER.md, AGENTS.md, HEARTBEAT.md, today's memory, yesterday's memory
- Sub-agents inherit too much context (consumption logger doesn't need D&D wiki awareness)
- Heartbeat sessions load full workspace but do nothing with it 90%+ of the time

### 10.3 Self-Improvement
- Improvement queue items get re-suggested after rejection (no rejection memory)
- "Done" items get re-opened (no verification)
- Ideas discussed in conversation never make it to the queue
- Heartbeats fail to pick up explicit TODO items from memory files (today's failure)
- No feedback loop: fixes are never verified to have worked

### 10.4 Coordination
- Memory flushes during conversation block the chat (5-min Discord timeout)
- Sub-agents sometimes work on stale data (cron cache not yet refreshed)
- Multiple sub-agents can write to the same memory file with conflicts
- Heartbeat doesn't carry over context about what was said earlier today

---

*This document is the single source of truth for the multi-agent architecture project. Updated as decisions are made and questions are resolved.*
