# Scalability guide

Applies when a change decides *where code lives* or *what it costs at scale* —
new features, new endpoints, new tables, new background work, new
infrastructure. For how the code inside those boundaries should read, see
`coding-practices.md`; where the two meet, the stricter rule wins.

## The priority order

Optimize in this order, and never trade an earlier item for a later one:

**Correctness → Security → Maintainability → Performance → Scalability →
Complexity**

Theoretical scalability is not worth real correctness or maintainability. Every
rule below resolves against this order.

## Follow the existing architecture first

Before adding a pattern, find the one already in use.

- **Inspect before you write.** Project structure, similar implementations,
  existing utilities, naming conventions, error-handling patterns.
- **Reuse beats reinvention.** A second HTTP wrapper, a second result type, a
  second cache is a maintenance cost paid forever.
- **Never introduce a competing architecture** because it is faster to
  implement. If the app is `API → Service → Repository → Database`, a new
  feature does not get to be `API → Database`.
- Consistency is the point. Divergence is technical debt with a delay fuse.

## Layer responsibilities

Keep the layers separated: UI, API, business logic/services, database, cache,
background jobs, external integrations.

- **A component never reaches the database directly.** The path is
  `UI → API endpoint → service → database`, and side effects that can wait go
  `service → queue → worker → provider`.
- **A single file that queries, decides, renders and sends** is the rule
  breaking, not a shortcut.
- The layer a change belongs in is decided before it is written, not
  discovered afterwards.

## Modules own their boundaries

Group by feature, not by technical kind — `features/billing/`,
`features/projects/`, `features/analytics/`, rather than a shared `services/`
and `utils/` where unrelated business logic collects.

- **A feature should be understandable, testable and changeable on its own.**
- **Don't create cross-feature dependencies** that force a future change to
  touch unrelated modules. If billing imports from analytics to read one
  constant, the constant is in the wrong place.

## Query the database the way the data is shaped

Before adding a query: know what data is needed, check the existing schema and
relationships, and know the access pattern it creates.

- **Filter, sort and paginate in the database**, not in application memory.
  Fetching 10,000 rows to return 20 is a bug that only shows up under real
  data.
- **Index what is frequently filtered, joined or sorted** — when the access
  pattern justifies it, not speculatively. Every index costs write throughput.
- **Never N+1.** One query per row of a result set is the most common
  performance failure in this kind of codebase; join, batch, or index the
  lookup instead.
- **Select the columns you need.** `SELECT *` over a wide table ships bytes
  nobody reads.
- **Measure against realistic volume.** A query that is instant over 50 rows
  tells you nothing about 500,000.
- **No sharding or partitioning** until the current scale actually demands it.

## API contracts must survive their consumers

- **Predictable and consistent shapes** for requests, responses and errors.
  One error envelope across the API, not one per endpoint.
- **Validate and narrow every input at the boundary** — see the security rules
  in `coding-practices.md`.
- **Collection endpoints take pagination, filtering and sorting**:
  `GET /api/projects?page=1&limit=20&status=active`, never
  `GET /api/projects?everything=true`.
- **Authentication, authorization, and rate limiting where the endpoint
  warrants it** — public endpoints especially.
- **Check the consumers before changing a contract.** If a breaking change is
  unavoidable, version or migrate deliberately; don't break a client and fix it
  afterwards.

## Move expensive work out of the request

A user-facing request should not wait for work that does not have to finish
synchronously.

- **Background-job candidates:** email, media processing, report generation,
  bulk import, external synchronization, analytics rollups, LLM calls,
  scheduled tasks.
- **The pattern is accept-then-work:** `POST /reports` creates a job and
  returns its id; a worker does the work and updates the job's status. The
  request stays fast and the client polls or subscribes.
- **A job must be safe to retry.** Workers fail mid-run; design for it rather
  than assuming exactly-once delivery.

## Assume more than one server

Treat application servers as stateless unless there is a reason not to.

- **Shared state belongs in shared infrastructure** — database, cache, object
  storage — never only in a single process's memory. In-memory session state,
  in-memory rate-limit counters and in-memory job locks all break the moment a
  second instance answers the next request.
- **In-process caching is allowed where staleness is harmless** and the cost of
  a miss is low. Say so, and make the correctness independent of the cache.
- **Don't add distributed infrastructure** for a problem a single database can
  still solve.

## Cache to remove measured repeated work

Caching is in `coding-practices.md` under Efficiency; the system-level decision
belongs here.

- **Cache what is expensive, repeated and stable** — hot records, costly
  computations, external API responses, configuration.
- **Every cache needs an answer for four questions**: how it expires, how it is
  invalidated, how stale the data may safely be, and what happens when the
  cache is unavailable. A cache whose outage takes the app down is now a single
  point of failure.
- **Do not add a cache because scalable systems have one.** Add it because you
  measured the repeated work.

## Observe what you operate

Anything important enough to run in production is important enough to see.

- **Track the signals that identify a failure**: API latency, error rate, slow
  queries, background-job failures, queue depth, external-API failures,
  resource usage, and the business events that matter.
- **Diagnose from evidence.** Latency goes 200ms → 2.5s: measure the API,
  measure the database, find the slow query, fix the index, measure again.
  Guessing costs more than instrumenting.
- Route operational reporting through the project's ops module rather than bare
  `console.*` — see the Logging rules in `coding-practices.md`.

## Know the blast radius before you change anything

Before modifying existing code, find out what depends on it.

- **Who calls it, what data it writes, which features it backs, what tests
  cover it, and how it behaves under larger data or traffic.**
- **Don't optimize one path into breaking another.** A refactor that improves
  one module and silently changes a contract used by three others is a net
  loss.
- Changes to shared code are reviewed against their consumers, not just
  against themselves.

## Match infrastructure to the current stage

Choose for the traffic you have and the growth you can evidence.

- A small application generally needs an app framework, a managed database,
  object storage, a simple queue and a CDN. It does not automatically need
  microservices, Kubernetes, Kafka, multiple databases or a service mesh.
- **Weigh the real costs**: operational complexity, team expertise, money, and
  how hard it is to migrate away from later.
- The goal is infrastructure that can evolve — not infrastructure that looks
  impressive on a diagram.

## Scale on evidence, not anticipation

**Measure → identify the bottleneck → understand the root cause → apply the
smallest effective fix → measure again.**

- **Solve the bottleneck you have.** A slow dashboard is usually one slow
  query, not an argument for redesigning the architecture.
- **Before adding a technology or a pattern, answer:** what problem does it
  solve, what evidence shows the current system can't, what is the simpler
  option, what does it cost to operate, and can the team maintain it.
- An occasional expensive operation becomes a background worker. It does not
  become a microservice, an event bus and a cluster.
