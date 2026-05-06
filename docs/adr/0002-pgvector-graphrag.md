# ADR 0002 — pgvector + Apache AGE for GraphRAG

- **Status:** Accepted
- **Date:** Phase 2

## Context

Curator needs hybrid retrieval: vector similarity, lexical match, and graph expansion across an entity-relationship knowledge graph. Privacy requires per-tribe isolation. We want one operational substrate, not three.

## Options

1. **Postgres + pgvector + Apache AGE + pg_trgm** in one DB.
2. Pinecone / Weaviate / Qdrant for vectors + Neo4j for graph + Postgres for the rest.
3. Elastic for hybrid + an RDBMS for graph.

## Decision

Stay in **Postgres** with **pgvector** (vectors), **AGE** (graph), and **pg_trgm** (lexical). One database, one RLS surface, one backup story.

## Consequences

- ✅ Single source of truth; transactions can span all three.
- ✅ Tribe-scoped queries enforceable by RLS (vectors) and per-tribe graph labels (`g_tribe_<uuid>`).
- ✅ Smaller ops surface; lower cost.
- ⚠️ AGE is younger than Neo4j; we accept fewer features (no Cypher write batching at scale) and pin a stable AGE version.
- ⚠️ Need careful index strategy for vector + lexical hybrid; documented in [08](../08_RAG_AND_KNOWLEDGE_GRAPH.md).

## Revisit when

- Graph cardinality per tribe exceeds AGE's tested envelope (millions of edges).
- We need cross-tribe (federated) graph queries — that would be a different design.
