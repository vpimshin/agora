# 08 — RAG & Knowledge Graph

## 1. Why GraphRAG, not flat RAG?

Flat vector RAG returns top-k similar chunks. It works, but for *learning* it has three failures:

1. It surfaces **redundant** chunks (semantically similar = often the same idea).
2. It loses **structure** (hierarchy, prerequisites, "see also").
3. It cannot represent **depth** ("explain at high level" vs "deep-dive").

**GraphRAG** combines vector search with a **concept knowledge graph** built per tribe. Nodes are concepts (extracted from chunks), edges are typed relationships (`IS_PREREQUISITE_OF`, `IS_EXAMPLE_OF`, `CONTRASTS_WITH`, `SEE_ALSO`). Retrieval traverses the graph, anchored by vector similarity, controlled by the user's requested *depth*.

This mirrors the team's "Obsidian LLM" / Karpathy-style note-graph idea, made queryable.

---

## 2. Architecture

```mermaid
flowchart LR
    Upload([📥 Upload PDF/URL]) --> Parse[Parse to Markdown]
    Parse --> Chunk[Chunk by heading + token cap]
    Chunk --> Embed[Embed each chunk]
    Embed --> Store[(pgvector chunks)]
    Chunk --> Concept[LLM extracts concepts<br/>and relations]
    Concept --> Graph[(Apache AGE graph<br/>per-tribe label)]
    Graph -. links to .-> Store

    Query([🔍 RAG query]) --> Vec[Vector ANN]
    Query --> Lex[Lexical BM25]
    Vec --> Hybrid[RRF fusion]
    Lex --> Hybrid
    Hybrid --> Anchor[Anchor concept nodes]
    Anchor --> Traverse[Graph traversal<br/>depth = summary | standard | deep_dive]
    Traverse --> Rerank[Cross-encoder re-rank]
    Rerank --> Synthesise[LLM synthesis<br/>with citations]
```

### Components in Postgres

- **`pgvector`** — embeddings (1536-d, cosine).
- **`pg_trgm` + `tsvector`** — lexical / fuzzy.
- **`Apache AGE`** — openCypher graph queries, per-tribe graph label `g_tribe_<uuid>`.

Why all in Postgres? One transactional boundary, RLS-consistent, no cross-system staleness, lower ops cost.

---

## 3. Ingestion pipeline

### Step 1 — Parse to Markdown

| Input | Tool |
|-------|------|
| PDF | `pdf-parse` for text + `pdf2json` for layout; OCR via Tesseract WASM as fallback |
| EPUB | `epub2md` |
| URL | `mozilla/readability` server-side |
| Audio (visio) | Web Speech captured client-side or Whisper fallback |

Output: clean Markdown with preserved heading levels and code fences.

### Step 2 — Chunk

- **Heading-aware**: split by `H1/H2/H3` boundaries.
- **Token cap**: 600 tokens per chunk, 80-token overlap.
- **Min chunk**: 80 tokens (merge with previous if smaller).
- Each chunk stores `ordinal`, `title_path` (e.g. `Chap 3 > JWT > Refresh`), `token_count`.

### Step 3 — Embed

- Default: `text-embedding-3-small` (1536-d, OpenAI).
- Local fallback: `all-MiniLM-L12-v2` via Transformers.js (384-d, separate index).
- Embeddings stored in `resource_chunks.embedding`.

### Step 4 — Concept extraction & graph linking

A specialised LLM pass per chunk produces:

```json
{
  "concepts": [
    { "name":"JWT Refresh Token", "salience":0.8 },
    { "name":"OAuth 2.0", "salience":0.5 }
  ],
  "relations": [
    { "from":"JWT Refresh Token", "to":"JWT Access Token", "type":"COMPLEMENTS" },
    { "from":"JWT Refresh Token", "to":"OAuth 2.0",        "type":"IS_USED_IN" }
  ]
}
```

Cypher (AGE) inserts:

```cypher
MERGE (a:Concept {name:'JWT Refresh Token', tribe_id:$tid})
  ON CREATE SET a.first_seen=$now
SET a.salience = COALESCE(a.salience,0) + 0.8
WITH a
MATCH (b:Concept {name:'JWT Access Token', tribe_id:$tid})
MERGE (a)-[r:COMPLEMENTS]->(b)
SET r.weight = COALESCE(r.weight,0) + 1
```

Concept nodes carry `chunk_ids` array — links from graph back to source.

### Step 5 — Tribe scoping

- `resource_chunks.tribe_id` is denormalised; **all** vector queries filter by `tribe_id` via RLS.
- Apache AGE labels are per-tribe (`g_tribe_<uuid>`), giving an extra physical isolation.
- Periodic verifier job runs cross-tribe similarity tests with synthetic markers to detect leakage (P0 incident if found).

---

## 4. Retrieval pipeline

### 4.1 Hybrid search

- **Vector**: top-k=20 by cosine.
- **Lexical**: top-k=20 via `ts_rank` + trigram boost.
- **Fusion**: Reciprocal Rank Fusion (RRF) k=60, returns top-30.

### 4.2 Concept anchoring

For each top chunk, look up its tagged concepts in the tribe graph. The set of anchored concepts becomes the *seed set*.

### 4.3 Depth-controlled traversal

| Depth | Traversal |
|-------|-----------|
| `summary` | Take only top-3 concepts by graph PageRank within seed set |
| `standard` | 1-hop expansion, prune by edge weight ≥ 0.3 |
| `deep_dive` | 2-hop expansion, include `IS_EXAMPLE_OF`, `CONTRASTS_WITH`, `SEE_ALSO` |

### 4.4 Cross-encoder re-rank

Final candidates re-ranked by `bge-reranker-base` (or Cohere Rerank API) for relevance to original query. Top-5 to 12 chunks fed to synthesis.

### 4.5 Synthesis

The Curator agent receives:

```json
{
  "query": "...",
  "depth": "standard",
  "format": "prose",
  "chunks": [{ id, title_path, content_md, score }],
  "concept_path": [{ from, to, type, weight }]
}
```

It produces a markdown answer + inline citations `[1]`, `[2]` referring to chunks. The graph path is returned so the UI can render the mini-map.

---

## 5. Personalisation: depth + style

The tribe (or solo user) selects:

- **Depth**: `summary | standard | deep_dive`
- **Style**: `prose | bullets | flashcards | mcq | mindmap | glossary | eli12`

These map to prompt templates in `prompts/curator/*.md`.

---

## 6. Pedagogy generation pipeline

Same retrieval, different synthesis:

| Format | Strategy |
|--------|----------|
| **Flashcards** | One front/back per atomic fact; concept tag per card |
| **MCQ** | Distractors mined from neighbouring graph nodes (semantically near, factually distinct) |
| **Mind-map** | Top concepts + edges in Mermaid; UI also renders interactive Cytoscape |
| **Glossary** | One row per concept with first-citation chunk |
| **Summary** | Graph PageRank-weighted bullets |

All outputs validated against strict Zod schemas; auto-repair on failure.

---

## 7. Update / re-ingestion

- Resources are immutable; edits produce a new resource version with shared concept identity.
- Stale embeddings detected when embedding model changes — re-embed asynchronously.
- Concept merge: when two concept nodes in the same tribe have cosine similarity ≥ 0.92 by name+context, they merge with audit log.

---

## 8. Privacy & isolation

- Vector index queries always include `WHERE tribe_id = ?`.
- AGE graphs use distinct labels per tribe — cross-label queries impossible without explicit JOIN.
- LLM prompts include only redacted, tribe-scoped chunks.
- Storage objects (`resources/{tribe_id}/{resource_id}/...`) follow a path-based RLS policy.

---

## 9. Performance budgets

| Stage | p95 |
|------|----:|
| Chunking + embed (per page) | < 200 ms |
| Concept extraction (per chunk) | < 800 ms |
| Hybrid retrieval | < 80 ms |
| Re-rank | < 250 ms |
| End-to-end (query → first token) | < 1.2 s |

Caching:
- LRU on (query, tribe_id, depth, format) with 10-min TTL for identical queries.
- Embeddings cached deterministically by content hash.

---

## 10. Failure modes

| Failure | Behaviour |
|---------|-----------|
| Embedding API down | Queue chunks, fall back to lexical-only retrieval, banner |
| Concept extraction fails | Chunk still embedded; concept extraction retried in background |
| Graph extension corruption | Read-only mode, alert ops |
| Query returns 0 chunks | Curator answers honestly: "No tribe-specific resource yet — would you like me to add three from public sources?" |

---

## 11. Why not Pinecone / Weaviate / Qdrant?

- Operational simplicity (one DB).
- Strong RLS story.
- Cost: free up to fairly large scale on Supabase.
- Postgres tooling (pg_dump, replication, PITR) "just works".

When pgvector becomes a bottleneck (>10 M chunks), migration path is documented in [adr/0002-pgvector-graphrag.md](adr/0002-pgvector-graphrag.md).

---

See [09_PEDAGOGICAL_ENGINE.md](09_PEDAGOGICAL_ENGINE.md) for what we do with the generated flashcards once they exist.
