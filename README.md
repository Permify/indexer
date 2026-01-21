# Indexer (Go)

Indexer is a high-throughput, incremental permission indexing service written in Go.  
It consumes database change events (Debezium), derives semantic permission mutations, and maintains a Leopard-style precomputed authorization index while filling client-facing permission tables.

The system is designed to eliminate recursive permission checks by continuously compiling raw relationship data into fast, queryable permission sets.

---

## Purpose

Indexer transforms raw database changes into:

- permission relationship diffs (GRANT / REVOKE)
- precomputed indexes
- materialized permission views in client databases
- audit-grade permission change streams

All changes are applied incrementally and are fully replayable from the event log.

---

## Architecture

Postgres / MySQL / Mongo
│
▼
Debezium
│
▼
Kafka / Redpanda Topics
│
▼
┌────────────────────────┐
│   Leopard Indexer     │
│        (Go)           │
│------------------------│
│ • CDC Consumer        │
│ • Relation Mapper    │
│ • Diff Engine        │
│ • Cascade Engine     │
│ • Leopard Writer     │
│ • Client DB Writer   │
└────────────────────────┘
│
├──► Index Store (Redis ...)
│
└──► Client Permission DB

---

## Pre-Requirements

### Infrastructure

- Debezium deployed and streaming CDC events  
- Kafka / Redpanda topics created  
- Index store provisioned  
- Client databases provisioned  

### Data Contracts

- Permission-relevant tables identified  
- Table → relation mappings defined  
- Permify / Zanzibar-style schema versioned  
- CDC payload schema frozen  

### Schema Readiness

- Materialized permission tables created  
- Unique constraints for idempotency  
- Revocation semantics defined  
- Index key formats finalized  

---

## Processing Pipeline

### 1. CDC Ingestion
- Consume Debezium topics  
- Preserve ordering guarantees  
- Track LSN and Kafka offsets  
- Support replay and rebuild  

---

### 2. Normalization
- Parse Debezium envelopes  
- Normalize into canonical change events  
- Validate schemas  
- Attach source metadata  

---

### 3. Relation Mapping
- Convert rows into permission tuples  
- Support direct and derived relations  
- Version all mappers  
- Enforce schema compatibility  

---

### 4. Diff Engine
- Produce semantic permission diffs  
- Treat updates as revoke + grant  
- Drop no-op mutations  
- Deduplicate repeated events  

---

### 5. Cascade Engine (Leopard Core)
- Maintain adjacency views  
- Expand transitive permission effects  
- Generate derived permission changes  
- Prevent cycles and infinite propagation  
- Guarantee convergence  

---

### 6. Index Writer
- Maintain sorted reachability sets  
- Apply atomic batch updates  
- Enforce idempotency  
- Support snapshot + incremental layers  

---

### 7. Client Database Writer
- Persist effective permissions  
- Maintain permission history  
- Upsert and revoke deterministically  
- Enforce constraints  

---

### 8. Consistency & Recovery
- Checkpoint stream offsets  
- Rebuild by replay  
- Validate against baseline snapshots  
- Support full reindex mode  

---

## Post-Conditions (Acceptance Criteria)

### Functional
- Every permission-relevant DB change is reflected in:
  - Leopard index
  - client permission tables
- Indirect and nested permissions are precomputed  
- Revocations cascade correctly  
- No full database rescans required  

---

### Correctness
- Deterministic and idempotent indexing  
- Out-of-order and duplicate CDC events tolerated  
- Index state matches recursive permission evaluation  
- No orphaned permissions  

---

### Performance
- Incremental updates only  
- Bounded cascade depth  
- Batch writes supported  
- Sustains production CDC throughput  

---

### Reliability
- Crash-safe recovery  
- Replay produces identical index  
- Partial failures do not corrupt state  
- Dead-letter handling supported  

---

### Observability
- Exposes:
  - CDC lag
  - cascade depth
  - write throughput
  - index size
- Structured logs for permission changes  
- Metrics for alerting  

---

## Operational Guarantees

- Incremental permission indexing  
- Deterministic rebuilds  
- Continuous permission compilation  
- Audit-grade change tracking  
- Leopard index is disposable and reproducible  

---

## Deliverables

- Leopard indexer service  
- Mapping registry  
- Cascade engine  
- Leopard store adapter  
- Client DB writer  
- Metrics and dashboards  
- Operational runbooks  

---

## Definition of Done

The Indexer is complete when:

> Any permission-relevant database mutation deterministically produces correct, observable, and queryable permission changes across all derived indexes, with full replay and audit guarantees.

---

## References

- Google Zanzibar paper — Leopard Indexing System  
- Permify authorization model  
- Change Data Capture (Debezium)



* a **`/docs/design.md`** deep technical design
* a **milestone/task breakdown**
* or a **minimal folder structure + interfaces in Go**
