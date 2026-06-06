# Complete RAG Architecture — Scratch to End

## Overview

A complete RAG (Retrieval-Augmented Generation) system consists of:

1. Ingestion Pipeline – prepares enterprise knowledge
2. Runtime / Query Pipeline – answers user questions using that knowledge
3. Governance Layer – provides security, observability, evaluation, and compliance

---

# Ingestion Pipeline (Offline)

```text
ENTERPRISE DATA SOURCES
SharePoint | Confluence | PDFs | Databases | APIs | Websites | Tickets
        ↓
DATA INGESTION
Connectors / Crawlers / Loaders
        ↓
DOCUMENT PROCESSING
Parse documents
Clean text
Extract tables/images if needed
Normalize formats
        ↓
CHUNKING
Split documents into retrievable pieces
        ↓
METADATA ENRICHMENT
Add source, owner, department, country, security level, date, version
        ↓
EMBEDDING MODEL
Convert each chunk into a vector
        ↓
VECTOR DATABASE / SEARCH INDEX
Store:
- Chunk text
- Vector
- Metadata
- Source reference
        ↓
INDEX READY
Knowledge is now searchable
```

---

# Runtime Query Pipeline (Online)

```text
USER QUESTION
        ↓
IDENTITY & ACCESS CONTEXT
Who is asking?
Role?
Department?
Permissions?
        ↓
QUERY PROCESSING
Clean query
Detect intent
Maybe rewrite query
        ↓
QUERY EMBEDDING
Embedding model converts question into vector
        ↓
METADATA / SECURITY FILTERS
Only search allowed documents
        ↓
RETRIEVAL
Vector Search
Keyword Search
Hybrid Search
ANN Search
        ↓
CANDIDATE RESULTS
Top 50 / Top 100 chunks
        ↓
RE-RANKING
Cross Encoder or other re-ranker scores best chunks
        ↓
TOP CHUNKS
Top 5 / Top 10
        ↓
CONTEXT ASSEMBLY
Deduplicate
Order
Compress
Select final context
        ↓
PROMPT CONSTRUCTION
System instructions
User question
Retrieved context
Citation rules
Safety instructions
        ↓
LLM
Generate grounded answer
        ↓
POST-PROCESSING
Citations
Formatting
Safety checks
Policy checks
        ↓
USER ANSWER
```

---

# End-to-End Architecture

```text
                ┌──────────────────────────────┐
                │      Enterprise Sources       │
                │ Docs | DBs | APIs | Web | PDF │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │        Ingestion Layer        │
                │ Load | Parse | Clean | Chunk  │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │      Metadata Enrichment      │
                │ Owner | ACL | Country | Type  │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │        Embedding Model        │
                │       Text → Vector           │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │ Vector DB / Search Platform   │
                │ Vector + Text + Metadata      │
                └───────────────┬──────────────┘
                                ↓
━━━━━━━━━━━━━━━━━━━━━━ RUNTIME ━━━━━━━━━━━━━━━━━━━━━━
                                ↓
                ┌──────────────────────────────┐
                │        User Question          │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │ Identity / Authorization      │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │ Query Embedding + Search      │
                │ Vector / Keyword / Hybrid     │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │ Metadata Security Filtering   │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │ Re-ranking / Cross Encoder    │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │ Context Assembly              │
                │ Select | Order | Compress     │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │ Prompt Construction           │
                │ Question + Context + Rules    │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │              LLM              │
                │        Generate Answer        │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │ Guardrails / Citations / Logs │
                └───────────────┬──────────────┘
                                ↓
                ┌──────────────────────────────┐
                │          Final Answer         │
                └──────────────────────────────┘
```

---

# Key Architecture Separation

```text
Ingestion Pipeline
= Prepares Knowledge

Retrieval Pipeline
= Finds Relevant Knowledge

Context Pipeline
= Prepares Knowledge For The LLM

Generation Pipeline
= Produces The Answer

Governance Layer
= Controls Safety, Access, Logging, Evaluation
```

---

# The Three Main AI Models

```text
Embedding Model
    Output: Vector

Re-ranking / Cross Encoder Model
    Output: Relevance Score

LLM
    Output: Natural Language Answer
```

---

# Major Architecture Insight

```text
RAG is not:

Vector DB + LLM

RAG is:

Data Ingestion
+ Indexing
+ Retrieval
+ Ranking
+ Context Assembly
+ Prompt Construction
+ Generation
+ Governance
```
