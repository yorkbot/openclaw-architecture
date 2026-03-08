# Usage Data — Actual System Metrics

**Pulled:** 2026-03-07 7:21 PM ET
**Source:** OpenClaw cron/jobs.json, memory files, crontab

---

## 1. Automated Session Volume

| Category | Sessions/Day | Notes |
|----------|-------------|-------|
| Heartbeats | ~33 | Every 30m, 8 AM-12:30 AM. Most are silent (HEARTBEAT_OK). |
| Cron jobs (day) | ~12 | weather-cache ×2, calendar-context ×2, morning crons ×5, evening summary, queue-audit, gap-cache |
| Cron jobs (overnight) | ~12 | overnight-work every 30m, 1-6:30 AM |
| Weekly crons | ~0.3 | weekly-checkin (Sun 8 PM), weekly-banner (Mon 7:40 AM) |
| **Total automated** | **~57** | Not counting interactive sub-agents spawned during conversation |

Interactive sub-agent spawns average 3-5/day on active days (consumption logging, research, architecture work).

**Total daily LLM sessions: ~60-65** (automated + interactive).

---

## 2. Cron Job Details

### Execution Duration

| Job | Last Duration | Timeout | Tier Recommendation |
|-----|--------------|---------|---------------------|
| weather-cache | 20s | 60s | **Script** — curl + file write, no LLM needed |
| calendar-context | 18s | 120s | **Script** — mcporter call + file write |
| compile-overnight | 16s | 120s | Small — session list scan, simple summary |
| queue-audit | 20s | ? | Small — read file, check structure |
| daily-memory-summary | 41s | 120s | Medium — reads session history, synthesizes |
| nutrition-weight-cache | 47s | 120s | Small — mcporter calls, format data |
| consumption-gap-cache | 70s | 240s | Small — read spreadsheet, compare to expected |
| daily-avatar | 89s | 120s | Small — generate image prompt, call API |
| morning-orchestrator | 105s | 180s | Medium — compile cached data, format brief |
| weekly-checkin | 125s | 180s | Large — cross-domain analysis, opinions |
| overnight-work | 144s | 300s | Large — open-ended improvement work |
| dnd-questions-cache | 288s | 600s | Large — wiki exploration, research, synthesis |

### All crons currently run on default model (Opus). Waste is significant.

**Crons that should be pure scripts (zero LLM):**
- weather-cache — `curl wttr.in` + write to file
- calendar-context — `mcporter call google-calendar.list-events` + write to file

**Crons that could run on Small tier:**
- compile-overnight — list sessions, extract results
- queue-audit — structural check on a markdown file
- nutrition-weight-cache — mcporter calls, format numbers
- consumption-gap-cache — compare spreadsheet data to expected entries
- daily-avatar — prompt construction + image API call

**Crons that need Medium tier:**
- daily-memory-summary — synthesizes across multiple sessions
- morning-orchestrator — compiles and formats multiple data sources

**Crons that need Large tier:**
- weekly-checkin — pattern analysis, opinions, cross-domain reasoning
- overnight-work — open-ended problem solving, experimentation
- dnd-questions-cache — deep wiki analysis, creative research questions

---

## 3. Heartbeat Patterns

From Mar 6 log (30 heartbeats recorded):
- **28/30 heartbeats were silent** (HEARTBEAT_OK or no post)
- **2/30 had actionable content** (~7% action rate)
- Most common heartbeat duration: startup checks + panel update = ~15-20s
- Version check + GCal auth check = ~5s combined
- Panel update = ~2s

**Heartbeat cost analysis:** At ~33 heartbeats/day on Opus, most doing nothing but reading memory + running 2 bash scripts. This is the single biggest waste source. Heartbeats should be Small or Medium tier at most — they rarely make complex decisions.

**Exception:** Cannabis gate interactions happen during heartbeats and require cross-domain judgment. But this could be a separate trigger, not embedded in every heartbeat.

---

## 4. System Crontab (non-OpenClaw)

| Schedule | Script | Purpose | Notes |
|----------|--------|---------|-------|
| `0 1 * * *` | reset-heartbeat-state.sh | Daily heartbeat state reset | Script, no LLM |
| `*/5 * * * *` | wifi-presence.sh | WiFi presence detection | Script, no LLM |
| `*/15 * * * *` | cameras/summarize.py | Camera event summary | Script, no LLM |
| `@reboot` | cameras/start-daemon.sh | Camera daemon startup | Script, no LLM |

These are already pure scripts. Good pattern to follow.

---

## 5. Sub-Agent Spawn Patterns (Mar 1-7)

| Day | Spawn Mentions | Context |
|-----|---------------|---------|
| Mar 1 | ~4 | D&D research, consumption logging |
| Mar 2 | 0 | Quiet day |
| Mar 3 | 0 | Quiet day |
| Mar 4 | ~8 | Active conversation day |
| Mar 5 | ~5 | Consumption, camera work |
| Mar 6 | ~2 | Minimal interaction (Audrey visit) |
| Mar 7 | ~11 | Architecture project, weekly checkin, consumption |

**Most common sub-agent tasks:**
1. Consumption logging (nearly daily)
2. D&D research/wiki updates
3. Morning brief compilation
4. Architecture/build work

---

## 6. Cost Implications

### Current (Opus for everything, $200/mo flat)
- ~60-65 sessions/day × 30 days = ~1,800-1,950 sessions/month
- All running Opus regardless of complexity

### Projected with tiering
- **Script (0 cost):** weather-cache, calendar-context = ~60 sessions/month eliminated
- **Small tier:** 7 crons/day × 30 + most heartbeats × 30 = ~1,200 sessions/month on cheapest model
- **Medium tier:** 2 crons/day × 30 + some interactive = ~100 sessions/month
- **Large tier:** 2-3 crons/day × 30 + York orchestrator + interactive = ~200 sessions/month
- **Waste eliminated:** ~65% of current sessions don't need Large tier

---

## 7. Key Finding: Heartbeat Waste

**Heartbeats are 53% of all automated sessions** (33/62 daily) and almost never do anything meaningful. On API billing, this is the #1 optimization target.

Options:
1. **Reduce frequency** — hourly instead of every 30 min (halves cost)
2. **Lower tier** — Small model for routine heartbeats, Large only when cannabis gate or complex decision needed
3. **Split heartbeat** — lightweight "alive check" script (no LLM) that escalates to LLM session only when something needs attention
4. **Event-driven** — instead of polling every 30 min, trigger on Discord messages or cron events

Option 3 (script + escalation) is the architecture-aligned answer. A bash script checks for pending events/messages, only spawns an LLM session if there's something to evaluate.
