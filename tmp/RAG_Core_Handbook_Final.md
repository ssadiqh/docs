# RAG Core Handbook — Final

### Architecture · Patterns · Trade-offs · Security · Evaluation · Observability · Cost · Tooling

This is the complete version of the Core Handbook: everything from v4 (architecture, component trade-offs, relationships/cascade, RAG-vs-alternatives, failure analysis, cost model, platform view) plus the remaining gaps — architecture **patterns**, **security architecture**, **evaluation**, **observability**, and a fleshed-out **Component → Technology** map (categories \+ what to evaluate \+ current tools, researched as of mid-2026).

---

## 1\. Why RAG Exists

LLMs alone have three problems: **missing enterprise knowledge**, **stale knowledge**, and **hallucination**. RAG inserts a retrieval step between question and model — `Enterprise Documents → RAG → LLM` — so the model writes from the right facts instead of guessing.

---

## 2\. End-to-End Architecture (the spine)

ENTERPRISE SOURCES → INGESTION → METADATA ENRICHMENT → EMBEDDING → VECTOR DB / INDEX

                                                                          │

══════════════════════════════ RUNTIME ══════════════════════════════════╪═══════

                                                                          ↓

USER QUESTION → IDENTITY/AUTH → QUERY PROCESSING \+ EMBEDDING → SECURITY FILTER

        → RETRIEVAL (vector/keyword/hybrid) → RE-RANKING → CONTEXT ASSEMBLY

        → PROMPT CONSTRUCTION → LLM → GUARDRAILS/CITATIONS/LOGS → FINAL ANSWER

| Pipeline | Job | One-liner |
| :---- | :---- | :---- |
| Ingestion | Prepares knowledge | Garbage in, garbage out |
| Retrieval | Finds relevant knowledge | Optimizes recall |
| Context | Prepares knowledge for the LLM | Optimizes signal-to-noise |
| Generation | Produces the answer | Only as good as what it's handed |
| Governance | Controls safety, access, logging, evaluation | Separates a demo from a product |

---

## 3\. Relationships Between Components (the cascade)

Source Quality → Chunking → Embedding → Retrieval → Re-ranking → Context Assembly → Prompt → Answer

**The rule:** *quality lost early cannot be recovered later.* RAG is a lossy pipeline — each stage can only **preserve or destroy** what it received. A perfect LLM cannot answer from context that never contained the right chunk; a perfect retriever cannot find a chunk that chunking destroyed.

A second cross-cutting relationship: **Identity/Authorization and Metadata Filtering define the search space retrieval is allowed to operate in** — get them wrong and you don't get a slightly-worse answer, you get a wrong or unsafe one.

---

## 4\. RAG vs. Alternatives — Should You Even Use RAG?

| Approach | What it does | Best when | Weak when |
| :---- | :---- | :---- | :---- |
| **RAG** | Retrieves relevant external knowledge at query time, feeds it to the LLM as context | Knowledge changes often, must be traceable/citable, too large for a prompt or model weights | Latency tolerance is very tight, or "knowledge" is really a *skill* (style/format/reasoning), not a *fact* |
| **Fine-tuning** | Adjusts model weights on your data | Need consistent style, domain language, structured outputs, narrow specialized skill | Knowledge changes often (constant retraining), or you need traceability/citations |
| **Long-context** | Puts source material directly in the prompt, no retrieval | Small, stable corpora that fit in context | Large/changing corpora — cost & latency scale with every query |
| **Agents** | Plans, calls tools (RAG can be one), takes multi-step action | Task needs multi-step reasoning, API calls, combining sources/actions | Task is a single grounded Q\&A — agent adds needless latency/complexity |
| **Workflow / deterministic pipeline** | Fixed, code-defined sequence, LLM/RAG as a component | Process is well-understood, repeatable, needs auditability | Task is open-ended / needs judgment that can't be pre-scripted |
| **Plain enterprise search** | Keyword/metadata search returns documents to a human | User wants to find and read a document themselves | User wants a synthesized conversational answer across many documents |
| **Knowledge graphs** | Entities \+ explicit relationships; supports multi-hop queries | Questions are about *how things relate* | Knowledge is mostly unstructured prose where relationships aren't the point |

**Decision shortcut:**

Up-to-date / large external knowledge that needs traceability?  → RAG (+ graph if relationships matter)

Need a different skill/style/format from the model itself?      → Fine-tuning

Small, stable corpus?                                            → Long context, skip retrieval

Multi-step reasoning / tool use?                                 → Agent (RAG as one of its tools)

**One-liner:** RAG answers "what do our documents say," fine-tuning changes "how the model behaves," agents decide "what to do next," knowledge graphs answer "how things relate." Usually combined, rarely substitutes.

---

## 5\. RAG Architecture Patterns — From Naive to Adaptive

Components explain the *parts*. Patterns explain how architects *assemble* those parts to fit a problem. This is now one of the most common interview and design-review topics.

### 5.1 Naive (Basic) RAG

`Question → Embed → Vector Search → Top-K → Prompt → LLM`

- **What it is:** The textbook pipeline — single embedding-based retrieval, no reranking or query rewriting.  
- **Best when:** Simple FAQ-style corpora, prototypes, low-stakes internal tools.  
- **Weak when:** Questions are vague, exact-term-heavy, or multi-hop — naive RAG misses all three.  
- **One-liner:** The "hello world" of RAG — a fine starting point, a poor production end-state.

### 5.2 Hybrid RAG

`Question → [Vector Search + Keyword Search] → Fusion → Re-rank → Prompt → LLM`

- **What it is:** Combines dense (semantic/vector) and sparse (keyword/BM25) retrieval, fused and reranked.  
- **Best when:** Enterprise corpora with a mix of natural language and exact terms (codes, IDs, names, acronyms) — i.e., almost all real enterprise content.  
- **Weak when:** Adds infrastructure cost/complexity that small, narrow-domain systems may not need.  
- **One-liner:** The 2026 production *baseline* — most enterprises should start here, not with naive RAG.

### 5.3 Multi-Query RAG

`Question → LLM generates several reformulated queries → Retrieve for each → Fuse results → Re-rank`

- **What it is:** Expands a single fragile query into several variants (paraphrases, sub-questions) and merges retrieval across all of them.  
- **Best when:** User questions are vague, ambiguous, or could be phrased many different ways.  
- **Weak when:** Adds latency (multiple retrieval rounds) and can over-broaden the candidate set if not fused/reranked carefully.  
- **One-liner:** Don't bet the whole answer on one way of asking the question.

### 5.4 Parent-Child (Hierarchical) RAG

`Index small "child" chunks for precision → on retrieval, expand to the larger "parent" section for context`

- **What it is:** Stores small, focused child chunks (\~100–500 tokens) for precise matching, but when one is retrieved, pulls in its larger parent section (\~500–2000 tokens) to give the LLM full surrounding context.  
- **Best when:** Documents have strong hierarchical structure (manuals, policies, contracts, specs) where a precise fact needs surrounding context to be understood correctly.  
- **Weak when:** Source documents are short or flat — the parent/child split adds indexing complexity for no benefit.  
- **One-liner:** Directly resolves the chunking trade-off ("small \= precision, large \= context") by doing *both*, at different stages.

### 5.5 Graph RAG

`Documents → Extract entities & relationships → Knowledge graph → Query → Graph traversal (+ vector search) → LLM`

- **What it is:** Instead of (or alongside) embedding chunks, extracts entities and their relationships into a knowledge graph, then retrieves via graph traversal — strong on multi-hop, "how do these things connect" questions.  
- **Best when:** Questions require reasoning across relationships ("who approved this change, under which policy, after which incident") rather than "what does this paragraph say."  
- **Weak when:** Source content is unstructured prose with few meaningful entity relationships — graph extraction becomes overhead without payoff. Also a heavier upfront engineering investment (entity/relation extraction pipeline).  
- **One-liner:** Vector search finds *similar text*; graph traversal finds *connected facts* — pick based on which one your questions actually need.

### 5.6 Agentic RAG

`Question → LLM plans → retrieves → evaluates sufficiency → retrieves again if needed → reasons → answers`

- **What it is:** Embeds an LLM *inside* the retrieval loop — it decides what to search, evaluates whether what came back is enough, and iterates (more retrieval, different retriever, different query) before answering, like a research analyst rather than a fixed pipeline.  
- **Best when:** Complex, multi-step, or ambiguous questions where a single retrieval pass is unlikely to be sufficient, and the cost/latency of iteration is acceptable.  
- **Weak when:** Simple, well-defined Q\&A — the iterative loop adds cost and latency for no quality gain, and makes behavior harder to predict/audit.  
- **One-liner:** Retrieval becomes a *decision the model makes*, not a *step the pipeline executes* — powerful, but harder to govern and cost-control.

### 5.7 Adaptive RAG (the emerging best practice)

`Question → Classifier estimates complexity → routes to the cheapest pattern that can answer it correctly`

- **What it is:** A query-classification layer in front of the pipeline that routes simple questions to naive/hybrid RAG and routes complex, multi-hop, or ambiguous ones to graph or agentic RAG — matching architectural sophistication to actual need.  
- **Best when:** You're running RAG at scale across a wide variety of question types and can't afford to run the most expensive pattern on every query.  
- **Weak when:** You have a narrow, homogeneous question set — the routing layer becomes pure overhead.  
- **One-liner:** Don't use a research analyst to answer "what's our return policy" — route by need, not by maximal capability.

**Pattern selection cheat sheet:**

Simple, narrow, FAQ-like?              → Naive or Hybrid RAG

Mixed natural-language \+ exact terms?  → Hybrid RAG (default for enterprise)

Vague / many phrasings of same intent? → Multi-Query RAG

Facts need surrounding context?        → Parent-Child RAG

Questions about relationships?         → Graph RAG

Multi-step, research-like questions?   → Agentic RAG

Wide variety of all the above at scale?→ Adaptive RAG (router in front of the rest)

---

## 6\. Stage-by-Stage: Purpose, Options, Decisions, Trade-offs, Failure Signatures

*(Each stage: Purpose → Options → How to Decide → Trade-off → Failure Signature → One-liner. "Failure Signature" \= what a problem here looks like downstream — your troubleshooting hook, expanded in §8.)*

### 6.1 Document Loading / Ingestion

Pulls raw content (SharePoint, Confluence, PDFs, DBs, APIs, web, tickets) and normalizes it. **Decide on:** how messy your sources are — templated sources need light processing, scans/slides need OCR/structure-aware extraction. **Trade-off:** source breadth vs. parsing fidelity. **Failure looks like:** garbled/truncated answers that *look* like a retrieval bug but are an ingestion bug. **One-liner:** sets the ceiling for everything downstream.

### 6.2 Chunking

Splits documents into retrievable units. **Options & decision:**

Need simplicity/speed?                     → Fixed-size

Need a balanced default?                   → Recursive

Need max precision on prose?                → Semantic

Source is strongly structured?              → Structure-aware

Facts need surrounding context preserved?   → Parent-Child pattern (§5.4)

**Trade-off:** small \= precision, large \= context. **Failure looks like:** "almost right" retrievals — topically related but missing the specific fact, or the fact split across chunks. **One-liner:** the single highest-leverage tuning knob.

### 6.3 Metadata Enrichment

Attaches owner, department, country, classification, source, type, date/version — the basis of both relevance and access control. **Decide:** let security/compliance set the *minimum* schema; let search-quality needs add the rest. **Trade-off:** richer metadata \= finer security/relevance, but more complexity and a permissions-sync problem. **Failure looks like:** over-exposure *or* false "I don't have that" responses. **One-liner:** where "search" becomes "governed enterprise search."

### 6.4 Embedding Model

Converts chunks/queries into vectors. **Decide:** match to dominant languages and domain; weigh hosted-API convenience vs. self-hosted cost/control at your query volume. **Trade-off:** higher dimensions often retrieve better but cost more to compute/store/search. **Failure looks like:** confidently-wrong "semantically similar but factually unrelated" results. **One-liner:** defines the "language" your retrieval thinks in.

### 6.5 Vector Database / Search Platform

Stores text \+ vectors \+ metadata; serves similarity search. **Decide:** existing stack (Postgres → pgvector; OpenSearch shop → OpenSearch) vs. need for graph+vector (Neo4j) vs. fully managed scale (Pinecone/Weaviate Cloud/Azure AI Search). **Trade-off:** managed \= less ops, more cost/lock-in; self-hosted \= more control, more ops ownership. **Failure looks like:** latency or relevance degradation as the corpus grows (usually an indexing/tuning issue). **One-liner:** pick for your stack and ops appetite — most are "good enough" on raw retrieval quality.

### 6.6 ANN Indexing

Builds fast approximate-search structures (HNSW, IVF). **Trade-off:** small accuracy loss for large speed gain. **Failure looks like:** inconsistent results on near-identical queries, or slowdowns past tuning assumptions. **One-liner:** "approximately right, very fast" — explicitly chosen.

### 6.7 Identity & Access Context

Establishes who's asking before retrieval happens — the entry point to the security architecture in §7. **Trade-off:** query-time enforcement costs more per query than ingest-time-only, but is the only correct way to handle permissions that change post-indexing. **Failure looks like:** the same question getting different answers depending on *when* it's asked. **One-liner:** skip it and a helpful assistant becomes a data-leak incident.

### 6.8 Query Processing & Embedding

Cleans, interprets intent, optionally rewrites/expands, then embeds the query (this is where Multi-Query RAG, §5.3, plugs in). **Trade-off:** rewriting improves recall on vague questions but adds latency and intent-drift risk. **Failure looks like:** the system answering a *related but different* question. **One-liner:** a well-formed query is half the retrieval battle.

### 6.9 Metadata / Security Filtering

Narrows the search space to authorized documents — see §7 for the underlying models (RBAC/ABAC/ACL/row-level/tenant isolation). **Decide:** pre-filter (before vector search — more secure/efficient, needs platform support) vs. post-filter (simpler, riskier). **Failure looks like:** "found something relevant but can't show it" loops, or silent leakage only caught in audit. **One-liner:** security and relevance working as one mechanism.

### 6.10 Retrieval (Vector / Keyword / Hybrid)

Casts a wide net — typically top 50–100 — via vector similarity and/or keyword matching (this *is* the choice between Naive and Hybrid RAG, §5.1–5.2).

Mostly natural-language, paraphrased questions? → Vector search

Mostly exact terms (codes, IDs, acronyms)?       → Keyword search

Mixed / enterprise-general?                       → Hybrid (default)

**Failure looks like:** the right document exists, is correctly chunked and tagged, but never appears in the candidate list — pure recall failure. **One-liner:** over-retrieve first, refine later — this stage's job is recall.

### 6.11 Re-ranking

Scores candidates jointly (cross-encoder: question \+ chunk together) and narrows to the best few. **Decide:** cross-encoder for cheap, consistent reranking of large sets; LLM-as-reranker only for small sets needing nuanced judgment, accepting the latency/cost. **Trade-off:** cross-encoders beat embedding-similarity ranking on accuracy (joint attention) but are too slow/costly to run over a whole corpus. **Failure looks like:** the right chunk *was* retrieved (visible in logs) but didn't survive ranking — easy to misdiagnose as a retrieval bug. **One-liner:** retriever finds candidates, reranker judges them — long-list vs. interview panel.

### 6.12 Context Assembly

Dedupes, orders, compresses top results into the token-budget-respecting final context. **Trade-off:** more context \= more completeness but higher cost/"lost in the middle" risk; tighter \= cheaper/sharper but can omit detail. **Failure looks like:** answer cites the wrong source among several correct ones, or blends sources incorrectly. **One-liner:** where retrieval quality is preserved — or quietly diluted — right before the model sees it.

### 6.13 Prompt Construction

Packages instructions, question, context, citation/safety rules. **Trade-off:** more explicit instruction \= more consistency, but consumes context budget. **Failure looks like:** correct context assembled, but the answer doesn't cite it / mis-cites it / ignores instructions. **One-liner:** the contract between your retrieval pipeline and the model.

### 6.14 Generation (LLM)

Produces the grounded answer. **Trade-off:** bigger models reason better over context but cost more, respond slower, and cannot fix weak retrieval. **Failure looks like:** fluent, confident, *unsupported* answers despite good retrieval — a prompting/model-choice issue, not a retrieval issue. **One-liner:** the final decision maker, not the whole process.

### 6.15 Post-Processing / Guardrails

Adds citations, formatting, safety/policy checks, logging. **Trade-off:** more checks \= less risk, more latency, occasional over-blocking. **Failure looks like:** good answers getting blocked or stripped — the "too safe to be useful" failure mode. **One-liner:** the seatbelt — invisible when things go right.

---

## 7\. Enterprise Security Architecture

Section 6.7/6.9 named *that* security matters; this section names the actual models architects choose between.

### 7.1 RBAC (Role-Based Access Control)

- **What:** Access is granted based on a user's *role* (e.g., "Finance Analyst," "HR Manager").  
- **Best when:** Roles map cleanly to access needs and don't change often — most internal enterprise tools.  
- **Weak when:** Access depends on *context* (project, region, time, data sensitivity) that role alone can't capture — you end up creating a role for every combination ("role explosion").  
- **One-liner:** Simple and auditable — until your roles start multiplying to cover edge cases.

### 7.2 ABAC (Attribute-Based Access Control)

- **What:** Access decisions evaluate *attributes* of the user, the resource, and the context together (e.g., "user.department \== document.department AND user.clearance \>= document.classification AND user.region \== document.region").  
- **Best when:** Access genuinely depends on multiple, combinable factors — regulated industries, multinational orgs, fine-grained data-sensitivity tiers.  
- **Weak when:** The added flexibility brings real complexity — policies are harder to write, test, and audit than role lists.  
- **One-liner:** More expressive than RBAC, and proportionally harder to get right.

### 7.3 Document-Level ACLs (Access Control Lists)

- **What:** Each document (or chunk) carries an explicit list of who/what can access it — often inherited directly from the source system (SharePoint, Confluence permissions, etc.).  
- **Best when:** Source systems already maintain authoritative, fine-grained permissions you can mirror into the RAG metadata layer.  
- **Weak when:** Source permissions change frequently — your index can drift out of sync with reality unless you actively re-sync (this is the most common real-world RAG security bug).  
- **One-liner:** Only as good as how fresh your sync with the source system is.

### 7.4 Row-Level Security (RLS)

- **What:** Enforced at the data-store layer — the database/search platform itself filters out rows (or vectors/chunks) the querying identity isn't allowed to see, rather than relying on the application to filter after the fact.  
- **Best when:** You want security enforced as close to the data as possible, so no application bug can accidentally leak it.  
- **Weak when:** Not all vector/search platforms support expressive RLS — you may be constrained by what your chosen platform can enforce natively.  
- **One-liner:** Push security down to the data layer whenever the platform allows it — it's much harder to bypass by accident.

### 7.5 Tenant Isolation

- **What:** Guarantees that one customer's/business unit's data is never retrievable by another's — via separate indices, namespaces, encryption keys, or fully separate infrastructure, depending on how strict the isolation requirement is.  
- **Best when:** You're running a shared RAG platform across multiple customers, business units, or regulatory domains (the "Shared RAG Service" from §11).  
- **Weak when:** Stricter isolation (separate infra per tenant) costs more and complicates the "shared platform" economics that justified building it centrally in the first place.  
- **One-liner:** The line between "multi-tenant platform" and "data breach waiting to happen" is exactly how seriously you take this.

**One-liner for the whole section:** In a regulated enterprise, the security model isn't a feature *of* RAG — it's a *prerequisite* for being allowed to run RAG on real data at all.

---

## 8\. Failure Analysis — "Why Is the Answer Bad?"

Work backwards through the cascade in §3 — a bad answer is a pipeline-tracing exercise, not a model-quality complaint:

| Symptom | Most likely stage(s) | What to check |
| :---- | :---- | :---- |
| Confidently wrong answer, no relevant source available | Generation / Prompting | Was the model instructed to say "I don't know" when context is insufficient? |
| Right document exists but never retrieved | Chunking, Embedding, or Retrieval | Is the fact split across chunk boundaries? Does the embedding model fit this domain? Is it in the top-50? |
| Right chunk retrieved but not in final answer | Re-ranking or Context Assembly | Check logs — did it survive reranking? Did assembly dedupe/drop it? |
| Answer cites the wrong source among several correct ones | Context Assembly or Prompting | Check ordering, deduplication, citation instructions |
| User sees content they shouldn't / can't see content they should | Metadata, Identity, or Security Filtering (§7) | Audit metadata freshness vs. actual source-system permissions — this is the \#1 real-world RAG security bug |
| Answers are technically correct but generic/unhelpful | Chunking (too large) or Context Assembly (too compressed) | Check chunk size and how much specific detail survives into final context |
| Latency too high | Retrieval breadth, Re-ranking, or Generation model size | Profile each stage independently — reranking and generation usually dominate |
| Inconsistent answers to the same question over time | Permissions drift, ANN approximation, or non-deterministic decoding | Did the index, permissions, or model settings change between runs? |
| Multi-hop / "how do these connect" questions answered poorly | Pattern mismatch | You may be running Naive/Hybrid RAG on questions that need Graph or Agentic RAG (§5) |

**One-liner:** A RAG failure is rarely "the AI is wrong" — it's almost always "a specific, identifiable stage lost information it was supposed to preserve."

---

## 9\. Evaluation — How Do You Know If It's Actually Working?

Evaluation in RAG splits cleanly into two halves — **did we find the right information**, and **did we say the right thing with it** — because a system can fail at either independently.

### 9.1 Retrieval Evaluation (did we find the right information?)

- **Recall@k:** Of all the chunks that *should* have been retrieved, how many were in the top-k? (The retriever's core job — see §6.10.)  
- **Precision@k:** Of the chunks retrieved, how many were actually relevant? (More relevant to reranking quality, §6.11.)  
- **MRR / NDCG (ranking-quality metrics):** Do the *most* relevant chunks rank highest, not just appear somewhere in the list?  
- **How to build a test set:** Curate question/expected-source pairs from real usage logs and subject-matter experts — this is the single highest-value evaluation investment, and the one most teams skip.

### 9.2 Answer Evaluation (did we say the right thing with it?)

- **Faithfulness / Groundedness:** Is every claim in the answer actually supported by the retrieved context — or did the model add things that "sound right" but aren't there? (The direct, measurable proxy for hallucination.)  
- **Relevance:** Does the answer actually address the question asked — not just "is it true," but "is it useful here"?  
- **Citation Accuracy:** Do the citations in the answer point to the sources that actually support each claim — not just *a* source that happens to be in context?  
- **Completeness:** Did the answer use all the relevant information that was available in the context, or leave out something important that was right there?

### 9.3 How These Get Measured in Practice

- **LLM-as-judge:** Using a (typically stronger or differently-tuned) model to score faithfulness, relevance, and citation accuracy at scale — fast and cheap, but needs periodic calibration against human judgment.  
- **Human review:** Slower and more expensive, but the ground truth your LLM-judge calibration depends on — usually sampled, not exhaustive.  
- **Golden datasets / regression suites:** A fixed set of question–expected-answer(-and-source) pairs run on every meaningful pipeline change, so you can tell whether a "improvement" to chunking or prompting actually helped or quietly made things worse elsewhere.

**One-liner:** If you can't measure recall, faithfulness, and citation accuracy *separately*, you can't tell whether a bad answer is a retrieval problem or a generation problem — and you'll keep "fixing" the wrong stage.

---

## 10\. Observability — What Should Be Watched in Production?

Evaluation tells you *if* the system is good; observability tells you *what's happening right now* and *when something changes*. The core things a production RAG system should be tracking continuously:

| Signal | Why it matters | What a problem looks like |
| :---- | :---- | :---- |
| **Query latency** (end-to-end and per-stage) | Tells you where time is going — usually reranking and generation dominate | A specific stage's latency creeping up as the corpus or query volume grows |
| **Retrieval recall / hit-rate** (sampled against a golden set) | The earliest warning that retrieval quality is degrading | Recall slowly dropping as new content is added without re-tuning chunking/embeddings |
| **Hallucination / faithfulness rate** (sampled, via LLM-as-judge \+ human spot-checks) | The most direct signal of whether users can trust the answers | Faithfulness scores dropping after a model, prompt, or context-assembly change |
| **Cost & token usage** (broken down by stage — embedding, reranking, generation) | Cost is perpetual and per-query (§11) — small inefficiencies compound fast | A "small" prompt-template change quietly doubling average context size |
| **Citation accuracy / coverage** | Whether users can actually verify what the system tells them | Citations present but pointing to the wrong or generic sources |
| **Access-denial / security-filter events** | Your earliest signal of a permissions-sync problem (§7.3) | A spike in "couldn't retrieve due to permissions" — often means the index is stale relative to source-system ACLs |
| **Query patterns / failure clustering** | Shows you *where* users are struggling, in aggregate | A cluster of similar questions all getting poor answers — usually a single fixable upstream issue (§8) |

**One-liner:** If evaluation is the report card, observability is the dashboard — you need both, because a system can pass its evaluation suite on Monday and silently degrade by Friday as data, usage, or models drift.

---

## 11\. Cost & Performance Model

| Component | Primary cost driver | Primary latency driver | Scaling consideration |
| :---- | :---- | :---- | :---- |
| Ingestion / Chunking | Volume & complexity of source docs (one-time \+ incremental) | Not user-facing (offline) | Re-ingestion cost when formats/strategy change |
| Embedding | \# chunks × calls (ingestion) \+ every query (runtime) | Per-query embedding latency | Changing embedding models means re-embedding the whole corpus |
| Vector DB / Index | Storage (vectors \+ metadata) \+ search compute | Index size & ANN parameters | Costs grow with corpus size and availability/replication needs |
| Retrieval (hybrid) | Query volume; running two search systems | Network \+ search time across both indices | Hybrid roughly doubles search-side infrastructure |
| Re-ranking | Query volume × candidate-set size (pairwise scoring) | Often the single biggest per-query latency contributor | Tune candidate-set size (top-50 vs. top-20) for quality/speed balance |
| LLM Generation | Input tokens (prompt \+ context) \+ output tokens | Model size, output length, provider | The largest recurring lever — context size has direct, compounding cost impact |
| Guardrails / Post-processing | \# and complexity of checks; possible secondary LLM calls | Adds a tail to every response | Each extra safety pass trades latency/cost against risk reduction |

**Rules of thumb:**

- **Context size is your biggest controllable lever** — every retrieved token is paid for on every query, forever.  
- **Re-ranking and generation dominate runtime latency** — profile these first.  
- **Ingestion costs are one-time-ish; runtime costs are perpetual** — over-invest in chunking/embedding quality up front, because every runtime query pays for whatever quality was baked in.

**One-liner:** In RAG you don't pay for "AI" — you pay per token, per query, per stage, forever. Architecture decisions are cost decisions.

---

## 12\. Component → Technology Mapping

The goal here isn't to memorize today's leaderboard — tools rotate every few months. It's to know **the category, what to evaluate it on, and a few names** so you can map any new tool you encounter back to a category you already understand. (Current names below reflect the landscape as of mid-2026 — verify against current docs before committing, as this segment moves fast.)

| Category | What it does | What to evaluate it on | Examples in current use |
| :---- | :---- | :---- | :---- |
| **Embedding models** | Text → vector | Retrieval quality on *your* domain/language, dimensions vs. cost, multilingual support, hosted vs. self-hosted | OpenAI embeddings, Amazon Titan, Cohere Embed, BGE, E5, Voyage |
| **Vector databases / search platforms** | Store \+ search vectors, text, metadata | Hybrid (vector+keyword) support, metadata-filter expressiveness, scaling/ops model, managed vs. self-hosted fit with your stack | Pinecone, Qdrant, Weaviate, OpenSearch, Azure AI Search, pgvector, Neo4j (graph+vector) |
| **Rerankers** | Score candidate relevance jointly | Accuracy lift vs. latency cost, pricing per query, ease of swapping into your pipeline | Cohere Rerank, BGE-reranker, open cross-encoder models |
| **Orchestration / RAG frameworks** | Wire ingestion → retrieval → generation together | How much they abstract vs. lock you in, debuggability, community/maintenance momentum, fit with your chosen vector DB and models | LangChain, LlamaIndex, Haystack |
| **Generation models (LLMs)** | Produce the grounded answer | Reasoning quality over long context, cost/latency at your volume, tool-calling support, context-window size | Claude, GPT-4-class models, open-weight options served internally |
| **AI Gateway / model-routing layer** | Auth, routing, rate-limiting, cost-tracking across models/providers | Multi-provider support, observability hooks, governance/policy enforcement, how it fits your existing identity stack | Cloud-native gateways (e.g., Bedrock, Azure AI Foundry), purpose-built AI gateways, custom-built internal gateways |
| **Evaluation tooling** | Score retrieval and answer quality (§9) | Support for the specific metrics you need (faithfulness, recall@k, citation accuracy), ability to build golden/regression sets, integration with your tracing | LangSmith, Arize Phoenix, and similar LLM-evaluation platforms |
| **Observability / tracing** | Monitor latency, cost, recall, hallucination rate in production (§10) | Per-stage tracing depth (can it show you *where* in the pipeline time/quality was lost?), alerting, integration with your existing ops stack | LangSmith, Arize Phoenix, and dedicated RAG-observability platforms |
| **Fully-managed RAG-as-a-service** | Bundles ingestion, embedding, indexing, retrieval, reranking, hallucination mitigation behind one API | How much control you're trading for convenience; exit/portability if you outgrow it; fit for teams without dedicated ML engineering capacity | Platforms such as Vectara and similar managed offerings |

**How to use this table:** when you meet an unfamiliar product name, your first question should be "which row does this belong to," not "is this better than X." Once you know the category, you already know what questions to ask it.

**One-liner:** Pick the *category* first based on your architecture decisions (§5–§7); pick the *product* last, and re-evaluate it periodically — this layer of the stack changes faster than any other.

---

## 13\. Enterprise Reference Architecture (the platform view)

                         Users

                           ↓

                      Applications

                           ↓

                       AI Gateway

        (routing · auth · rate-limiting · cost-tracking · model abstraction)

                           ↓

                   Shared RAG Service

   (reusable ingestion, retrieval, reranking, context-assembly APIs)

                           ↓

                      Vector Platform

              (shared indices, multi-tenant metadata/ACL model — see §7.5)

                           ↓

                    Models (Embedding · Reranker · LLM)

                           ↓

              Observability & Governance

   (logging · evaluation · guardrails · compliance — wraps the entire stack)

**Key platform-level questions:**

- **Central AI team vs. domain teams:** who owns shared retrieval/ingestion infrastructure (central, for consistency/reuse) vs. who owns domain knowledge, tuning, content quality (domain teams, for relevance/accountability)?  
- **Shared RAG services:** a reusable internal platform avoids every team reinventing chunking, embedding, and security filtering — but demands genuinely generic, well-governed APIs.  
- **AI Gateway:** the single control point for routing, auth, rate limits, and cost-tracking — without it, cost and access sprawl across every application.  
- **Agent platforms:** when RAG becomes one *tool* an agent calls alongside others, not the whole application.  
- **Reference architecture:** a standardized pattern so every new RAG use case starts from "plug into the platform," not "design a pipeline from scratch."

**One-liner:** A single RAG pipeline is a project; a governed, observable, cost-aware shared platform in front of it is what makes RAG an *enterprise capability* rather than a pile of one-off PoCs.

---

## 14\. Architect Mental Models (quick recall)

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

---

## 15\. Interview-Ready One-Liners

- "RAG is not Vector DB \+ LLM — it's ingestion \+ indexing \+ retrieval \+ ranking \+ context assembly \+ prompt construction \+ generation \+ governance."  
- "Small chunks \= precision, large chunks \= context — parent-child RAG exists specifically to resolve that trade-off."  
- "Over-retrieve first, refine later — the retriever optimizes recall, the reranker optimizes precision."  
- "Retriever finds candidates, reranker judges candidates — long-list vs. interview panel."  
- "Cross-encoders compare jointly, embedding models compare separately — that's why cross-encoders win on relevance and lose on speed."  
- "Quality lost early in the pipeline cannot be recovered later — a perfect LLM can't fix a chunk that was never retrieved."  
- "Vector search finds similar text; graph traversal finds connected facts — pick based on what your questions actually need."  
- "RAG answers 'what do our documents say'; fine-tuning changes 'how the model behaves'; agents decide 'what to do next'; knowledge graphs answer 'how things relate.'"  
- "If you can't measure recall, faithfulness, and citation accuracy separately, you can't tell whether a bad answer is a retrieval problem or a generation problem."  
- "In RAG you don't pay for AI — you pay per token, per query, per stage, forever."  
- "Governance is what separates a PoC from a product; security architecture is what gets you permission to run it on real data at all."

---

## 16\. One-Page Summary

RAG prepares enterprise knowledge offline (load → clean → chunk → enrich with metadata → embed → index), then at runtime takes a question through identity checks, query processing, security-filtered hybrid search, reranking, context assembly, prompt construction, and generation — wrapped in guardrails, citations, and logging. Quality flows through this chain and can only be preserved or lost at each stage, never recovered downstream, which is why chunking and metadata decisions matter as much as model choice. Before building any of it, ask whether RAG is even the right tool versus fine-tuning, long-context, agents, or knowledge graphs — and if it is, pick the *pattern* (naive, hybrid, multi-query, parent-child, graph, agentic, or adaptive) that matches your actual question types rather than defaulting to the most complex one available. Wrap the system in a real security model (RBAC/ABAC/ACLs/row-level security/tenant isolation), measure it on both retrieval and answer quality (recall, faithfulness, citation accuracy), watch it continuously in production (latency, cost, hallucination rate, access-denial events), and price it like what it is — a perpetual, per-token, per-query cost. Do all of that, and you've moved from "I can explain RAG" to "I can architect, secure, run, and evolve a RAG platform" — which is the actual job.  
