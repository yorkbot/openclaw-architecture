# Database Engine Research — york-data Service Layer

**Date:** 2026-03-07
**Context:** Open Question 4.1 from DESIGN-BRIEF.md
**Target:** Embedded DB for Raspberry Pi 5 (8GB), no server process, Node.js MCP server

---

## Requirements

1. **Embedded** — no separate server process, library-level integration
2. **ARM64 compatible** — must run on RPi 5
3. **Node.js bindings** — york-data MCP server will be Node.js (same runtime as OpenClaw)
4. **Relational queries** — JOINs, GROUP BY, aggregates (SUM calories by date, weight trends)
5. **Schema enforcement** — fixed schemas for consumption, daily metrics, workouts, chores
6. **Backup = file copy** — simple disaster recovery
7. **Low data volume** — ~2,000 rows/month across all tables. This is not a scale problem.
8. **JSON support** — nice-to-have for flexible fields, not required if schema covers everything

---

## Options Evaluated

### 1. Node.js Built-in SQLite (`node:sqlite`)

**Available since Node 22.** Our system runs Node 25.4.0.

**Tested:** ✅ Works on this system. Full schema with 6 tables, INSERT, SELECT, JOINs, GROUP BY, json_extract all work. SQLite 3.51.1 via Node, system SQLite 3.51.2.

**Pros:**
- Zero dependencies. No npm install, no native compilation, no node-gyp issues on ARM64
- Synchronous API (`DatabaseSync`) — simpler MCP server code, no async complexity
- SQLite is the most deployed database engine in the world. Battle-tested.
- Single file, backup = `cp york-data.db york-data.db.bak`
- Full SQL: JOINs, CTEs, window functions, JSON1 extension built in
- Schema enforcement via column types + CHECK constraints
- WAL mode for concurrent reads (multiple agents reading simultaneously)
- Node.js maintains it — updates come with Node upgrades, not a separate dependency

**Cons:**
- Synchronous API blocks event loop (irrelevant — MCP calls are sequential, data volume is tiny)
- No built-in replication (irrelevant — single Pi, single user)
- SQL, not document — if we wanted schemaless flexibility (James said we don't)

**Verdict:** The boring-correct answer. Zero dependency overhead, proven reliability, already available.

### 2. better-sqlite3 (npm package)

**What:** Most popular Node.js SQLite binding. Synchronous, fast, well-maintained.

**Pros:**
- Mature, widely used, excellent documentation
- Synchronous API (same benefit as built-in)
- Slightly faster than Node built-in for some operations (C++ addon)

**Cons:**
- Native addon — requires node-gyp compilation. Can be tricky on ARM64/RPi.
- Extra dependency to maintain
- Node.js built-in SQLite makes this redundant for our use case

**Verdict:** No advantage over built-in `node:sqlite`. Skip.

### 3. DuckDB

**What:** Analytical/OLAP database, columnar storage, designed for aggregation queries.

**Pros:**
- Exceptional at analytical queries (weight trends, calorie aggregations, cross-domain analysis)
- Columnar storage is efficient for time-series data
- Can query Parquet, CSV, JSON files directly

**Cons:**
- Heavier runtime than SQLite (~50MB binary)
- Node.js bindings require native compilation (ARM64 support exists but not guaranteed smooth)
- Overkill for our data volume (DuckDB shines at millions of rows, we have thousands)
- Less ecosystem support for simple CRUD operations
- Designed for analytics, not transactional writes

**Verdict:** Would be interesting if we had years of historical data to analyze. At our scale, SQLite handles analytics fine. Pass.

### 4. LevelDB / RocksDB

**What:** Key-value stores from Google/Facebook. LevelDB is simpler, RocksDB is more feature-rich.

**Pros:**
- Fast writes, good for append-heavy workloads
- Embedded, no server process
- Good Node.js bindings (classic-level for LevelDB)

**Cons:**
- Key-value, not relational. Every "query" becomes application-level code.
- No SQL, no JOINs, no GROUP BY. Would have to implement aggregation logic in the service layer.
- "What did James eat on March 5?" becomes a range scan + filter, not a SELECT.
- More code to maintain, more places for bugs. Defeats the purpose of the service layer simplifying things.

**Verdict:** Wrong abstraction level. We need queries, not key-value lookups. Pass.

### 5. LMDB

**What:** Lightning Memory-Mapped Database. Ultra-fast reads, used by Caffe and other ML frameworks.

**Pros:**
- Extremely fast reads (memory-mapped)
- ACID compliant
- Small footprint

**Cons:**
- Same problem as LevelDB: key-value, no SQL
- Node.js bindings less mature (lmdb-js exists but less ecosystem)
- Overkill performance for our data volume

**Verdict:** Same reasoning as LevelDB. We need relational queries. Pass.

### 6. SurrealDB (embedded mode)

**What:** Multi-model database (document, graph, relational). Rust-based. Can run embedded.

**Pros:**
- Flexible query language (SurrealQL) with SQL-like syntax
- Graph queries could model relationships (agent → task → outcome for self-improvement)
- Rust = fast, safe

**Cons:**
- Young project, breaking changes between versions
- Embedded mode Node.js bindings are early-stage
- Learning new query language (SurrealQL) adds cognitive load
- Complexity not justified for our simple schemas
- ARM64 support: works but not first-class focus

**Verdict:** Interesting for future complexity (graph queries for self-improvement patterns). Too risky for a core system right now. Pass.

### 7. libSQL / Turso

**What:** SQLite fork by Turso. Adds: server mode, replication, branching, ALTER TABLE improvements.

**Pros:**
- All SQLite benefits plus modern features
- Turso offers managed hosting (cloud sync if ever wanted)
- Better ALTER TABLE support (can add/drop columns properly)

**Cons:**
- Extra dependency (not built into Node.js)
- Cloud sync features irrelevant (everything stays local on Pi)
- Fork maintenance risk — will it stay compatible with upstream SQLite?
- For our use case, standard SQLite's limitations don't apply (we're not doing schema migrations frequently, data volume is tiny)

**Verdict:** Interesting fork but solves problems we don't have. Standard SQLite is sufficient. Pass.

---

## Recommendation

### Node.js Built-in SQLite (`node:sqlite`)

**Zero dependencies.** This is the decisive factor. No npm install, no native compilation, no ARM64 build issues, no dependency updates. It ships with Node.js.

The york-data MCP server's entire data layer can be:
```javascript
const { DatabaseSync } = require('node:sqlite');
const db = new DatabaseSync('/path/to/york-data.db');
```

That's it. No package.json dependency, no version conflicts, no supply chain risk.

**Schema tested and validated:** 6 tables (consumption, daily_metrics, workouts, exercises, cannabis_sessions, chores) with appropriate types, constraints, and cross-table queries all work.

**What about James's "why SQLite over literally anything else?"**: The answer is that every alternative either (a) adds dependencies we don't need, (b) removes query capabilities we do need (key-value stores), or (c) adds complexity for problems we don't have (distributed systems, millions of rows). The service layer makes the engine swappable if this reasoning changes.

---

## MCP Server Implementation Notes

### How york-data fits into the system

Current flow:
```
Agent → mcporter call google-sheets.update_cells → Google Sheets API → Spreadsheet
```

New flow:
```
Agent → mcporter call york-data.log_food → york-data MCP server → SQLite DB
```

### MCP server setup

The york-data server would be configured in mcporter just like google-sheets:
```json
{
  "name": "york-data",
  "transport": "stdio",
  "command": "node",
  "args": ["/path/to/york-data/server.js"],
  "env": {
    "DB_PATH": "/path/to/york-data.db"
  }
}
```

### Concurrency model

MCP servers are spawned per-mcporter-call via stdio. Each invocation opens the SQLite DB, runs the operation, returns the result, and exits. SQLite in WAL mode handles this safely — multiple readers, single writer, automatic locking.

For our data volume and access patterns (sequential operations from a single agent at a time), there are zero concurrency concerns.

### Backup strategy

```bash
# Daily backup via cron (already have system crontab infrastructure)
cp /path/to/york-data.db /path/to/backups/york-data-$(date +%Y%m%d).db
```

SQLite's `.backup` API is also available if we need hot backups while writers are active, but at our volume a simple file copy during quiet hours is fine.

---

## Open Question: Schema Migrations

When the schema needs to change (new column, new table, renamed field):

**Option A: Migration files.** Numbered SQL scripts (`001-add-energy-column.sql`, `002-create-cannabis-table.sql`). Server checks version on startup, runs pending migrations.

**Option B: Recreate from code.** Schema defined in code, DB is ephemeral + data is imported. Works if we have good export/import tooling.

**Recommendation:** Option A (migration files). Simple, auditable, standard practice. The york-data server checks a `schema_version` table on startup and runs any pending migrations.
