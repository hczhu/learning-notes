file-created-at:: 2026-11-09

- # Datadog Husky: architecture, multi-tenancy, and consistency
- **Sources**:
	- Richard Artoul and Cecilia Watt, [Introducing Husky, Datadog’s third-generation event store](https://www.datadoghq.com/blog/engineering/introducing-husky/), Datadog Engineering, May 17, 2022. User-supplied PDF, pp. 1–10.
	- Daniel Intskirveli and Cecilia Watt, *Husky: Exactly-once ingestion and multi-tenancy at scale*, Datadog Engineering, February 22, 2023. User-supplied PDF, pp. 1–15.
- **Core insight**: Put large, immutable data files in inexpensive blob storage; put authoritative visibility and coordination in a small transactional metadata layer. Independently scale ingestion, compaction, and querying—but preserve deterministic routing and atomic deduplication to make that separation correct.
- **Scope**: These notes describe the designs reported in the 2022–2023 posts, not necessarily Datadog’s current implementation. Failure implications and guarantee boundaries below are identified separately from reported mechanisms.

- ## Why Husky was needed
	- Logs and other events retain raw, high-cardinality information and support arbitrary query-time aggregation. A preaggregated metrics architecture cannot provide the same flexibility.
	- **First generation: clustered storage/search.** A malfunctioning node could disrupt the cluster; adding nodes during overload could trigger rebalancing and make the overload worse.
	- **Second generation: isolated storage shards.** A custom router repartitioned Kafka traffic; independent storage replicas and a custom query engine reduced the blast radius. Remaining problems included tenant bursts, expensive retention, and limitations on unindexed fields and richer analytics.
	- **Husky: an unbundled, schemaless, vectorized column store.** Optimized primarily for scans and aggregations, with search capabilities—not for a high rate of low-latency point lookups.

- ## Architecture: separate durable state from compute
	- **Write path**: Kafka → Shard Routers → shard-partitioned Kafka → Writers → columnar files in blob storage → metadata commit.
	- **Read path**: distributed query engine → Readers → metadata and blob files → partial aggregates → merged result. Queries never call Writers.
	- | Component | Responsibility | Important consequence |
	  | --- | --- | --- |
	  | Writers | Buffer events briefly, build files, upload them, and commit their presence | Can relocate or scale independently of queries |
	  | Blob storage | Hold durable event and ID files | Storage capacity is not tied to the number of query servers |
	  | Metadata Service / FoundationDB | Authoritative record of visible files; transactional coordination | Correctness depends on this state, not a worker’s local cache |
	  | Compactors | Merge small files and atomically replace their metadata with output-file metadata | Distributed, autoscaling, LSM-like maintenance |
	  | Readers | Read files and compute partial results | Query compute scales separately; Readers are described as almost stateless |
	- FoundationDB supplies **strictly serializable interactive transactions**. Blob storage supplies durable bytes; FoundationDB supplies the authoritative file set. They perform different jobs.
	- Fresh and historical data share one serving architecture. A small tenant’s year-long query can use many Readers; a large tenant’s rarely queried archive need not reserve comparable query compute.
	- Query pools can be isolated by workload or tenant—for example, monitors versus interactive exploration. This improves isolation, but does not imply that every underlying dependency is dedicated.

- ## Complex design 1: deterministic routing despite resharding
	- **Invariant**: An event with the same tenant, timestamp, and ID must return to the same logical shard, even when retried much later. Otherwise, shard-local deduplication can miss it.
	- Each tenant receives a list of shards sized for its volume. Within that list, an event-ID hash selects a shard. Simply changing the list breaks determinism: `hash(event_id) % shard_count` can change when the count changes.
	- **Solution: immutable, time-bounded Shard Placements.** Each placement records a tenant, a nonoverlapping time interval, and a shard list. Routing looks up the interval using the **event timestamp**, not its arrival time.
		- Example: an event timestamped during a two-shard placement still uses that placement when retried after the tenant has grown to four shards. New placement intervals use the new allocation.
		- Routers call `GetPlacement(tenant_id, timestamp)`. FoundationDB atomically reads or creates the placement so competing routers agree; immutable placements can then be cached.
		- Allocation is a logical coordination authority, but the service is split into isolated deployments with dedicated FoundationDB clusters—not one global singleton.
	- **Why locality matters twice**: it bounds the IDs each deduplicator must inspect and reduces file proliferation. Event tables are tenant-specific, and files do not mix tables; spreading more tenants across each Writer produces more small files, uploads, and compaction work.
	- **Terminology trap**: Husky removes much of the old storage-shard/replica machinery, but still uses logical ingestion shards. Those shards can map to multiple Kafka partitions, topics, or clusters.

- ## Complex design 2: four different scaling/control loops
	- | Mechanism | Decision | Constraint / tradeoff |
	  | --- | --- | --- |
	  | Autosharder | How many shards a tenant needs, using byte throughput | Downshards more slowly than it upshards; reuses the existing count if throughput metrics are unavailable |
	  | Sharding Allocator | Which shards appear in a placement | Consistent-hashes the tenant plus a salt to a starting shard, then selects neighboring shards, skipping exclusions; minimizes churn |
	  | Sharding Balancer | Which tenant allocations should move | Changes persistent tenant salts for future placements; reduces uneven shard load |
	  | Writer autoscaling / reassignment | How many workers serve the shards | Gives each shard more processing capacity without changing historical event routing |
	- **Uniform hashing is not uniform workload.** Tenants have different byte volumes, and neighboring shard lists overlap. The balancer measures variance across shard loads, simulates moves involving tenants on the hottest shard, and chooses moves that reduce variance, subject to a move budget. This is a heuristic, not a proven global optimum.
	- **Burst handling is deliberately separate from routing changes.** An immutable placement cannot instantly spread its events over extra shards. Instead, add Writers and assign fewer shards to each: moving from ten to five shards per Writer approximately doubles available throughput per shard in the article’s example.
	- Hot shards can be excluded from **future** placements while their backlog drains. Existing placements and retries still honor the old mapping.

- ## Consistency: what makes exactly-once ingestion work?
	- The goal is **one committed effect**, not one delivery or one execution. Kafka replay, retries, restarts, and overlapping shard ownership still happen. Duplicate events matter even when rare: they can change monitor outcomes and cost attribution.
	- Three mechanisms must work together:
		- **Stable placement** sends duplicates to the same logical shard.
		- **Atomic publication** keeps recorded IDs and visible event data consistent.
		- **Conflict detection** prevents concurrent Writers from committing incompatible decisions based on stale ID caches.
	- ### Durable deduplication without stateful Writers
		- Event data lives in ordinary tenant tables. Event IDs live in separate **per-shard ID tables**, with files partitioned by time.
		- Writers lazily load time intervals of IDs into memory, perform fast lookups, and evict cold intervals using an LRU cache. These are bulk reads, not a remote lookup for every event.
		- ID tables reuse Husky’s blob storage, compaction, and configurable TTL. IDs are **not all stored as individual FoundationDB records**; FoundationDB coordinates the metadata that publishes the files.
		- Event data and its IDs are committed to the Metadata Service in the **same FoundationDB transaction**. There must not be a durable “already seen” ID without its corresponding committed data.
		- A separate deduplication database would create a dual-write problem: recording the ID and then failing to publish the event could make a retry incorrectly discard data. RAM alone loses history on restart; local RocksDB would complicate worker relocation.
	- ### Concurrent Writers: atomicity alone is insufficient
		- Two Writers can briefly own the same shard and both see an ID as absent in their caches. Merely committing each Writer’s IDs and events together does not resolve that race.
		- Husky versions ID-table updates and uses **optimistic concurrency control** to reject out-of-order competing updates.
		- On conflict, the losing Writer discards its ID cache and resets consumer progress to the latest committed fragment, then reconciles/reprocesses against committed state.
		- “Stateless Writer” therefore means **no indispensable local durable state**, not no buffers, caches, or recovery work. Coordination has moved into FoundationDB, not disappeared.
	- ### Compaction: atomic visibility, not an atomic blob-store rewrite
		- Compactors upload merged files, then atomically remove input-file metadata and add output-file metadata. This prevents a partially published replacement file set.
		- The transaction concerns **metadata**. The posts do not specify the physical deletion/garbage-collection protocol or how every long-running query pins its file snapshot; do not infer those guarantees from atomic replacement alone.
	- ### Failure scenarios and guarantee boundaries
		- The first two rows are implications of the publication design; exact acknowledgment and cleanup protocols are not documented in these posts.
		- | Scenario | Correctness implication |
		  | --- | --- |
		  | File upload succeeds; metadata commit fails | The upload alone does not make the event visible; retry is possible, but unreferenced-object cleanup is unspecified |
		  | Commit succeeds; Writer dies before recognizing success | Committed ID history allows replay to suppress duplicates; the precise Kafka acknowledgment/offset protocol is not given |
		  | Two Writers act on stale ID caches | Version conflict forces reconciliation instead of accepting inconsistent updates |
		  | Old event arrives after a reshard | Event-time placement routes it to the original shard, provided routing history remains available |
		  | ID history expires, or a producer changes the event’s identity | Do not assume the same deduplication guarantee still applies |
	- **Strong metadata consistency is not immediate freshness.** Buffering, Kafka lag, file upload, and commit delay when an event becomes queryable. Strict serializability in FoundationDB does not establish a globally consistent snapshot across an entire distributed Husky query.
	- **Availability boundary**: cached placements reduce coordination work, but the posts do not establish safe creation of new placements or successful metadata commits during FoundationDB unavailability. The documented metrics-outage fallback is not a metadata-outage fallback.

- ## Technical tradeoffs and reported results
	- **Batching versus freshness and file count**: frequent small flushes expose data sooner but create more objects and read/compaction overhead. Locality and compaction help manage this tension.
	- **Remote storage versus latency**: the 2022 post reports a median-latency cost of a few hundred milliseconds versus SSD-backed storage, but improvements of seconds, tens of seconds, and hundreds of seconds at p95, p99, and maximum latency respectively. These are operational observations, not universal benchmark guarantees.
	- **Recovery versus cache size**: restarts require loading ID intervals. Datadog reports ID-loading throughput around two orders of magnitude above event-processing throughput; roughly time-ordered arrival also keeps normal page-in demand low.
	- **Retention versus replay safety**: configurable ID TTL bounds cost, but a deployment must align retained deduplication and placement history with the retries it promises to handle. The posts do not specify a universal deduplication window or placement-history TTL.
	- **Migration discipline**: reliability first, performance second, efficiency third. Datadog dual-wrote to old and new systems and shadowed 100% of queries for months; the first complete product migration took more than 1.5 years. The post reports no increase in incident rate and successful autoscaling through bursts.

- ## Reusable engineering lessons
	- Separate heavy immutable bytes from a small, transactional record of what is committed.
	- Treat routing as part of the correctness protocol when deduplication is local—not merely as load balancing.
	- Co-commit data visibility and deduplication evidence; also detect concurrent stale decisions.
	- Use separate controls for tenant allocation, shard placement, load skew, and worker capacity; one hashing scheme cannot solve all four.
	- Define “exactly once” through identity, retained history, publication boundaries, and recovery behavior—not a promise that failures or retries disappear.
