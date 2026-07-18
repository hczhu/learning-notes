- **Source**: Reynold Xin (Databricks), "From monolith to Lakebase to LTAP: rethinking the database from storage up," Databricks Engineering Blog, 30 Jun 2026. Plus a Hacker News discussion summary (first-hand builder experiences + highly-voted insights).
- **One-liner**: Externalize a monolithic database's two on-disk structures (WAL + data files) into independent cloud services, then materialize the durable copy in an open columnar format (Parquet/Iceberg/Delta) so **transactions and analytics run on a single copy of the data** — no CDC, no mirroring, no ETL. Databricks calls this **LTAP** (Lake Transactional/Analytical Processing).

- ## The Monolith and Its Problems
	- Most databases (MySQL, Postgres, Oracle) are **monoliths**: one machine runs the engine + storage. Two on-disk structures matter most:
		- **WAL (write-ahead log)** — sequential append; makes **writes** fast and safe. A commit is durable the moment the log entry is flushed.
		- **Data files** — make **reads** fast by holding current state directly (no replaying history). Updated asynchronously after the WAL.
	- Every core problem traces to one root cause: **WAL + data files live inside a single machine**.
		- **Data loss from misconfiguration**: a commit is only as durable as the disk flush behind it; "acknowledged before flushed" settings are subtle, silent, and the OS may even lie about flushing.
		- **Data loss from node loss**: disk dies → data dies. RAID/NAS helps durability but doesn't solve it (mount dies → access dies).
		- **Scaling reads requires a physical clone**: a read replica is a full physical copy streaming + replaying the WAL; slow to provision for large DBs.
		- **HA requires a physical clone**: ≥1 standby full copy kept in sync; pay ≥2× infra, slow standby bring-up, need synchronous replication to avoid data loss (often 3+ nodes recommended).
		- **Analytics contends with transactions**: one big reporting query or GDPR cleanup degrades latency-sensitive OLTP on shared hardware.

- ## Lakebase Architecture (make Postgres compute stateless)
	- Redesign OLTP on modern-cloud primitives: **cheap durable object storage + elastic compute** (the path the Neon team took → Lakebase). Externalize WAL and data files into two purpose-built services so the Postgres compute layer becomes **stateless** (start/stop/replicate freely because it no longer owns the data).
	- **Scaling writes → SafeKeeper**: the WAL moves to a distributed service. Durability comes from **Paxos-based quorum replication** across SafeKeeper nodes, not a local disk flush. No single disk whose failure loses data; no misconfigured flush. The extra network hop adds no net overhead — combined with PageServer it can give **5× higher write throughput and 2× lower read latency**.
	- **Scaling reads → PageServer**: data files move to a distributed service acting as a **write-through cache** over object storage. WAL streams SafeKeeper → PageServer, which asynchronously materializes pages into low-cost object storage (the "lake"). Cache miss? PageServer replays logs to reconstruct latest state.
	- **Read path caching** (why latency ≈ monolith): buffer pool (local memory) → local file cache (local disk) → PageServer → object stores. Only a cache miss reaches PageServer; a compute node sized like a monolith keeps the same cache hit rate.
	- **What it unlocks**: still real Postgres (wire protocol/SQL/drivers/extensions); **unlimited storage** (object storage, no capacity ceiling); **serverless elastic compute** (scale to zero when idle); durable quorum writes; simpler HA (durable state independent of any compute node); **instant branching/cloning/PITR** as a metadata operation, not a physical copy ("the database finally moves as fast as your code").

- ## LTAP: One Copy for Transactions AND Analytics
	- Even in Lakebase, object-storage data was still Postgres's **row-oriented page format** — great for transactions, poor for analytics. Any analytical engine had to pay a per-read conversion cost or keep a **separate synced copy** (brittle pipeline; two copies → governance nightmare with diverged permissions).
	- **LTAP's key idea**: unify at the **storage layer, not the engine layer**. Keep the best tool per job — Postgres (full ACID) for transactions, Lakehouse engines for analytics — but underneath there is **one durable copy in open columnar formats (Delta/Iceberg as Parquet)** that both sides read (with caches).
	- ### Materializing in columnar form (Postgres-internals heavy)
		- As PageServer materializes pages, its **spare CPU transcodes row → Parquet columnar** as data lands in the lake — no burden on the Postgres compute serving transactions. Unlike CDC (which ships logical change events into a foreign schema, losing physical/transactional semantics), LTAP **preserves the exact Postgres representation down to the bits**.
		- **Type system**: most Postgres types map directly to native Parquet types. Values with no lossless columnar counterpart (NaN, ±Infinity, out-of-range NUMERICs, exotic/extension types) are **not dropped or coerced** — carried in a structured overflow field holding canonical Postgres text, queryable and sufficient to reconstruct original bytes.
		- **Multi-versioning** (separate durability from visibility): every columnar row carries its **physical heap address (block + offset)** so heap pages stay reconstructable. The Postgres heap page becomes a **cache** for point reads; the durable source of truth is the columnar files. Indexes aren't transcoded — served/rebuilt from the hot cache tier. Intermediate MVCC row versions are retained (preserving snapshot isolation + PITR) but **invisible to Iceberg/Delta readers** and eventually garbage-collected. Net: analytical engines see clean snapshot-consistent tables; Postgres underneath sees full time-travelable history.
		- **Side effect**: columnar compresses >10× better than row data, cutting network volume between cache and object store to near-negligible. (During rollout they dual-write row + columnar for verification.)
	- ### Freshness (the question that sinks "just point analytics at the lake")
		- An analytical query first asks Postgres for the **current LSN** (log sequence number) — a cheap metadata lookup.
		- With that LSN, the engine reads the **overwhelming majority of data directly from object storage** (everything materialized up to that point), then fetches only the **small set of very recent, not-yet-materialized changes** from PageServer and merges on top.
		- Result: a consistent, fully up-to-date read as of that LSN. **Postgres serves none of the analytical read traffic except returning one number (LSN)** — transactional workload doesn't slow down.
		- Optimization: very small tables (a handful of rows) aren't converted to columnar/Iceberg — the bookkeeping would cost more than it saves; still present and queryable.
	- **LTAP vs CDC/"mirroring"/"zero-ETL"**: CDC replication costs something, so you must **explicitly select tables** and accept a **delay**. LTAP has **nothing to opt into** — every table is, by construction, already in the lake and queryable. No replication, one governed copy, engines scale independently, and the two views **can never drift** (analytics reads the same data the app just wrote).

- ## Why LTAP over HTAP (and the deeper thesis)
	- LTAP is a deliberate play on **HTAP** (hybrid transactional/analytical processing — the "holy grail" of one engine doing both). No single HTAP system got widely adopted because such systems suffer:
		- **Immature engine**: building one engine great at both compounds the work; they lag on SQL breadth (e.g. foreign keys), optimizer maturity.
		- **No ecosystem**: Postgres and Spark each sit at the center of vast ecosystems; a brand-new engine starts outside all of it.
		- **No performance isolation**: running both on shared hardware reproduces the monolith's contention.
	- All three trace to unifying **at the engine**. Lakebase/LTAP instead unify **at the storage layer** while using **different compute engines** per workload — keeping full feature sets, ecosystems, and performance isolation.

- ## Hacker News Discussion — First-Hand Experiences & Debated Insights
	- ### Builder experiences (production realities)
		- **ianberdin (Playcode Cloud)** built a Rust page-server/chunked-FS/NVMe-cache/object-sync stack and warned it's much harder than on paper: object storage has rate limits, dropped packets, hanging requests; **NVMe write speeds are often the bottleneck** (~400 MB/s write despite 5 GB/s read); high uplink (5–10 Gbps) and CPU for chunk compression become bottlenecks.
		- **conradludgate (Lakebase/Neon)**: the caching layer (PageServers on NVMe) is **inherent** — reading directly from S3 is too slow. Storing S3 data as Parquet just optimizes for analytics **without adding copies**.
		- **sunzhousz (Moonlink)**: switched from a ZeroETL mirroring approach to LTAP because **CDC is too expensive to maintain for rarely-queried tables**, and simple mirroring inevitably morphs into complex ETL. Noted **up to 100× write amplification** when merging CDC into column stores for fresh reads.
		- **scritty-dev**: migrated to **RustFS** as a drop-in S3 replacement — plug-and-play vs SeaweedFS/Garage, relevant after MinIO's licensing change.
	- ### Debated themes
		- **The data lifecycle ("special sauce," nikita)**: recent/working-set data stays in **Postgres page format** (OLTP hot path); historical data below a certain **LSN** is pushed to **S3 as Parquet** asynchronously (off the transaction hot path); query engines read **both representations simultaneously** for real-time analytics.
		- **CDC vs zero-copy**: anti-CDC camp says CDC is error-prone, burns CPU/memory, occupies replication slots, needs heavy ETL. **Skeptics (saisrirampur)**: can async Parquet conversion keep up with heavy OLTP (**50K+ TPS**)? A well-built CDC pipeline stays native to each datastore and avoids the compromises of a "unified" engine.
		- **History / SCD Type 2 & time travel**: Delta Lake and Apache Iceberg natively support **time travel** (query a past snapshot). But **hasyimibhar's** data-modeling point: relying on SCD Type 2 downstream is often a workaround for poor upstream design — if an entity's history matters (e.g. email changes), model it explicitly (an `email_history` table) rather than inferring from CDC timestamps that can be delayed/misaligned with logical events.
		- **Storage costs & S3 alternatives**: S3↔EC2 same-region traffic is free, but vendor lock-in is debated. Consensus: large enterprises are more **risk-averse than price-sensitive** — they stick with AWS as the "safe" choice, though Azure Blob, Cloudflare R2, or self-hosted (SeaweedFS, RustFS) are viable for cost-cutting/on-prem.

- ## Takeaways
	- The recurring architectural move: **separate durability from visibility, and unify at storage rather than at the engine.** Row format becomes a hot cache; open columnar in object storage is the durable source of truth.
	- The hard parts are operational, not conceptual: NVMe write bandwidth, object-storage tail latency, keeping async columnar materialization ahead of high-TPS writes, and per-read merge of unmaterialized recent changes.
	- Related: [[2026-05-inference-engineering]], [[2026-05-what-si-cloud-again-gcp]]
