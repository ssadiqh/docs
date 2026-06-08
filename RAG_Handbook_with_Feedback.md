# RAG Core Handbook — Learner-Sequenced Edition

---

## Contents

1. Why RAG Exists  
2. RAG vs. Alternatives — Should You Even Use RAG?  
3. End-to-End Architecture & the Cascade  
4. Stage-by-Stage Deep Dive  
5. Architecture Patterns — From Naive to Adaptive  
6. Security Architecture & Failure Analysis  
7. Evaluation & Observability  
8. Cost Model & Tooling Map  
9. Enterprise Reference Architecture — The Platform View  
10. Quick-Recall Companion — Mental Models & One-Liners  
11. One-Page Summary  
12. Solutions Architect Playbook — Requirements Gathering  
    Appendix A. Staged for Future Handbook — Build vs. Buy, Phased Rollout, Portfolio View  
    Appendix B. Changelog — What Changed in This Revision

---

## 1\. Why RAG Exists

LLMs alone have three problems: missing enterprise knowledge, stale knowledge, and hallucination. RAG inserts a retrieval step between question and model — Enterprise Documents → RAG → LLM — so the model writes from the right facts instead of guessing.

**Quick Recall:**

Without RAG:  
Question → LLM → Guess

With RAG:  
Question → Retrieve Facts → LLM → Answer

---

## 2\. RAG vs. Alternatives — Should You Even Use RAG?

| Approach | What it does | Best when | Weak when |
| :---- | :---- | :---- | :---- |
| RAG | Retrieves relevant external knowledge at query time, feeds it to the LLM as context | Knowledge changes often, must be traceable/citable, too large for a prompt or model weights | Latency tolerance is very tight, or "knowledge" is really a skill (style/format/reasoning), not a fact |
| Fine-tuning | Adjusts model weights on your data | Need consistent style, domain language, structured outputs, narrow specialized skill | Knowledge changes often (constant retraining), or you need traceability/citations |
| Long-context | Puts source material directly in the prompt, no retrieval | Small, stable corpora that fit in context | Large/changing corpora — cost & latency scale with every query |
| Agents | Plans, calls tools (RAG can be one), takes multi-step action | Task needs multi-step reasoning, API calls, combining sources/actions | Task is a single grounded Q\&A — agent adds needless latency/complexity |
| Workflow / deterministic pipeline | Fixed, code-defined sequence, LLM/RAG as a component | Process is well-understood, repeatable, needs auditability | Task is open-ended / needs judgment that can't be pre-scripted |
| Plain enterprise search | Keyword/metadata search returns documents to a human | User wants to find and read a document themselves | User wants a synthesized conversational answer across many documents |
| Knowledge graphs | Entities \+ explicit relationships; supports multi-hop queries | Questions are about how things relate | Knowledge is mostly unstructured prose where relationships aren't the point |

**Decision shortcut**

- Up-to-date / large external knowledge that needs traceability? → RAG (+ graph if relationships matter)  
- Need a different skill/style/format from the model itself? → Fine-tuning  
- Small, stable corpus? → Long context, skip retrieval  
- Multi-step reasoning / tool use? → Agent (RAG as one of its tools)

**RAG \+ Fine-Tuning — complements, not just alternatives:** the table above compares them as competing options, but in production they're frequently combined rather than chosen between.

| Technique | Changes | Answers the question |
| :---- | :---- | :---- |
| Fine-tuning | Model *behaviour* — style, format, domain language, narrow skills | "How should the model respond?" |
| RAG | Model *knowledge* — the facts available at answer time | "What does the model know right now?" |

**Relationship:** fine-tuning changes *how* the model behaves; RAG changes *what* the model knows. A common production pattern is a model fine-tuned on domain style/format, fed by a RAG pipeline for current facts — neither one substitutes for the other.

**One-liner:** "RAG answers 'what do our documents say'; fine-tuning changes 'how the model behaves'; agents decide 'what to do next'; knowledge graphs answer 'how things relate.' Usually combined, rarely substitutes."

---

## 3\. End-to-End Architecture & the Cascade

### 3.1 Runtime Pipeline

**Offline / ingestion side:** `ENTERPRISE SOURCES → CONNECTORS → PARSING / EXTRACTION → CLEANING / NORMALIZATION → CHUNKING → METADATA + ACL ENRICHMENT → EMBEDDING → INDEXING → REFRESH / DELETE / PERMISSION SYNC → VECTOR DB / SEARCH INDEX`

*— runtime boundary —*

**Runtime / query side:** `USER QUESTION → IDENTITY / AUTH → CONVERSATION CONTEXT HANDLING → QUERY PROCESSING / REWRITE → QUERY EMBEDDING → CACHE CHECK → SECURITY FILTER → RETRIEVAL → RE-RANKING → CONTEXT ASSEMBLY → PROMPT CONSTRUCTION → LLM → CITATIONS / GUARDRAILS / LOGGING → FINAL ANSWER`

**Feedback loop:** `TRACES + USER FEEDBACK + EVALUATION → TUNE CHUNKING / RETRIEVAL / PROMPTS / INDEXING`

*Note: Section 7 owns Evaluation & Observability in detail. The feedback loop is shown here only to illustrate how production systems continuously improve over time.*

| Pipeline | Job | One-liner |
| :---- | :---- | :---- |
| Ingestion | Prepares knowledge | Garbage in, garbage out |
| Retrieval | Finds relevant knowledge | Optimizes recall |
| Context | Prepares knowledge for the LLM | Optimizes signal-to-noise |
| Generation | Produces the answer | Only as good as what it's handed |
| Governance | Controls safety, access, logging, evaluation | Separates a demo from a product |

**Source of Truth — a relationship, not just a principle:** the pipeline above starts at "Enterprise Sources" for a reason — the RAG index is a *derived, lossy copy* of those sources, never the authoritative record.

`Source Systems (SharePoint · Confluence · Databases · Tickets) ⇄ RAG Index`

- *Source systems:* remain the system of record — ownership, edit rights, and authoritative permissions live there.  
- *RAG index:* a queryable reflection of that knowledge at the moment it was last ingested — always at risk of drifting out of sync with the source.

**Principle:** RAG indexes knowledge; it does not own knowledge. Every freshness problem, every permissions-drift bug (Section 6), and every "the document says X but the system answered Y" complaint traces back to this relationship — the index quietly fell out of step with its source of truth.

### 3.2 The Cascade Principle

`Source Quality → Chunking → Embedding → Retrieval → Re-ranking → Context Assembly → Prompt Construction → Answer`

**The rule:** Quality lost early cannot be recovered later. RAG is a lossy pipeline — each stage can only preserve or destroy what it received. A perfect LLM cannot answer from context that never contained the right chunk; a perfect retriever cannot find a chunk that chunking destroyed.

This is the lens for the rest of the handbook: every stage in Section 4, every failure symptom in Section 6, and every "why is the answer bad" conversation reduces to *which link in this chain lost the information first*.

**Quick Recall:** The whole pipeline \= a lossy chain — each link can only preserve or destroy what it received. "Quality lost early in the pipeline cannot be recovered later — a perfect LLM can't fix a chunk that was never retrieved."

### 3.3 Component Relationships

A second cross-cutting relationship, beyond the cascade itself: Identity/Authorization and Metadata Filtering define the search space retrieval is allowed to operate in — get them wrong and you don't get a slightly-worse answer, you get a wrong or unsafe one.

These component-to-component relationships recur throughout Section 4 and are worth previewing here, since they're what an architect actually reasons with day to day:

- **Chunking ↔ Embedding:** chunking determines *what* gets embedded; embedding determines *what can be retrieved*. A bad split can't be fixed by a good embedding model.  
- **Metadata ↔ Security:** security filtering depends on metadata — bad metadata *is* bad security, not just bad search.  
- **Retrieval ↔ Re-ranking:** retrieval optimizes recall (cast a wide net); re-ranking optimizes precision (judge the net's contents). Neither can substitute for the other.  
- **Context Assembly ↔ Prompt Construction:** context assembly decides *what* the model sees; prompt construction decides *how* it sees it.

---

## 4\. Stage-by-Stage Deep Dive

*Each stage: Purpose → Trade-off → Failure Signature → Quick Recall analogy.*

### 4.1 Document Loading / Ingestion

- Pulls raw content (SharePoint, Confluence, PDFs, DBs, APIs, web, tickets) and normalizes it.  
- Decide on: how messy your sources are — templated sources need light processing, scans/slides need OCR/structure-aware extraction.  
- Trade-off: source breadth vs. parsing fidelity.  
- Failure looks like: garbled/truncated answers that look like a retrieval bug but are an ingestion bug.  
- One-liner: sets the ceiling for everything downstream.

**Typical ingestion responsibilities include:**

- Connector integration (SharePoint, Confluence, databases, APIs, file systems)  
- Parsing and extraction of text, tables, images, PDFs, slides, and scanned content  
- OCR where required  
- Cleaning and normalization (removing boilerplate, fixing encoding issues, normalizing formats)

**Failure looks like:** garbled, incomplete, duplicated, or poorly extracted content that later appears to be a retrieval issue but actually originated during ingestion.

*Relationship reminder: ingestion is the first link in the Source-of-Truth relationship (3.1) — it's the moment the authoritative source becomes a derived copy, and the moment drift starts accumulating.*

### 4.2 Chunking

- Splits documents into retrievable units.  
- → Need simplicity/speed? Fixed-size → Need a balanced default? Recursive → Need max precision on prose? Semantic → Source strongly structured? Structure-aware → Facts need surrounding context? Parent-Child pattern  
- Trade-off: small \= precision, large \= context.  
- Failure looks like: "almost right" retrievals — topically related but missing the specific fact, or the fact split across chunks.  
- One-liner: the single highest-leverage tuning knob.

**Quick Recall:** Chunk \= Book chapter

**Chunking Decision & Trade-off Table**

| Approach | Precision | Context preserved | Complexity | Best when |
| :---- | :---- | :---- | :---- | :---- |
| Fixed-size | Medium | Medium | Low | Speed/simplicity matter more than nuance; uniform source format |
| Recursive | High | Medium | Medium | The balanced default for mixed enterprise content |
| Semantic | High | High | High | Prose-heavy sources where topical coherence drives retrieval quality |
| Structure-aware | High | High (structural) | Medium–High | Strongly structured sources (manuals, contracts, specs) |
| Parent-Child | Very High | Very High | High | Precise facts need surrounding context — resolves the precision-vs-context trade-off by doing both, at different stages |

**Practical starting points** *(defaults to validate against your own domain and embedding model, not fixed rules — optimal ranges shift with embedding-model context limits and content density)*:

- Recursive chunking as a sensible default for mixed enterprise corpora  
- Roughly 500–1000 tokens per chunk as a starting range  
- Roughly 10–20% overlap between chunks as a starting range  
- Validate against a golden retrieval set (Section 7\) before committing — the "right" numbers are domain- and embedding-model-specific, and the only way to know is to measure

**Relationship — Chunking ↔ Embedding:** chunking determines *what* gets embedded; the embedding model determines *what can be retrieved*. This is why chunking is called the highest-leverage knob — everything downstream (embedding quality, retrieval quality, final-answer quality) inherits whatever chunking handed it, for better or worse.

### 4.3 Metadata Enrichment

- Attaches owner, department, country, classification, source, type, date/version — the basis of both relevance and access control.  
- Decide: let security/compliance set the minimum schema; let search-quality needs add the rest.  
- Trade-off: richer metadata \= finer security/relevance, but more complexity and a permissions-sync problem.  
- Failure looks like: over-exposure or false "I don't have that" responses.  
- One-liner: where "search" becomes "governed enterprise search."

**Relationship — Metadata ↔ Security:** security filtering (4.9, Section 6\) operates entirely on metadata — it can only enforce what the metadata correctly records. Bad or stale metadata isn't a search-quality nuisance; it *is* a security failure, just one that surfaces as a wrong answer instead of an error message.

### 4.4 Embedding Model

- Converts chunks/queries into vectors.  
- Decide: match to dominant languages and domain; weigh hosted-API convenience vs. self-hosted cost/control at your query volume.  
- Trade-off: higher dimensions often retrieve better but cost more to compute/store/search.  
- Failure looks like: confidently-wrong "semantically similar but factually unrelated" results.  
- One-liner: defines the "language" your retrieval thinks in.

**Quick Recall:** Embedding \= Customer profile

### 4.5 Vector Database / Search Platform

- Stores text \+ vectors \+ metadata; serves similarity search.  
- Decide: existing stack (Postgres → pgvector; OpenSearch shop → OpenSearch) vs. need for graph+vector (Neo4j) vs. fully managed scale (Pinecone/Weaviate Cloud/Azure AI Search).  
- Trade-off: managed \= less ops, more cost/lock-in; self-hosted \= more control, more ops ownership.  
- Failure looks like: latency or relevance degradation as the corpus grows (usually an indexing/tuning issue).  
- One-liner: pick for your stack and ops appetite — most are "good enough" on raw retrieval quality.

**Quick Recall:** Vector DB \= Semantic search engine

### 4.6 ANN Indexing

- Builds fast approximate-search structures (HNSW, IVF).  
- Trade-off: small accuracy loss for large speed gain.  
- Failure looks like: inconsistent results on near-identical queries, or slowdowns past tuning assumptions.  
- One-liner: "approximately right, very fast" — explicitly chosen.

### 4.6a Index Lifecycle & Data Freshness

Building the index is only half the architecture story — the other half is *keeping it true to its source of truth* (3.1). A document changes; until the index catches up, the system is confidently answering from a stale copy.

`Document Updated → Change Detection → Re-Chunking → Re-Embedding → Index Update`

**Refresh strategy options & trade-offs**

| Approach | Freshness | Cost | Complexity | Best when |
| :---- | :---- | :---- | :---- | :---- |
| Batch (scheduled re-index) | Lower — staleness window between runs | Lowest | Lowest | Source content changes slowly; staleness window is acceptable |
| Incremental (delta updates / CDC) | Balanced | Medium | Medium | Most enterprise corpora — change is frequent but not constant |
| Real-time (event-driven) | Highest | Highest | Highest | Regulatory, safety, or fast-moving content where staleness itself is the risk |

**Architectural principle:** a RAG system is only as current as its indexing strategy — and that strategy is a deliberate freshness-vs-cost-vs-complexity choice, not a default you inherit from your vector platform.

**Index maintenance includes more than refreshes and updates.**

Typical lifecycle operations include:

- Refresh / re-index  
- Incremental updates  
- Delete / de-index removed content  
- Permission synchronization with source systems

**Failure looks like:**

- Deleted documents still appearing in answers  
- Revoked permissions continuing to expose content  
- Source systems and RAG indexes drifting apart

(*Original failure symptom:* "the document was updated last week but the assistant still describes the old policy" — easy to misdiagnose as a retrieval or generation bug when it's actually an index-lifecycle gap.)

### 4.7 Identity & Access Context

- Establishes who's asking before retrieval happens — the entry point to the security architecture covered later in this edition.  
- Trade-off: query-time enforcement costs more per query than ingest-time-only, but is the only correct way to handle permissions that change post-indexing.  
- Failure looks like: the same question getting different answers depending on when it's asked.  
- One-liner: skip it and a helpful assistant becomes a data-leak incident.

### 4.8 Query Processing & Embedding

- Cleans, interprets intent, optionally rewrites/expands, then embeds the query (this is where Multi-Query RAG plugs in).  
- Trade-off: rewriting improves recall on vague questions but adds latency and intent-drift risk.  
- Failure looks like: the system answering a related but different question.  
- One-liner: a well-formed query is half the retrieval battle.

**Conversation Context Handling**

In chat-based RAG systems, recent conversation history may be incorporated before retrieval. This helps resolve references such as "What about the previous policy?" or "How does that compare?"

- Trade-off: More context improves continuity but increases token usage and can introduce retrieval drift if irrelevant history is included.

### 4.8a Semantic / Response Cache

Many production RAG systems check a cache before retrieval.

`Question → Cache Check → Retrieval (if required)`

**Benefits:**

- Lower latency  
- Reduced retrieval costs  
- Reduced LLM costs

**Trade-off:**

Cached answers must be invalidated when source content changes, otherwise stale responses may be returned.

**One-liner:** The cheapest retrieval is the one you do not need to perform.

### 4.9 Metadata / Security Filtering

- Narrows the search space to authorized documents — see Section 6 for the underlying models (RBAC/ABAC/ACL/row-level/tenant isolation).  
- Decide: pre-filter (before vector search — more secure/efficient, needs platform support) vs. post-filter (simpler, riskier).  
- Failure looks like: "found something relevant but can't show it" loops, or silent leakage only caught in audit.  
- One-liner: security and relevance working as one mechanism.

**Permission Synchronization is Critical**

Security filtering can only enforce permissions that have been accurately synchronized from source systems. A stale permissions model may expose unauthorized content or incorrectly deny access to legitimate users.

**One-liner:** Permission drift is one of the most common enterprise RAG security failures.

*Relationship reminder: this stage is where the Metadata ↔ Security relationship (4.3) becomes operational — it can only filter as well as the metadata underneath it is accurate and current.*

### 4.10 Retrieval — Vector / Keyword / Hybrid

- Casts a wide net — typically top 50–100 — via vector similarity and/or keyword matching (this is the choice between Naive and Hybrid RAG).  
- Failure looks like: the right document exists, is correctly chunked and tagged, but never appears in the candidate list — pure recall failure.  
- One-liner: over-retrieve first, refine later — this stage's job is recall.

**Quick Recall:** Retriever \= Long-list candidate search

**Retrieval Strategy Selection — consolidated decision table**

| Strategy | Best for | Semantic strength | Exact-match strength | Relative cost |
| :---- | :---- | :---- | :---- | :---- |
| Keyword search | Codes, IDs, acronyms, exact terms | Low | High | Low |
| Vector search | Natural-language, paraphrased, semantic questions | High | Low | Medium |
| Hybrid search | Mixed / enterprise-general content — almost all real enterprise corpora | High | High | Higher (runs both systems) |

**Default recommendation:** Hybrid — it's the 2026 production baseline (see Section 5.2); reach for pure vector or pure keyword only when you're confident the corpus and question style are narrowly one-sided.

**Relationship — Retrieval ↔ Re-ranking:** retrieval optimizes *recall* — cast the widest net that still contains the right answer; re-ranking optimizes *precision* — judge that net's contents and keep only the best few. Neither stage can do the other's job: a retriever that tries to be precise will miss candidates, and a reranker can't promote a chunk that was never retrieved in the first place.

### 4.11 Re-ranking

- Scores candidates jointly (cross-encoder: question \+ chunk together) and narrows to the best few.  
- Decide: cross-encoder for cheap, consistent reranking of large sets; LLM-as-reranker only for small sets needing nuanced judgment, accepting the latency/cost.  
- Trade-off: cross-encoders beat embedding-similarity ranking on accuracy (joint attention) but are too slow/costly to run over a whole corpus.  
- Failure looks like: the right chunk was retrieved (visible in logs) but didn't survive ranking — easy to misdiagnose as a retrieval bug.  
- One-liner: retriever finds candidates, reranker judges them — long-list vs. interview panel.

**Why the split exists:** embedding models compare the question and each chunk *separately*, then measure similarity between the two representations; cross-encoders compare them *jointly*, attending to both at once. That joint attention is why cross-encoders win on relevance — and why they're too expensive to run over an entire corpus, only over the retriever's shortlist. This is one of the most common architecture-interview discussions on RAG, precisely because it's the cleanest illustration of the recall-vs-precision split between the two stages.

**Quick Recall:** Reranker \= Interview panel. "Cross-encoders compare jointly, embedding models compare separately — that's why cross-encoders win on relevance and lose on speed."

### 4.12 Context Assembly

- Dedupes, orders, compresses top results into the token-budget-respecting final context.  
- Trade-off: more context \= more completeness but higher cost / "lost in the middle" risk; tighter \= cheaper/sharper but can omit detail.  
- Failure looks like: answer cites the wrong source among several correct ones, or blends sources incorrectly.  
- One-liner: where retrieval quality is preserved — or quietly diluted — right before the model sees it.

**Quick Recall:** Context Window \= RAM

**The funnel, visualized:**

`Top 50 Retrieved Chunks → Re-ranking → Top 10 Chunks → Deduplication → Compression → Ordering → Final Context`

| Stage | Purpose |
| :---- | :---- |
| Top 50 Retrieved | Maximize recall |
| Re-ranking | Improve relevance |
| Top 10 Chunks | Keep strongest candidates |
| Deduplication | Remove repeated information |
| Compression | Reduce token usage |
| Ordering | Improve model attention |
| Final Context | Context sent to the LLM |

Every stage in this funnel can remove information. A chunk being retrieved does not guarantee it reaches the LLM. This is why Context Assembly is a first-class architecture component rather than a simple formatting step.

Each step in this funnel is a place where a correct chunk can still be lost — which is exactly why Context Assembly shows up as its own row in the Failure Analysis table (Section 6): "the right chunk was retrieved" and "the right chunk made it into the answer" are two different claims, and this funnel is the gap between them.

**A note on trust — retrieved content is data, not instructions:** everything that survives this funnel becomes part of the prompt the LLM sees. That means retrieved chunks are *untrusted input* in the same sense that user input is — a document that happens to contain text phrased as an instruction ("ignore previous instructions and...") should be treated as a quoted fact to reason about, never as a command to follow. The architectural habit worth building here is simple: treat retrieved content as data to cite, not as instructions to obey. (The full attack-surface treatment — indirect prompt injection, exfiltration patterns, mitigations — belongs in a dedicated security/governance treatment; the point to internalize at the pipeline-design level is just that this boundary exists and has to be designed for.)

#### Context Assembly — Ownership and Strategies

**Who actually does this?** Context Assembly is usually performed by the orchestration layer — LangChain, LlamaIndex, Haystack, Spring AI, or custom application code — rather than by a dedicated AI model. That layer decides which chunks to keep, which to drop, how to order them, whether to compress or summarize them, and how much fits inside the token budget. Context Assembly is often logic, not a model.

**The strategies it applies:**

| Strategy | Who does it? | Purpose |
| :---- | :---- | :---- |
| Top-K | Retriever / app code | Select the best N chunks |
| Deduplication | App code | Remove repeated chunks |
| Ordering | App code | Place chunks where the model is most likely to attend to them |
| Compression | Rules / embeddings / small LLM | Strip irrelevant text while preserving the original wording |
| Summarization | LLM | Rewrite chunks into a shorter form |

**Compression vs. Summarization** — easy to conflate, but architecturally distinct choices with different risk profiles:

| Factor | Compression | Summarization |
| :---- | :---- | :---- |
| Preserves original text | Yes | No — rewrites it |
| Uses an LLM | Usually no | Yes |
| Hallucination risk | Very low | Higher |
| Accuracy | Very high | Medium–high |
| Token reduction | Medium | Very high |

**One-liner:** Compression preserves facts; summarization rewrites facts. Reach for compression when traceability matters, summarization when the token budget is the binding constraint.

**The optional fourth model:** the pipeline is usually described with three models — embedding, reranker, generation LLM. Context Assembly can introduce an optional fourth.

**Standard RAG model flow:**

`Question → Embedding Model → Retrieval → Re-ranking Model → Context Assembly → Main LLM → Answer`

**When summarization is added,** Context Assembly forks into two possible handling paths before the answer is generated — compression (rules/embeddings, no model) or summarization (a small LLM rewriting chunks):

`Question → Embedding Model → Retrieval → Re-ranking Model → Context Assembly → Compression (rules / embeddings) → Main LLM → Answer`

`Question → Embedding Model → Retrieval → Re-ranking Model → Context Assembly → Summarization LLM → Main LLM → Answer`

Most RAG systems use three models:

1. Embedding Model  
2. Re-ranking Model  
3. Generation LLM

When summarization is introduced during Context Assembly, a fourth model may appear:

4. Summarization LLM

This model is optional and exists solely to reduce context size before generation.

| Model | Purpose |
| :---- | :---- |
| Embedding Model | Convert text into vectors |
| Re-ranking Model | Improve retrieval precision |
| Summarization Model (optional) | Reduce context size |
| Generation LLM | Produce the final answer |

Not every RAG system contains four models, but enterprise-scale systems often do when token budgets become a significant cost or latency constraint.

**Lost-in-the-Middle, in more depth:** LLMs tend to pay more attention to information near the *beginning* and *end* of the supplied context than to information buried in the middle. This makes ordering a quality lever, not a cosmetic detail — Context Assembly may deliberately place the most critical chunks first or last so the model is more likely to attend to them, rather than simply preserving retrieval-rank order.

**Relationship to Prompt Construction:**

Retrieval → Re-ranking → Context Assembly → Prompt Construction → LLM

Retrieval finds information. Context Assembly decides what the LLM sees. Prompt Construction decides how the LLM sees it.

### 4.13 Prompt Construction

- Packages instructions, question, context, citation/safety rules.  
- Trade-off: more explicit instruction \= more consistency, but consumes context budget.  
- Failure looks like: correct context assembled, but the answer doesn't cite it / mis-cites it / ignores instructions.  
- One-liner: the contract between your retrieval pipeline and the model.

### 4.14 Generation — LLM

- Produces the grounded answer.  
- Trade-off: bigger models reason better over context but cost more, respond slower, and cannot fix weak retrieval.  
- Failure looks like: fluent, confident, unsupported answers despite good retrieval — a prompting/model-choice issue, not a retrieval issue.  
- One-liner: the final decision maker, not the whole process.

**Quick Recall:** LLM \= Final decision maker

### 4.15 Post-Processing / Guardrails

- Adds citations, formatting, safety/policy checks, logging.  
- Trade-off: more checks \= less risk, more latency, occasional over-blocking.  
- Failure looks like: good answers getting blocked or stripped — the "too safe to be useful" failure mode.  
- One-liner: the seatbelt — invisible when things go right.

---

## 5\. Architecture Patterns — From Naive to Adaptive

Components explain the parts. Patterns explain how architects assemble those parts to fit a problem. This is now one of the most common interview and design-review topics.

### 5.1 Naive (Basic) RAG

`Question → Embed → Vector Search → Top-K → Prompt → LLM`

- **Best when:** Simple FAQ-style corpora, prototypes, low-stakes internal tools.  
- **Weak when:** Questions are vague, exact-term-heavy, or multi-hop — naive RAG misses all three.  
- **One-liner:** The "hello world" of RAG — a fine starting point, a poor production end-state.

**Quick Recall:** Naive RAG \= Asking one person one question, once

### 5.2 Hybrid RAG

`Question → [Vector Search + Keyword Search] → Fusion → Re-rank → Prompt → LLM`

- **Best when:** Enterprise corpora with a mix of natural language and exact terms (codes, IDs, names, acronyms) — i.e., almost all real enterprise content.  
- **Weak when:** Adds infrastructure cost/complexity that small, narrow-domain systems may not need.  
- **One-liner:** The 2026 production baseline — most enterprises should start here, not with naive RAG.

### 5.3 Multi-Query RAG

`Question → LLM generates several reformulated queries → Retrieve for each → Fuse → Re-rank`

- **Best when:** User questions are vague, ambiguous, or could be phrased many different ways.  
- **Weak when:** Adds latency (multiple retrieval rounds) and can over-broaden the candidate set if not fused/reranked carefully.  
- **One-liner:** Don't bet the whole answer on one way of asking the question.

### 5.4 Parent-Child (Hierarchical) RAG

`Index small "child" chunks for precision → expand to larger "parent" section for context`

- **Best when:** Documents have strong hierarchical structure (manuals, policies, contracts, specs) where a precise fact needs surrounding context.  
- **Weak when:** Source documents are short or flat — the parent/child split adds indexing complexity for no benefit.  
- **One-liner:** Resolves the chunking trade-off ("small \= precision, large \= context") by doing both, at different stages.

### 5.5 Graph RAG

`Documents → Extract entities & relationships → Knowledge graph → Graph traversal (+ vector search) → LLM`

- **Best when:** Questions require reasoning across relationships ("who approved this change, under which policy, after which incident").  
- **Weak when:** Source content is unstructured prose with few meaningful entity relationships — heavier upfront engineering investment.  
- **One-liner:** Vector search finds similar text; graph traversal finds connected facts.

**Quick Recall:** Graph RAG \= Following a chain of connections, not just matching keywords

### 5.6 Agentic RAG

`Question → LLM plans → retrieves → evaluates sufficiency → retrieves again if needed → reasons → answers`

- **Best when:** Complex, multi-step, or ambiguous questions where a single retrieval pass is unlikely to be sufficient.  
- **Weak when:** Simple, well-defined Q\&A — the iterative loop adds cost/latency for no quality gain and is harder to govern.  
- **One-liner:** Retrieval becomes a decision the model makes, not a step the pipeline executes.

**Quick Recall:** Agentic RAG \= A research analyst who keeps digging until satisfied

### 5.7 Adaptive RAG (the emerging best practice)

`Question → Classifier estimates complexity → routes to the cheapest pattern that can answer it correctly`

- **Best when:** You're running RAG at scale across a wide variety of question types and can't afford to run the most expensive pattern on every query.  
- **Weak when:** You have a narrow, homogeneous question set — the routing layer becomes pure overhead.  
- **One-liner:** Don't use a research analyst to answer "what's our return policy" — route by need, not by maximal capability.

### Pattern selection cheat sheet

- Simple, narrow, FAQ-like? → Naive or Hybrid RAG  
- Mixed natural-language \+ exact terms? → Hybrid RAG (default for enterprise)  
- Vague / many phrasings of same intent? → Multi-Query RAG  
- Facts need surrounding context? → Parent-Child RAG  
- Questions about relationships? → Graph RAG  
- Multi-step, research-like questions? → Agentic RAG  
- Wide variety of all the above at scale? → Adaptive RAG (router in front of the rest)

---

## 6\. Security Architecture & Failure Analysis

### Enterprise Security Models

The stage-by-stage section named that security matters; here are the actual models architects choose between.

**RBAC — Role-Based Access Control**

- *What:* Access is granted based on a user's role (e.g., "Finance Analyst," "HR Manager").  
- *Best when:* Roles map cleanly to access needs and don't change often — most internal enterprise tools.  
- *Weak when:* Access depends on context (project, region, time, sensitivity) that role alone can't capture — "role explosion."  
- *One-liner:* Simple and auditable — until your roles start multiplying to cover edge cases.

**ABAC — Attribute-Based Access Control**

- *What:* Access decisions evaluate attributes of the user, resource, and context together (e.g., user.department \== document.department AND clearance ≥ classification AND region matches).  
- *Best when:* Access genuinely depends on multiple, combinable factors — regulated industries, multinational orgs, fine-grained sensitivity tiers.  
- *Weak when:* The added flexibility brings real complexity — policies are harder to write, test, and audit than role lists.  
- *One-liner:* More expressive than RBAC, and proportionally harder to get right.

**Document-Level ACLs (Access Control Lists)**

- *What:* Each document (or chunk) carries an explicit list of who/what can access it — often inherited from the source system (SharePoint, Confluence).  
- *Best when:* Source systems already maintain authoritative, fine-grained permissions you can mirror into the RAG metadata layer.  
- *Weak when:* Source permissions change frequently — your index can drift out of sync (the most common real-world RAG security bug).  
- *One-liner:* Only as good as how fresh your sync with the source system is.

**Row-Level Security (RLS)**

- *What:* Enforced at the data-store layer — the database/search platform itself filters out rows/vectors/chunks the querying identity isn't allowed to see.  
- *Best when:* You want security enforced as close to the data as possible, so no application bug can accidentally leak it.  
- *Weak when:* Not all vector/search platforms support expressive RLS — you may be constrained by the platform.  
- *One-liner:* Push security down to the data layer whenever the platform allows it — much harder to bypass by accident.

**Tenant Isolation**

- *What:* Guarantees one customer's/business unit's data is never retrievable by another's — via separate indices, namespaces, encryption keys, or fully separate infrastructure.  
- *Best when:* You're running a shared RAG platform across multiple customers, business units, or regulatory domains.  
- *Weak when:* Stricter isolation (separate infra per tenant) costs more and complicates the "shared platform" economics that justified building it centrally.  
- *One-liner:* The line between "multi-tenant platform" and "data breach waiting to happen" is exactly how seriously you take this.

**Security Model Trade-off Table**

| Model | Simplicity | Flexibility | Drift / audit risk |
| :---- | :---- | :---- | :---- |
| RBAC | High | Low | Low — but "role explosion" creeps in over time |
| ABAC | Medium | High | Medium — expressive policies are harder to test and audit |
| Document-Level ACL | Medium | Medium | High — only as fresh as the sync with the source system |
| Row-Level Security (RLS) | High (once set up) | Medium | Low — enforced at the data layer, hard to bypass by accident |
| Tenant Isolation | Medium | N/A (orthogonal) | Low if designed in; high cost to retrofit |

**Section one-liner:** In a regulated enterprise, the security model isn't a feature of RAG — it's a prerequisite for being allowed to run RAG on real data at all.

### Failure Analysis — "Why Is the Answer Bad?"

Work backwards through the cascade — a bad answer is a pipeline-tracing exercise, not a model-quality complaint:

| Symptom | Most likely stage(s) | What to check |
| :---- | :---- | :---- |
| Confidently wrong answer, no relevant source available | Generation / Prompting | Was the model instructed to say "I don't know" when context is insufficient? |
| Right document exists but never retrieved | Chunking, Embedding, or Retrieval | Is the fact split across chunk boundaries? Does the embedding model fit this domain? Is it in the top-50? |
| Right chunk retrieved but not in final answer | Re-ranking or Context Assembly | Check logs — did it survive reranking? Did assembly dedupe/drop it? |
| Answer cites the wrong source among several correct ones | Context Assembly or Prompting | Check ordering, deduplication, citation instructions |
| User sees content they shouldn't / can't see content they should | Metadata, Identity, or Security Filtering | Audit metadata freshness vs. actual source-system permissions — the \#1 real-world RAG security bug |
| Answer reflects an outdated version of a document | Index Lifecycle / Data Freshness | Check change-detection and re-indexing latency against the source system's update cadence |
| Answers are technically correct but generic/unhelpful | Chunking (too large) or Context Assembly (too compressed) | Check chunk size and how much specific detail survives into final context |
| Latency too high | Retrieval breadth, Re-ranking, or Generation model size | Profile each stage independently — reranking and generation usually dominate |
| Inconsistent answers to the same question over time | Permissions drift, ANN approximation, or non-deterministic decoding | Did the index, permissions, or model settings change between runs? |
| Multi-hop / "how do these connect" questions answered poorly | Pattern mismatch | You may be running Naive/Hybrid RAG on questions that need Graph or Agentic RAG |

**One-liner:** A RAG failure is rarely "the AI is wrong" — it's almost always "a specific, identifiable stage lost information it was supposed to preserve."

---

## 7\. Evaluation & Observability

### Evaluation — How Do You Know If It's Actually Working?

Evaluation splits cleanly into two halves — did we find the right information, and did we say the right thing with it — because a system can fail at either independently.

**Retrieval Evaluation**

- *Recall@k:* Of all chunks that should have been retrieved, how many were in the top-k?  
- *Precision@k:* Of the chunks retrieved, how many were actually relevant?  
- *MRR / NDCG:* Do the most relevant chunks rank highest, not just appear somewhere?  
- *Test sets:* Curate question/expected-source pairs from real usage logs and SMEs — the single highest-value evaluation investment, and the one most teams skip.

**Answer Evaluation**

- *Faithfulness / Groundedness:* Is every claim actually supported by the retrieved context, or did the model add things that "sound right"? (the measurable proxy for hallucination)  
- *Relevance:* Does the answer address the question asked — useful, not just true?  
- *Citation Accuracy:* Do citations point to sources that actually support each claim?  
- *Completeness:* Did the answer use all the relevant information available, or omit something important?

**How these get measured in practice**

- LLM-as-judge: a (typically stronger) model scores faithfulness, relevance, citation accuracy at scale — fast and cheap, needs periodic calibration against human judgment  
- Human review: slower, more expensive — the ground truth your LLM-judge calibration depends on, usually sampled  
- Golden datasets / regression suites: a fixed set of question–expected-answer(-and-source) pairs run on every meaningful pipeline change

**One-liner:** If you can't measure recall, faithfulness, and citation accuracy separately, you can't tell whether a bad answer is a retrieval problem or a generation problem.

### Observability — What Should Be Watched in Production?

Evaluation tells you if the system is good; observability tells you what's happening right now and when something changes.

| Signal | Why it matters | What a problem looks like |
| :---- | :---- | :---- |
| Query latency (end-to-end and per-stage) | Tells you where time is going — usually reranking and generation dominate | A specific stage's latency creeping up as corpus or volume grows |
| Retrieval recall / hit-rate | The earliest warning that retrieval quality is degrading | Recall slowly dropping as new content is added without re-tuning |
| Hallucination / faithfulness rate | The most direct signal of whether users can trust the answers | Faithfulness scores dropping after a model, prompt, or assembly change |
| Cost & token usage (by stage) | Cost is perpetual and per-query; small inefficiencies compound fast | A "small" prompt-template change quietly doubling average context size |
| Citation accuracy / coverage | Whether users can actually verify what the system tells them | Citations present but pointing to wrong or generic sources |
| Access-denial / security-filter events | Your earliest signal of a permissions-sync problem | A spike in "couldn't retrieve due to permissions" — often a stale index |
| Query patterns / failure clustering | Shows you where users are struggling, in aggregate | A cluster of similar questions all getting poor answers — usually one fixable upstream issue |

**One-liner:** If evaluation is the report card, observability is the dashboard — you need both, because a system can pass its evaluation suite on Monday and silently degrade by Friday as data, usage, or models drift.

---

## 8\. Cost Model & Tooling Map

### Cost & Performance Model

| Component | Primary cost driver | Primary latency driver | Scaling consideration |
| :---- | :---- | :---- | :---- |
| Ingestion / Chunking | Volume & complexity of source docs (one-time \+ incremental) | Not user-facing (offline) | Re-ingestion cost when formats/strategy change |
| Embedding | \# chunks × calls (ingestion) \+ every query (runtime) | Per-query embedding latency | Changing embedding models means re-embedding the whole corpus |
| Vector DB / Index | Storage (vectors \+ metadata) \+ search compute | Index size & ANN parameters | Costs grow with corpus size and availability/replication needs |
| Retrieval (hybrid) | Query volume; running two search systems | Network \+ search time across both indices | Hybrid roughly doubles search-side infrastructure |
| Re-ranking | Query volume × candidate-set size (pairwise scoring) | Often the single biggest per-query latency contributor | Tune candidate-set size (top-50 vs. top-20) for quality/speed balance |
| LLM Generation | Input tokens (prompt \+ context) \+ output tokens | Model size, output length, provider | The largest recurring lever — context size has direct, compounding cost impact |
| Guardrails / Post-processing | \# and complexity of checks; possible secondary LLM calls | Adds a tail to every response | Each extra safety pass trades latency/cost against risk reduction |

**Rules of thumb**

- Context size is your biggest controllable lever — every retrieved token is paid for on every query, forever  
- Re-ranking and generation dominate runtime latency — profile these first  
- Ingestion costs are one-time-ish; runtime costs are perpetual — over-invest in chunking/embedding quality up front  
- Repeated or near-duplicate questions don't need a fresh retrieval pass every time — semantic/response caching can skip retrieval (and its cost) entirely for queries the system has effectively already answered; it sits architecturally between Query Processing and Retrieval, and is one of the most direct levers on the "perpetual, per-query" cost problem this section describes

**One-liner:** In RAG you don't pay for "AI" — you pay per token, per query, per stage, forever. Architecture decisions are cost decisions.

### Component → Technology Mapping

The goal isn't to memorize today's leaderboard — tools rotate every few months. It's to know the category, what to evaluate it on, and a few names, so you can map any new tool back to a category you already understand. (Reflects the landscape as of mid-2026 — verify against current docs before committing.)

| Category | What it does | What to evaluate it on | Examples |
| :---- | :---- | :---- | :---- |
| Embedding models | Text → vector | Retrieval quality on your domain/language, dimensions vs. cost, multilingual support, hosted vs. self-hosted | OpenAI embeddings, Amazon Titan, Cohere Embed, BGE, E5, Voyage |
| Vector databases / search platforms | Store \+ search vectors, text, metadata | Hybrid support, metadata-filter expressiveness, scaling/ops model, managed vs. self-hosted fit | Pinecone, Qdrant, Weaviate, OpenSearch, Azure AI Search, pgvector, Neo4j |
| Rerankers | Score candidate relevance jointly | Accuracy lift vs. latency cost, pricing per query, ease of swapping in | Cohere Rerank, BGE-reranker, open cross-encoder models |
| Orchestration / RAG frameworks | Wire ingestion → retrieval → generation together | Abstraction vs. lock-in, debuggability, community momentum, fit with stack | LangChain, LlamaIndex, Haystack |
| Generation models (LLMs) | Produce the grounded answer | Reasoning quality over long context, cost/latency at volume, tool-calling, context window | Claude, GPT-4-class models, open-weight options served internally |
| AI Gateway / model-routing layer | Auth, routing, rate-limiting, cost-tracking across providers | Multi-provider support, observability hooks, governance fit with identity stack | Cloud-native gateways (Bedrock, Azure AI Foundry), purpose-built/custom gateways |
| Evaluation tooling | Score retrieval and answer quality | Support for faithfulness/recall@k/citation metrics, golden-set building, tracing integration | LangSmith, Arize Phoenix, similar platforms |
| Observability / tracing | Monitor latency, cost, recall, hallucination rate in production | Per-stage tracing depth, alerting, integration with ops stack | LangSmith, Arize Phoenix, dedicated RAG-observability platforms |
| Fully-managed RAG-as-a-service | Bundles ingestion, embedding, indexing, retrieval, reranking, hallucination mitigation behind one API | Control traded for convenience, exit/portability, fit for teams without ML engineering capacity | Platforms such as Vectara and similar managed offerings |

**One-liner:** Pick the category first based on your architecture decisions; pick the product last, and re-evaluate it periodically — this layer of the stack changes faster than any other.

---

## 9\. Enterprise Reference Architecture — The Platform View

`Users → Applications → AI Gateway (routing · auth · rate-limiting · cost-tracking · model abstraction) → Shared RAG Service (reusable ingestion, retrieval, reranking, context-assembly APIs) → Vector Platform (shared indices, multi-tenant metadata/ACL model) → Models (Embedding · Reranker · LLM) → Observability & Governance (logging · evaluation · guardrails · compliance — wraps the entire stack)`

**Key platform-level questions**

- Central AI team vs. domain teams: who owns shared retrieval/ingestion infrastructure (central, for consistency/reuse) vs. who owns domain knowledge, tuning, content quality (domain teams, for relevance/accountability)?  
- Shared RAG services: a reusable internal platform avoids every team reinventing chunking, embedding, and security filtering — but demands genuinely generic, well-governed APIs  
- AI Gateway: the single control point for routing, auth, rate limits, and cost-tracking — without it, cost and access sprawl across every application  
- Agent platforms: when RAG becomes one tool an agent calls alongside others, not the whole application  
- Reference architecture: a standardized pattern so every new RAG use case starts from "plug into the platform," not "design a pipeline from scratch"

**One-liner:** A single RAG pipeline is a project; a governed, observable, cost-aware shared platform in front of it is what makes RAG an enterprise capability rather than a pile of one-off PoCs.

---

## 10\. Quick-Recall Companion — Mental Models & One-Liners

*Kept here as a consolidated review tool — the individual analogies and quotes have already been woven in throughout this edition.*

### Architect Mental Models

| Concept | Analogy |
| :---- | :---- |
| Token | Customer ID |
| Embedding | Customer profile |
| Vector DB | Semantic search engine |
| Chunk | Book chapter |
| Context Window | RAM |
| Retriever | Long-list candidate search |
| Reranker | Interview panel |
| LLM | Final decision maker |
| Naive RAG | Asking one person one question, once |
| Agentic RAG | A research analyst who keeps digging until satisfied |
| Graph RAG | Following a chain of connections, not just matching keywords |
| The whole pipeline | A lossy chain — each link can only preserve or destroy what it received |

### Interview-Ready One-Liners

"RAG is not Vector DB \+ LLM — it's ingestion \+ indexing \+ retrieval \+ ranking \+ context assembly \+ prompt construction \+ generation \+ governance."

"Small chunks \= precision, large chunks \= context — parent-child RAG exists specifically to resolve that trade-off."

"Over-retrieve first, refine later — the retriever optimizes recall, the reranker optimizes precision."

"Retriever finds candidates, reranker judges candidates — long-list vs. interview panel."

"Cross-encoders compare jointly, embedding models compare separately — that's why cross-encoders win on relevance and lose on speed."

"Quality lost early in the pipeline cannot be recovered later — a perfect LLM can't fix a chunk that was never retrieved."

"Vector search finds similar text; graph traversal finds connected facts — pick based on what your questions actually need."

"RAG answers 'what do our documents say'; fine-tuning changes 'how the model behaves'; agents decide 'what to do next'; knowledge graphs answer 'how things relate.'"

"If you can't measure recall, faithfulness, and citation accuracy separately, you can't tell whether a bad answer is a retrieval problem or a generation problem."

"In RAG you don't pay for AI — you pay per token, per query, per stage, forever."

"Governance is what separates a PoC from a product; security architecture is what gets you permission to run it on real data at all."

"RAG indexes knowledge; it does not own knowledge — the index is a derived, lossy copy of the source of truth."

---

## 11\. One-Page Summary

RAG prepares enterprise knowledge offline (load → clean → chunk → enrich with metadata → embed → index), then at runtime takes a question through identity checks, query processing, security-filtered hybrid search, reranking, context assembly, prompt construction, and generation — wrapped in guardrails, citations, and logging. Quality flows through this chain and can only be preserved or lost at each stage, never recovered downstream, which is why chunking and metadata decisions matter as much as model choice. Before building any of it, ask whether RAG is even the right tool versus fine-tuning, long-context, agents, or knowledge graphs — and if it is, pick the pattern (naive, hybrid, multi-query, parent-child, graph, agentic, or adaptive) that matches your actual question types rather than defaulting to the most complex one available. Wrap the system in a real security model (RBAC/ABAC/ACLs/row-level security/tenant isolation), measure it on both retrieval and answer quality (recall, faithfulness, citation accuracy), watch it continuously in production (latency, cost, hallucination rate, access-denial events), and price it like what it is — a perpetual, per-token, per-query cost. Do all of that, and you've moved from "I can explain RAG" to "I can architect, secure, run, and evolve a RAG platform" — which is the actual job.

---

## 12\. Solutions Architect Playbook — Requirements Gathering

The earlier sections (1–11) answer "how do you architect, secure, run, and evolve a RAG platform." This section answers a question that comes *before* that — the one an SA has to resolve with stakeholders before a single architecture decision can be made well.

### 12.1 Requirements-Gathering & Sizing — The "Week One" Discovery Checklist

This is arguably the most distinctly SA skill in the whole lifecycle — the one this handbook assumes is already done by the time the pattern-selection cheat sheet (Section 5\) comes into play. Before any architecture decision can be made well, an SA needs answers to:

- **Corpus questions** (feeds the ingestion trade-off): How many documents, in what formats, how often do they change, who owns updates, how messy are the source systems — templated and clean, or scanned/unstructured and in need of OCR?  
- **Query questions** (the actual input to the Section 5 pattern-selection cheat sheet): What does a representative sample of real user questions look like? Vague or precise? Single-hop or multi-hop? Mostly natural language, or exact-term-heavy?  
- **Access-control questions** (determines whether the Section 6 RBAC/ABAC/ACL choice is even feasible): What permission model exists today, how often does it change, who is the system of record? Getting this wrong at discovery is how the "\#1 real-world RAG security bug" gets baked into a design before a single line of code is written.  
- **A back-of-envelope sizing heuristic:** translate "X documents, Y average length, Z queries/day" into a rough estimated token spend using the cost model (Section 8\) — embedding cost scales with corpus size at ingestion and query volume at runtime; generation cost scales with context size × queries, perpetually. An SA typically needs to produce a number like this within days of a first conversation, long before any architecture is finalized.

**One-liner for the whole section:** Sections 1–11 teach you how to architect a RAG system once the problem is well-framed. This section is about framing the problem — the conversations, decisions, and heuristics that happen before the first pipeline diagram gets drawn.

*(Note: the broader SA lifecycle — build vs. buy, phased rollout, and portfolio strategy — has been relocated to Appendix A below. It belongs conceptually with this Requirements-Gathering material, but is staged separately because it's platform-strategy content destined for a future "Enterprise AI Platform" handbook rather than RAG-pipeline architecture proper.)*

---

## Appendix A. Staged for Future Handbook — Build vs. Buy, Phased Rollout, Portfolio View

*This material is unchanged from the original Section 12.1–12.3. It has been moved here — not removed — because it's Solutions-Architect / platform-strategy content rather than RAG-pipeline architecture. It's staged for inclusion in a future "Enterprise AI Platform" handbook; until then, it remains here for reference and continuity.*

### A.1 Build vs. Buy

*Extends the "Fully-managed RAG-as-a-service" row in the Tooling Map (Section 8\)*

- Lean toward **buying** when time-to-value is the binding constraint, the team lacks dedicated ML engineering capacity, the use case is fairly standard (single corpus, common document types, FAQ-like questions), and the data isn't so sensitive that it can't sit inside a third party's pipeline.  
- Lean toward **building** when the corpus or access-control model is unusual enough that off-the-shelf won't fit (multi-tenant ACL inheritance, regulated data residency, exotic source systems), the organization is making a multi-year platform bet, or query volume is high enough that per-query managed pricing becomes the dominant cost driver over time.  
- The usual real answer is **hybrid**: buy the commodity layers (embedding, reranking, vector DB — the rows that rotate fastest and differentiate least), build the differentiated layers (ingestion connectors into proprietary systems, security filtering, domain-specific prompt and context-assembly logic).

**The artifact an SA actually owes the stakeholder:** It isn't a recommendation — it's an exit-cost analysis: what would it take, in time and engineering effort, to migrate off this vendor in 18 months if it doesn't work out? That question is what every "build vs. buy" conversation eventually turns into.

### A.2 Phased Rollout — PoC → Pilot → Platform

| Stage | Duration | Scope | What "good" looks like | Sections in play |
| :---- | :---- | :---- | :---- | :---- |
| PoC | 4–6 weeks | Single use case, narrow corpus, naive or hybrid RAG | Retrieval quality clears a bar on a small golden set — not production security or observability | Patterns (Section 5), Evaluation (Section 7\) |
| Pilot | \~1 quarter | Real users, real (if limited) access controls, first eval/observability pass | Cost-per-query model holds up against real usage; access-control model survives an actual audit | Security (Section 6), Eval & Obs (Section 7), Cost (Section 8\) |
| Platform | Ongoing | Shared services, AI Gateway, multi-tenant vector platform, governance | "Central vs. domain ownership" resolved in practice, not just diagrammed | Reference Architecture (Section 9\) |

The point of staging this explicitly — and naming it in a proposal — is that it lets an SA scope and price each phase separately, set the right expectations about what each phase will not yet deliver (e.g., "the PoC will prove retrieval quality; it will not yet prove security or cost at scale"), and create natural go/no-go checkpoints rather than one large, hard-to-estimate commitment.

### A.3 Portfolio View — Staging the Shared-Platform Investment

Section 9 describes what a shared RAG platform looks like once it exists. The harder, more SA-specific problem is how you get there:

- **Identify the second and third use case before building the first one's platform layer.** A shared-service investment is far easier to justify — and far less risky — when backed by two or three concrete future use cases, not a single pilot and a hopeful roadmap slide.  
- **Separate what generalizes from what doesn't.** Ingestion connectors and security/identity filtering tend to generalize well across business units; chunking strategy, prompt construction, and evaluation golden sets tend to stay domain-specific. That split is usually the actual line an SA draws when deciding what belongs in a shared platform versus what stays owned by domain teams.  
- **Use a rough break-even heuristic.** Shared infrastructure tends to pay for itself once an organization is supporting roughly three to four concurrent use cases. Below that, the platform-building overhead usually outweighs the reuse benefit, and a per-engagement build is the more defensible call.

---

## Appendix B. Changelog — What Changed in This Revision

*Per your instruction, nothing was removed — only reorganized, relocated, or added. Listed in the order the changes appear in the document.*

**Restructured**

- Section 3 split into 3.1 Runtime Pipeline / 3.2 The Cascade Principle / 3.3 Component Relationships, elevating the Cascade into its own named subsection as the interpretive lens for the rest of the handbook.  
- Section 12 narrowed to Requirements-Gathering only (renumbered as 12.1) with a note pointing to Appendix A for the relocated material.

**Relocated (not removed)**

- Original Sections 12.1–12.3 (Build vs. Buy, Phased Rollout PoC→Pilot→Platform, Portfolio View) moved verbatim to **Appendix A**, staged for a future "Enterprise AI Platform" handbook.

**Added — new architectural concepts**

- **Source of Truth** relationship (Section 3.1): RAG index framed as a derived, lossy copy of source systems — "RAG indexes knowledge; it does not own knowledge" — plus a corresponding Quick-Recall one-liner in Section 10\.  
- **Index Lifecycle & Data Freshness** (new Section 4.6a): change-detection → re-chunk → re-embed → re-index flow, refresh-strategy options (Batch / Incremental / Real-time), and a new failure-analysis row in Section 6 ("answer reflects an outdated version of a document").  
- **RAG \+ Fine-Tuning as complements** (Section 2): a short table and relationship statement reframing the two as combinable rather than purely competing options.  
- **Untrusted-content note** (Section 4.12): one-paragraph acknowledgment that retrieved content is data, not instructions — the pipeline-level seed of the prompt-injection topic, with the deeper attack-surface treatment flagged as belonging to a future security/governance treatment.  
- **Semantic caching** rule of thumb added to the Cost Model section (8) as a lever that sits between Query Processing and Retrieval.

**Added — trade-off / decision tables**

- Chunking Decision & Trade-off Table (Section 4.2): Fixed / Recursive / Semantic / Structure-aware / Parent-Child × Precision / Context / Complexity, plus practical starting-point guidance (chunk size, overlap) explicitly framed as defaults to validate, not fixed rules.  
- Retrieval Strategy Selection table (Section 4.10): consolidates guidance on keyword vs. vector vs. hybrid that was previously scattered across the stage and pattern sections, with a clear "Hybrid is the default" recommendation.  
- Index Refresh trade-off table (Section 4.6a): Batch / Incremental / Real-time × Freshness / Cost / Complexity.  
- Security Model trade-off table (Section 6): RBAC / ABAC / ACL / RLS / Tenant Isolation × Simplicity / Flexibility / Drift-and-audit risk.

**Added — explicit relationship one-liners** (woven into existing sections, in the Quick-Recall style already used throughout)

- Chunking ↔ Embedding (Section 4.2): "chunking determines what gets embedded; embedding determines what can be retrieved."  
- Metadata ↔ Security (Section 4.3 and 4.9): "security filtering depends on metadata — bad metadata is bad security."  
- Retrieval ↔ Re-ranking (Section 4.10): "retrieval optimizes recall, re-ranking optimizes precision."  
- Context Assembly ↔ Prompt Construction: already present in the original (Section 4.12) — left as-is and cross-referenced from the new Section 3.3 preview.

**Added — visual aid**

- Context Assembly funnel diagram (Section 4.12): Top-50 → Re-rank → Top-10 → Dedupe → Compress → Order → Final Context, visualizing the narrowing process that the prose already described.

**Front matter**

- Added a short "Revision note" near the top of the document explaining the nothing-removed / relocate-don't-delete approach, and pointing to this changelog.

**Round 2 — small precision fixes**

- Section 1's Quick Recall replaced with a RAG-specific contrast — "Without RAG: Question → LLM → Guess · With RAG: Question → Retrieve Facts → LLM → Answer" — so the section's call-out reinforces its own three problems (missing knowledge, stale knowledge, hallucination) instead of reusing an embedding-stage analogy that was better placed near 4.4 (where "Embedding \= Customer profile" already lives) and in the Section 10 mental-models table (where "Token \= Customer ID" remains).  
- Cascade diagram (3.2) updated from "→ Prompt → Answer" to "→ Prompt Construction → Answer" for naming consistency with Section 4.13 and the rest of the handbook.

