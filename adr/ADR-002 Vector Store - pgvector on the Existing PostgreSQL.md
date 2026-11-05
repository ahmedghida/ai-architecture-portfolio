# ADR-002: Vector Store - pgvector on the Existing PostgreSQL

Status: Accepted · System: Marketing Automation with AI & Agents (property seeker side, WhatsApp) · Principles: P3, P4

## Context

The Matching Engine runs vector search over active property listings to find matches for each seeker. Every search must also apply hard filters such as budget, area and number of bedrooms. Assumed scale: about 100-500K active listings, which is roughly 1.5-3 GB of vectors (768-1536 dimensions, float32). Matching runs as a daily batch (see ADR-003), so there is no real-time freshness requirement. PostgreSQL is already the main database for listings. The team is assumed to be small.

## Options considered

| Option | Pros | Cons |
| --- | --- | --- |
| A. pgvector inside the existing PostgreSQL | No new infrastructure; SQL filters and vector search in one query; vectors stay consistent with listing updates; one governed data store | Less scalable than a dedicated engine; the batch job competes for database resources |
| B. Managed dedicated vector DB (e.g., Pinecone) | Scales easily; low operational effort | New vendor and cost; data copied to another system; filters and sync need extra work |
| C. Self-hosted dedicated engine (e.g., Qdrant) | Strong filtering and performance; data stays in-house | A new system to run, secure and monitor with a small team |

## Decision

Use pgvector in the existing PostgreSQL. Run the matching queries on a read replica or off-peak, so the daily batch does not load the primary database.

## Consequences

- Positive: no new system to operate or secure; hard filters and similarity in a single SQL query; listing data and embeddings stay in one governed store (supports P3).
- Negative: if the data grows a lot, performance may degrade; mitigated by the read replica and by the revisit triggers below.
- Negative: batch matching shares database resources; mitigated by running it off-peak.

## Assumptions and revisit triggers

Assumptions: 100-500K listings, daily batch matching, small team. Revisit if active listings pass several million, if search latency becomes a product requirement, or if the batch job starts affecting production traffic. Moving to a dedicated engine would be a new ADR and requires ARB review.
