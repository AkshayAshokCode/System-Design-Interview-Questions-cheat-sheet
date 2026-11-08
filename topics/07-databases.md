# Databases: SQL, NoSQL & Indexing

[⬅ Back to index](../README.md)

Covers: SQL vs NoSQL, ACID vs BASE, NoSQL families, how to choose, indexes, B-trees vs LSM trees, normalisation vs denormalisation, query optimisation, connection pooling, time-series and graph databases, NewSQL.

---

### 1. SQL vs NoSQL – how do you choose?

<details>
<summary>Answer</summary>

| Relational (SQL) | NoSQL |
|---|---|
| Fixed schema, tables, joins | Flexible schema (document, key-value, wide-column, graph) |
| **ACID transactions** across rows and tables | Often single-item/partition atomicity; tunable consistency |
| Rich ad-hoc queries (SQL, joins, aggregates) | Queries designed around known access patterns |
| Scales vertically + read replicas; sharding is manual or via NewSQL | Built to scale horizontally (partitioning is native) |
| Postgres, MySQL | DynamoDB, Cassandra, MongoDB, Redis, Neo4j |

Choose **SQL** when you need transactions, relationships, flexible querying and the scale fits (which covers most systems; Postgres can go a long way). Choose **NoSQL** for massive scale with simple access patterns, very high write throughput, flexible or nested data, or specific models (graph, time-series). Many large systems use both (polyglot persistence).

</details>

### 2. What does ACID mean?

<details>
<summary>Answer</summary>

- **Atomicity** – a transaction happens entirely or not at all (rollback on failure).
- **Consistency** – a transaction moves the DB from one valid state to another (constraints, foreign keys hold).
- **Isolation** – concurrent transactions don't see each other's partial work (to the degree of the isolation level).
- **Durability** – once committed, data survives crashes (write-ahead log flushed to disk, replicated).

Example: a bank transfer debits A and credits B in one transaction; you never see money disappear or appear.

</details>

### 3. What is BASE?

<details>
<summary>Answer</summary>

**Basically Available, Soft state, Eventually consistent** – the philosophy of many distributed NoSQL stores. The system stays available during partitions, replicas may temporarily disagree, and they converge if writes stop. It trades immediate consistency for availability and scale. Modern systems blur the line: DynamoDB and Cassandra offer per-request strong reads, and NewSQL offers ACID at scale.

</details>

### 4. What are the NoSQL families and when do you use each?

<details>
<summary>Answer</summary>

- **Key-value** (Redis, DynamoDB, Riak) – lookups by key; sessions, carts, caches, counters.
- **Document** (MongoDB, Couchbase, Firestore) – JSON-like nested documents; catalogs, user profiles, CMS content where each record is read whole.
- **Wide-column** (Cassandra, HBase, Bigtable, ScyllaDB) – rows with many columns in partitions sorted by a clustering key; huge write volumes, time-series, messages, feeds ("all messages in channel X ordered by time").
- **Graph** (Neo4j, Neptune) – nodes and edges; social graphs, recommendations, fraud rings, multi-hop traversals.
- **Time-series** (InfluxDB, TimescaleDB, Prometheus) – metrics, IoT; compression and time-window queries.
- **Search** (Elasticsearch, OpenSearch) – inverted indexes for full-text search.
- **Vector** (pgvector, Pinecone, Milvus) – similarity search over embeddings.

</details>

### 5. How does a database index work?

<details>
<summary>Answer</summary>

An index is an extra data structure that lets the DB find rows without scanning the whole table, like a book's index. Most relational indexes are **B+ trees**: sorted keys in a shallow, wide tree (3–4 levels for billions of rows), so lookups and range scans take O(log n) page reads.

Trade-offs:
- Faster reads, filters, sorts and joins on indexed columns.
- **Slower writes** (every insert/update maintains each index) and extra storage.

Types: primary/clustered (rows stored in key order, as in InnoDB), secondary, **composite** (multi-column; order matters, leftmost prefix rule), unique, covering (includes all queried columns), partial, hash, GIN (full-text, JSON), geospatial (R-tree/GiST).

</details>

### 6. B-tree vs LSM tree storage engines?

<details>
<summary>Answer</summary>

| B-tree (Postgres, MySQL InnoDB) | LSM tree (Cassandra, RocksDB, LevelDB, ScyllaDB) |
|---|---|
| Updates pages in place | Writes go to a WAL + in-memory **memtable**, flushed to immutable sorted files (**SSTables**), merged by **compaction** |
| Reads: one tree lookup → fast, predictable | Reads may check memtable + several SSTables (Bloom filters help) |
| Writes: random I/O | Writes: sequential → **very high write throughput** |
| Good for read-heavy, transactional workloads | Good for write-heavy workloads (logs, events, time-series) |

LSM trade-offs: compaction uses background I/O, and there's read and space amplification. B-trees suffer write amplification and fragmentation.

</details>

### 7. How do composite indexes work? What is the leftmost-prefix rule?

<details>
<summary>Answer</summary>

An index on `(user_id, created_at)` is sorted by `user_id`, then `created_at` within each user. It can serve:
- `WHERE user_id = ?`
- `WHERE user_id = ? AND created_at > ?`
- `WHERE user_id = ? ORDER BY created_at DESC LIMIT 20` (no sort needed)

But **not** `WHERE created_at > ?` alone, because it's not the leftmost column. Put **equality columns first, then range/sort columns**. A **covering index** that includes all selected columns avoids reading the table at all.

</details>

### 8. Normalisation vs denormalisation?

<details>
<summary>Answer</summary>

- **Normalisation** – store each fact once (1NF/2NF/3NF), relate tables with foreign keys. No update anomalies, less storage, but reads need joins.
- **Denormalisation** – duplicate data to make reads cheap (store the author's name in each post, precomputed counts, materialised feeds). Faster reads, but writes must update every copy (or accept staleness).

At scale, especially with NoSQL and sharding (joins across shards are expensive), you **design the tables around queries** and denormalise deliberately, keeping one source of truth and updating copies asynchronously.

</details>

### 9. How do you optimise a slow query?

<details>
<summary>Answer</summary>

1. Look at the **query plan** (`EXPLAIN ANALYZE`): full table scans, bad join order, sorts on disk, wrong row estimates.
2. Add or fix **indexes** that match the WHERE / JOIN / ORDER BY columns; use covering indexes.
3. Fetch less: select only needed columns, paginate with keysets, avoid `SELECT *`.
4. Fix N+1 queries (batch or join).
5. Update statistics; rewrite subqueries; avoid functions on indexed columns (`WHERE LOWER(email)=…` needs an expression index).
6. Cache results, or precompute with materialised views.
7. Check locks, connection pool saturation and hardware (I/O, memory for the buffer pool).

</details>

### 10. What is the N+1 query problem?

<details>
<summary>Answer</summary>

Fetching a list (1 query) and then issuing one query per item for related data (N queries): 1 query for 50 posts + 50 queries for authors. Each round trip adds latency and DB load. Fix with a **join**, an `IN (…)` batch query, ORM eager loading, or a **DataLoader** pattern (batching and caching per request, common in GraphQL).

</details>

### 11. Why do you need connection pooling for databases?

<details>
<summary>Answer</summary>

Opening a DB connection is expensive (TCP + TLS + auth, and in Postgres a whole process per connection). Databases support a limited number of connections (hundreds to low thousands). With many app instances × threads, you can exhaust them. A **pool** keeps a fixed set of open connections that requests borrow and return. At larger scale, use an external pooler (**PgBouncer**, RDS Proxy) so thousands of app connections share a few hundred DB connections.

</details>

### 12. What are stored procedures, triggers and materialised views, and should you use them?

<details>
<summary>Answer</summary>

- **Stored procedures** – logic inside the DB. Fewer round trips, but hard to version, test and scale; business logic usually belongs in services.
- **Triggers** – run on insert/update. Hidden side effects; use sparingly (audit logs are a reasonable case).
- **Materialised views** – precomputed query results stored as a table, refreshed periodically or incrementally. Great for expensive aggregates and reporting.

</details>

### 13. What is NewSQL / distributed SQL?

<details>
<summary>Answer</summary>

Databases that keep **SQL and ACID transactions** while **scaling horizontally** by automatically sharding and replicating data with consensus (Raft/Paxos):
- **Google Spanner** – global, externally consistent transactions using TrueTime (GPS + atomic clocks).
- **CockroachDB, YugabyteDB, TiDB** – Spanner-inspired, open-source.
- **Amazon Aurora** – scales storage and reads; writes on one primary (Aurora Limitless/DSQL add more).

Trade-off: higher write latency (consensus across replicas or regions) and cost. Use when you need relational semantics at a scale where manual sharding would be painful.

</details>

### 14. When would you use a graph database?

<details>
<summary>Answer</summary>

When queries traverse relationships of variable depth: "friends of friends who like X", fraud rings (shared devices/cards), knowledge graphs, access-control graphs, recommendations. In SQL these become recursive joins that get slow with depth; graph DBs store adjacency directly (index-free adjacency). For simple one-hop relations (followers list), a regular DB or wide-column store is fine; Meta's TAO, for example, is a graph *model* on top of MySQL + caching.

</details>

### 15. What is a write-ahead log (WAL)?

<details>
<summary>Answer</summary>

Before changing data pages, the DB appends the change to a sequential **log on disk**. On a crash, it replays the log to recover committed transactions (durability) and undo incomplete ones. The WAL also powers **replication** (ship the log to replicas), **point-in-time recovery** and **change data capture** (Debezium reads the Postgres WAL or MySQL binlog). Many systems share this idea: Kafka is essentially a distributed log.

</details>

### 16. How do you store money, time and IDs correctly?

<details>
<summary>Answer</summary>

- **Money** – never floating point. Use integers in minor units (cents) or `DECIMAL`, and store the currency code.
- **Time** – store timestamps in **UTC** (`timestamptz`), convert in the UI; store the user's time zone separately when it matters (calendar events, recurring schedules).
- **IDs** – avoid exposing sequential IDs publicly (enumeration). Use UUIDv7/ULID or Snowflake-style IDs: unique across shards and roughly time-sortable (good for index locality).

</details>

### 17. How do you do a schema migration without downtime?

<details>
<summary>Answer</summary>

Use **expand and contract**:
1. **Expand** – add the new column/table (nullable, no locking default). Deploy code that writes to both old and new.
2. **Backfill** – copy existing data in batches (throttled).
3. Switch reads to the new column; verify.
4. **Contract** – stop writing the old column, then drop it later.

Use online schema-change tools for big tables (gh-ost, pt-online-schema-change, Postgres `CREATE INDEX CONCURRENTLY`). Every step must be backward compatible so you can roll back.

</details>

---

[⬅ Previous: Caching](06-caching.md) · [⬅ Back to index](../README.md) · [Next: Replication, Partitioning & Sharding ➡](08-replication-sharding.md)
