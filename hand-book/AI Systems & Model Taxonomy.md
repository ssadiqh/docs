# AI Solution Architect Handbook

## Table of Contents

1. [Enterprise AI Systems & Model Taxonomy](#enterprise-ai-systems--model-taxonomy)
2. [Retrieval Enhancement & Orchestration](#retrieval-enhancement--orchestration)
3. [Advanced RAG Patterns](#advanced-rag-patterns)
4. [Evaluation & Observability](#evaluation--observability)
5. [Appendix: Reference Tables](#appendix-reference-tables)

---

# Enterprise AI Systems & Model Taxonomy

## Purpose

This chapter establishes the foundational mental model for enterprise AI architecture.

Most architects think: **User → LLM → Answer**

Enterprise AI actually works: **User → Specialized AI Pipeline → LLM → Evaluation → Answer**

---

## The Enterprise AI Processing Pipeline

### Visual Model

```
Ingestion Phase
    ↓
Query Understanding Phase
    ↓
Routing Phase
    ↓
Retrieval Phase
    ↓
Context Assembly Phase
    ↓
Prompt Construction Phase
    ↓
Generation Phase
    ↓
Safety & Governance Phase
    ↓
Evaluation & Observability Phase
    ↓
Answer
```

### Key Insight

An enterprise AI application is **not a single LLM sitting in the middle**. It is a **pipeline of specialized models**, each solving a specific problem.

---

## AI Model Taxonomy

Enterprise systems use **8 model families**:

| Model Family          | Purpose                                    | Examples                          |
| --------------------- | ------------------------------------------ | --------------------------------- |
| Embedding Models      | Convert text to vectors for semantic search | OpenAI Embeddings, BGE, E5        |
| Classification Models | Detect intent, route queries, categorize   | Small LLMs, FastText              |
| Extraction Models     | Extract metadata, entities, PII            | spaCy NER, GLiNER, Small LLMs     |
| Re-ranking Models     | Improve retrieval ordering                 | bge-reranker, Cohere Rerank       |
| Generative LLMs       | Reason and generate answers                | GPT-4, Claude Sonnet, Gemini      |
| Agent Models          | Plan and orchestrate tool usage            | GPT-4, Claude Sonnet              |
| Safety Models         | Moderation, compliance, PII blocking       | Llama Guard, Presidio             |
| Judge Models          | Evaluate quality and correctness           | Judge LLMs, evaluation frameworks |

---

## AI Models Throughout the Enterprise Lifecycle

### Ingestion Phase

| Component                 | AI Model Needed? | Typical Model Type       | Examples                                  |
| ------------------------- | ---------------- | ------------------------ | ----------------------------------------- |
| Document Classification   | Optional         | Small LLM / Classifier   | GPT-4o-mini, Claude Haiku                 |
| Metadata Extraction       | Often            | Small LLM                | GPT-4o-mini, Claude Haiku                 |
| Entity Extraction         | Often            | NER Model / Small LLM    | spaCy NER, GLiNER, GPT-4o-mini            |
| PII Detection             | Often            | Safety Model             | Presidio, Azure AI Content Safety         |
| Document Summarization    | Sometimes        | Small LLM                | Claude Haiku, GPT-4o-mini                 |
| Chunking Optimization     | Sometimes        | Small LLM                | GPT-4o-mini                               |
| Embedding Generation      | Yes              | Embedding Model          | text-embedding-3-large, bge-large, e5     |

### Query Understanding Phase

| Component                  | AI Model Needed? | Typical Model Type | Examples     |
| -------------------------- | ---------------- | ------------------ | ------------ |
| Intent Detection           | Often            | Small LLM          | GPT-4o-mini  |
| Query Classification       | Often            | Small LLM          | Claude Haiku |
| Language Detection         | Sometimes        | Classifier         | FastText     |
| User Context Understanding | Often            | Small LLM          | GPT-4o-mini  |
| Query Rewrite              | Often            | Small LLM          | GPT-4o-mini  |
| Query Expansion            | Often            | Small LLM          | Claude Haiku |
| Query Decomposition        | Often            | Small LLM          | GPT-4o-mini  |

### Routing Phase

| Component                | AI Model Needed? | Typical Model Type      | Examples               |
| ------------------------ | ---------------- | ----------------------- | ---------------------- |
| Router RAG               | Often            | Small LLM               | GPT-4o-mini            |
| Domain Selection         | Often            | Small LLM               | Claude Haiku           |
| Knowledge Base Selection | Often            | Small LLM               | Llama 3 8B             |
| Tool Selection           | Often            | Agent Model             | GPT-4.1, Claude Sonnet |
| Workflow Selection       | Often            | Small LLM               | GPT-4o-mini            |

### Retrieval Phase

| Component             | AI Model Needed? | Typical Model Type | Examples                                   |
| --------------------- | ---------------- | ------------------ | ------------------------------------------ |
| Embedding Generation  | Yes              | Embedding Model    | Same as ingestion phase                    |
| Vector Search         | No               | Search Engine      | Vector DB                                  |
| Keyword Search        | No               | Search Engine      | BM25                                       |
| Metadata Filtering    | No               | Search Engine      | Database / Search Engine                   |
| Re-ranking            | Often            | Re-ranker Model    | bge-reranker-large, Cohere Rerank          |
| Retrieval Evaluation  | Often            | Judge LLM          | GPT-4o-mini                                |
| Corrective RAG Check  | Often            | Judge LLM          | GPT-4o-mini                                |

### Context Assembly Phase

| Component              | AI Model Needed? | Typical Model Type | Examples     |
| ---------------------- | ---------------- | ------------------ | ------------ |
| Context Compression    | Sometimes        | Small LLM          | Claude Haiku |
| Context Summarization  | Sometimes        | Small LLM          | GPT-4o-mini  |
| Duplicate Removal      | Sometimes        | Small LLM / Rules  | GPT-4o-mini  |
| Context Prioritization | Sometimes        | Small LLM          | GPT-4o-mini  |

### Prompt Construction Phase

| Component                 | AI Model Needed? | Typical Model Type | Examples                    |
| ------------------------- | ---------------- | ------------------ | --------------------------- |
| Prompt Builder            | Usually No       | Templates          | Jinja, Format Strings       |
| Dynamic Prompt Creation   | Sometimes        | Small LLM          | GPT-4o-mini                 |
| Prompt Optimization       | Sometimes        | Small LLM          | Claude Haiku                |
| Prompt Injection Detection | Often            | Safety LLM         | Llama Guard, Content Safety |

### Generation Phase

| Component                    | AI Model Needed? | Typical Model Type | Examples                        |
| ---------------------------- | ---------------- | ------------------ | ------------------------------- |
| Task Planning                | Often            | Agent LLM          | GPT-4.1, Claude Sonnet          |
| Tool Selection               | Often            | Agent LLM          | GPT-4.1                         |
| Tool Calling / Reflection    | Often            | Agent LLM          | Claude Sonnet, GPT-4.1          |
| Self-Critique                | Often            | Agent LLM          | Claude Sonnet                   |
| Final Reasoning              | Yes              | Large LLM          | GPT-4.1, Claude Sonnet, Gemini  |
| Response Generation          | Yes              | Large LLM          | GPT-4.1                         |
| Structured Output Generation | Sometimes        | Large LLM          | GPT-4.1                         |

### Safety & Governance Phase

| Component                 | AI Model Needed? | Typical Model Type | Examples                       |
| ------------------------- | ---------------- | ------------------ | ------------------------------ |
| Input Moderation          | Often            | Safety Model       | Llama Guard, Content Safety    |
| Prompt Injection Detection | Often            | Safety Model       | Llama Guard                    |
| PII Detection & Masking   | Often            | Safety Model       | Presidio                       |
| Toxicity Detection        | Often            | Safety Model       | Azure AI Content Safety        |
| Policy Validation         | Often            | Judge LLM          | GPT-4o-mini                    |
| Compliance Validation     | Often            | Judge LLM          | Claude Haiku                   |
| Citation Validation       | Sometimes        | Judge LLM          | GPT-4o-mini                    |
| Hallucination Detection   | Often            | Judge LLM          | GPT-4o-mini                    |

### Evaluation & Observability Phase

| Component                    | AI Model Needed? | Typical Model Type | Examples     |
| ----------------------------- | ---------------- | ------------------ | ------------ |
| Answer Quality Scoring        | Often            | Judge LLM          | GPT-4o-mini  |
| Faithfulness Evaluation       | Often            | Judge LLM          | GPT-4o-mini  |
| Groundedness Evaluation       | Often            | Judge LLM          | Claude Haiku |
| Relevance Evaluation          | Often            | Judge LLM          | GPT-4o-mini  |
| RAGAS-style Evaluation        | Often            | Judge LLM          | GPT-4o-mini  |
| User Feedback Classification  | Often            | Small LLM          | GPT-4o-mini  |
| Drift Detection               | Sometimes        | ML Model           | Custom Model |

---

## Safety Model Placement

### Key Insight

Safety is **not a single component**. Safety is a **cross-cutting concern** that appears throughout the pipeline.

### Safety Checkpoints

| Checkpoint      | Purpose                                      |
| --------------- | -------------------------------------------- |
| Ingestion       | Prevent sensitive information from entering  |
| Query Input     | Detect malicious prompts and injections      |
| Retrieval       | Prevent retrieval of restricted content      |
| Prompt Assembly | Validate context before prompt construction  |
| Generation      | Apply safety constraints during generation   |
| Output          | Validate response before returning to user   |

---

## Decision Framework: When to Use Which Model Type

| Situation                         | Recommended Model Type     | Why                                     |
| --------------------------------- | -------------------------- | --------------------------------------- |
| Semantic similarity search        | Embedding Model            | Optimized for vector operations         |
| Intent/category detection         | Small LLM or Classifier    | Fast, cost-effective classification     |
| Extract structured information    | NER Model or Small LLM     | Domain-specific or general-purpose      |
| Improve retrieval ranking         | Re-ranker Model            | Purpose-built for ranking              |
| Generate text or reason           | Large LLM                  | Reasoning and generation capability    |
| Plan multi-step tasks             | Agent LLM                  | Planning and orchestration              |
| Detect policy violations          | Safety Model               | Optimized for detection                |
| Evaluate output quality           | Judge LLM                  | Quality assessment capability          |

---

## Review Questions

1. Name the 8 model families and their primary purposes.
2. Why is it incorrect to say "we're using GPT-4" as a complete architecture description?
3. At what phases of the pipeline do safety models typically appear?
4. When would you use a small LLM instead of a large LLM?
5. What is the role of embedding models in retrieval?
6. Why do enterprise systems often use multiple model types?
7. How does the judge LLM fit into the overall pipeline?

---

# Retrieval Enhancement & Orchestration

## Purpose

Enterprise systems don't just retrieve information. They retrieve information **intelligently**.

This chapter covers patterns and techniques that improve retrieval quality, selection, validation, and orchestration.

---

## The Retrieval Lifecycle

Enterprise retrieval typically progresses through stages:

```
User Question
      ↓
Query Transformation
      ↓
Router Decision
      ↓
Retrieval Execution
      ↓
Retrieval Validation
      ↓
Re-ranking
      ↓
Context Assembly
      ↓
LLM Generation
```

Each stage solves a different retrieval challenge.

---

## Query Transformation

### Purpose

Improve retrieval by improving the query.

### Problem

Users ask vague, incomplete, or poorly-phrased questions.

Example:
```
"leave policy?"
```

Instead of:
```
"What is the annual leave entitlement policy for employees?"
```

### Techniques

| Technique               | Example                                                     | When to Use                              |
| ----------------------- | ----------------------------------------------------------- | ---------------------------------------- |
| Query Rewriting         | "leave" → "annual leave entitlement"                        | When query is too vague                  |
| Query Expansion         | "AI" → "AI, ML, Generative AI, Artificial Intelligence"    | When synonyms matter                     |
| Query Decomposition     | Complex question → Multiple sub-questions                   | When question is multi-part              |
| Multi-Query Generation  | Generate 3-5 retrieval variants of same question           | When different phrasings matter          |
| HyDE                    | Generate hypothetical answer, then retrieve using it       | When semantic matching is weak           |
| Step-Back Prompting     | "What is X?" → "What broader concept does X relate to?"   | When specific query is too narrow        |

### Key Insight

**Better questions produce better retrieval.**

### Trade-offs

| Approach            | Pros                          | Cons                               |
| ------------------- | ----------------------------- | ---------------------------------- |
| Simple pass-through | Low latency                   | Low retrieval quality              |
| Query Rewriting     | Moderate improvement          | Requires good LLM                  |
| Multi-Query         | Better coverage               | Higher latency, higher cost        |
| HyDE                | Works for semantic mismatch   | Requires good hypothetical model   |

---

## Router RAG

### Purpose

Select the most appropriate retrieval source before searching.

### Problem

Searching all repositories is inefficient and often returns noise.

### How It Works

```
Question
    ↓
Router LLM
    ↓
Determine intent/domain
    ↓
Select knowledge source
    ↓
Retrieve from selected source
```

### Router Decision Examples

| Question Type             | Routed To        | Why                                 |
| ------------------------- | ---------------- | ----------------------------------- |
| "What is our leave policy?"      | Policy Index     | Structured policy documents         |
| "How should I design X component?" | Architecture DB  | Technical architecture documents    |
| "What depends on Service Y?"      | Dependency Graph | Relationship queries need graphs    |
| "How many users were active yesterday?" | SQL Database | Operational metrics need SQL        |
| "What does API X do?"             | External API     | Latest external documentation       |

### Key Insight

**Choose the correct library before searching for the book.**

### Trade-offs

| Approach        | Pros                   | Cons                              |
| --------------- | ---------------------- | --------------------------------- |
| Single search   | Simple, low latency    | Searches everything, high noise   |
| Router RAG      | Precise targeting      | Requires good routing logic       |
| Pre-filtering   | Fast routing           | Requires domain configuration     |

---

## Hybrid RAG

### Purpose

Improve retrieval by combining multiple retrieval techniques.

### Components

| Component           | Purpose                              |
| ------------------- | ------------------------------------ |
| Vector Search       | Semantic similarity matching         |
| Keyword Search      | Lexical/term matching (BM25)        |
| Metadata Filtering  | Structured field filtering           |
| Re-ranking          | Score reordering and filtering       |

### Architecture

```
Question
    ↓
┌─────────────────────┬──────────────────┐
│                     │                  │
Vector Search    Keyword Search    Metadata Filter
│                     │                  │
└─────────────────────┴──────────────────┘
    ↓
Merge & Deduplicate Results
    ↓
Re-ranking
    ↓
Top K Results
```

### When to Use Each

| Technique      | Best For                          |
| -------------- | --------------------------------- |
| Vector Search  | Semantic concepts, paraphrasing   |
| Keyword Search | Exact terms, acronyms, numbers   |
| Metadata       | Date ranges, categories, authors |
| Hybrid         | Complex queries needing both      |

### Key Insight

**Most enterprise RAG systems are Hybrid RAG systems.**

### Trade-offs

| Approach       | Pros                        | Cons                           |
| -------------- | --------------------------- | ------------------------------ |
| Vector only    | Good semantic matching      | Misses exact terms             |
| Keyword only   | Exact matching              | Poor semantic understanding    |
| Metadata only  | Fast filtering              | Limited to structured data     |
| Hybrid         | Best coverage               | More complex, higher latency   |

---

## Re-ranking

### Purpose

Improve retrieval precision by reordering results.

### Problem

Relevant documents may not appear in the best order after initial retrieval.

### Architecture

```
Retrieve Top N (e.g., 20)
        ↓
Re-ranker Model
        ↓
Return Top K (e.g., 5)
```

### Mental Model

**Retrieval finds candidates. Re-ranking chooses winners.**

### When to Use

| Scenario                                 | Use Re-ranking? |
| ---------------------------------------- | --------------- |
| Simple single-term queries               | No              |
| Complex multi-aspect queries             | Yes             |
| When precision matters more than recall  | Yes             |
| When cost is the primary constraint      | No              |

### Popular Re-ranking Models

* **Cross-Encoders**: Compare query-document pairs directly
* **Dedicated Re-rankers**: BGE Reranker, Cohere Rerank
* **Judge LLMs**: Use LLMs to score relevance

### Trade-offs

| Approach            | Pros                   | Cons                      |
| ------------------- | ---------------------- | ------------------------- |
| No re-ranking       | Low cost, low latency  | Lower precision           |
| Cross-Encoder       | Good precision         | Moderate cost             |
| Dedicated Re-ranker | Optimized for ranking  | Higher cost               |
| Judge LLM           | Flexible scoring       | High cost and latency     |

---

## Corrective RAG (CRAG)

### Purpose

Validate retrieval quality and retry if necessary.

### Problem

Poor retrieval produces poor answers. But we don't always know if retrieval failed.

### Architecture

```
Retrieve
    ↓
Evaluate Retrieval Quality
    ↓
Good?
 ┌──────┴──────┐
Yes            No
 ↓              ↓
Generate      Correct Retrieval
Answer
    ↓
              Re-formulate Query?
              Expand Search?
              Change Strategy?
              Escalate Source?
```

### Corrective Actions

When retrieval quality is poor:

| Action                      | When to Use                              |
| --------------------------- | ---------------------------------------- |
| Retry with expanded search  | When context recall is low               |
| Reformulate query           | When initial query was unclear           |
| Change retrieval strategy   | When vector search failed, try hybrid    |
| Use alternative knowledge   | When source is insufficient              |
| Escalate to human review    | When confidence is very low              |

### Key Insight

**Trust retrieval only after validation.**

### Trade-offs

| Approach                | Pros                    | Cons                        |
| ----------------------- | ----------------------- | --------------------------- |
| No validation           | Low cost, low latency   | May generate poor answers   |
| Simple threshold        | Easy to implement       | Fragile heuristics          |
| LLM-based CRAG          | Intelligent validation  | Higher cost and latency     |
| Multi-strategy CRAG     | Best quality            | Significant complexity      |

---

## Self-RAG

### Purpose

Allow the model to decide whether retrieval is needed and whether retrieved evidence is sufficient.

### Problem

* Not all questions require retrieval (some use training knowledge)
* Retrieved evidence may not be sufficient
* Continuous retrieval for all queries wastes cost and latency

### Architecture

```
Question
    ↓
Need Retrieval?
    (Model decides)
 ┌──────┴──────┐
Yes            No
 ↓              ↓
Retrieve    Answer from
 ↓          Training
Is Useful?     Knowledge
┌───┴───┐
Yes     No
 ↓       ↓
Answer  Retrieve Again
```

### Key Questions the Model Answers

1. Do I need to retrieve information for this query?
2. Is the retrieved information sufficient?
3. Should I search again for more information?

### Comparison: CRAG vs Self-RAG

| Aspect                    | Standard RAG | Corrective RAG | Self-RAG        | Agentic RAG    |
| ------------------------- | ------------ | -------------- | --------------- | -------------- |
| Who evaluates retrieval?  | Nobody       | External LLM   | The model       | Agent/Planner  |
| When is retrieval checked? | Never        | After retrieve | Before & after  | Continuous     |
| Can skip retrieval?       | No           | No             | Yes             | Yes            |
| Implementation complexity | Low          | Medium         | Medium          | High           |

### Key Insight

**The model evaluates retrieval quality itself.**

### Trade-offs

| Approach            | Pros                              | Cons                           |
| ------------------- | --------------------------------- | ------------------------------ |
| Always retrieve     | Predictable, high quality         | High cost, high latency        |
| CRAG               | Validates retrieval              | External evaluation overhead   |
| Self-RAG           | Adaptive cost, flexible           | Requires capable model         |
| Agentic RAG        | Full orchestration, best quality  | Complex, high latency          |

---

## Retrieval Pattern Classification

### Retrieval Quality Patterns

**Goal**: Improve the quality of retrieved results

* Hybrid RAG - Combine multiple retrieval techniques
* Query Transformation - Improve the query
* Re-ranking - Reorder results by relevance
* Parent-Child Retrieval - Search small, generate large

### Retrieval Selection Patterns

**Goal**: Select the correct retrieval source or strategy

* Router RAG - Route to appropriate knowledge base
* Agentic RAG - Agent chooses retrieval strategy

### Retrieval Validation Patterns

**Goal**: Verify retrieval worked and retry if not

* Corrective RAG - External validation with retry
* Self-RAG - Model validates its own retrieval

### Relationship Discovery Patterns

**Goal**: Find connected information across documents

* Multi-Hop Retrieval - Iterative retrieval based on discovered relationships
* Graph RAG - Pre-built relationship graph traversal

### Orchestration Patterns

**Goal**: Coordinate and plan retrieval execution

* Agentic RAG - Full multi-step planning and tool selection

---

## Enterprise Retrieval Pipeline Architecture

A mature enterprise system combines multiple patterns:

```
User Question
    ↓
Query Transformation
    (Ask better questions)
    ↓
Router RAG
    (Choose knowledge source)
    ↓
Agentic RAG
    (Plan retrieval strategy)
    ↓
Hybrid Retrieval
    (Vector + Keyword + Metadata)
    ↓
Graph / Multi-Hop Retrieval
    (Discover relationships)
    ↓
Parent-Child Retrieval
    (Search small, generate large)
    ↓
Re-ranking
    (Order by relevance)
    ↓
Corrective Validation
    (Validate quality, retry if needed)
    ↓
Context Assembly
    (Compress, summarize, prioritize)
    ↓
LLM Generation
    (Generate answer)
    ↓
Answer
```

---

## Common Enterprise Pipelines

### Knowledge Assistant Pipeline

```
Question
    ↓
Router RAG (Policy vs. Technical vs. Operational)
    ↓
Hybrid Retrieval
    ↓
Parent-Child Retrieval
    ↓
Re-ranking
    ↓
LLM
```

### Architecture Assistant Pipeline

```
Question
    ↓
Agentic RAG
    ↓
Graph RAG
    ↓
Hybrid Retrieval
    ↓
LLM
```

### Compliance Assistant Pipeline

```
Question
    ↓
Router RAG
    ↓
Agentic RAG
    ↓
Graph RAG
    ↓
Corrective RAG
    ↓
LLM
```

---

## Decision Framework: Which Patterns to Combine

| Business Need                              | Recommended Pattern Stack                |
| ------------------------------------------ | ---------------------------------------- |
| High precision, narrow domain              | Router + Hybrid + Re-ranking              |
| Complex, multi-step queries                | Agentic + Graph + Multi-Hop              |
| Strict compliance/governance requirements | Router + Agentic + Corrective + Judge    |
| Cost-sensitive with decent quality         | Hybrid + Simple Router                    |
| Maximum quality, no cost constraints       | All patterns combined                    |

---

## Review Questions

1. What is the purpose of query transformation? Give three examples.
2. When would you use Router RAG instead of searching all sources?
3. Compare the trade-offs of vector-only vs. hybrid retrieval.
4. How does CRAG differ from Self-RAG in terms of who evaluates retrieval?
5. Classify the following as Quality/Selection/Validation/Discovery patterns: Hybrid, Router, Multi-Hop, Self-RAG.
6. Design a retrieval pipeline for a healthcare compliance assistant.
7. Why would you use re-ranking if it adds latency?

---

# Advanced RAG Patterns

## Purpose

Beyond basic retrieval enhancement, advanced patterns tackle specific architectural challenges:

* Finding information across relationships
* Handling large documents efficiently
* Enabling intelligent agent behavior
* Supporting complex reasoning tasks

---

## Parent-Child RAG

### Purpose

Search small chunks for precision, but provide larger context for generation.

### Problem

Chunking creates a dilemma:
* Small chunks improve retrieval precision but lose context
* Large chunks improve generation context but hurt retrieval

### Architecture

```
Document Chunking (Ingestion Time)
        ↓
        Chunk Document into:
        - Small chunks (search units)
        - Parent chunks (generation units)
        ↓
Embedding & Indexing
        ↓
Query Time:
        Question
            ↓
        Vector Search Small Chunks
            ↓
        Retrieve Parent Context
            ↓
        LLM (with full parent context)
```

### Visual Example

```
Document:
┌─────────────────────────────────────────┐
│ Full Document (Parent)                   │
├─────────────────┬───────────────────────┤
│ Chunk 1 (Small) │ Chunk 2 (Small)      │
│ Chunk 3 (Small) │ Chunk 4 (Small)      │
└─────────────────┴───────────────────────┘

Index:
Small Chunks → Searchable

Retrieval:
Small Chunk 2 matches query
        ↓
Return Parent (full document)
```

### When to Use

| Scenario                           | Use Parent-Child? |
| ---------------------------------- | ----------------- |
| Long documents (>3000 tokens)      | Yes               |
| Precise technical specifications   | Yes               |
| Short form content                 | No                |
| When context window is limited     | Yes               |

### Trade-offs

| Aspect              | Pros                           | Cons                        |
| ------------------- | ------------------------------ | --------------------------- |
| Small chunks only   | Better retrieval precision     | Lost context for generation |
| Large chunks only   | Full context for generation    | Poor retrieval precision    |
| Parent-Child        | Best of both                   | More complex indexing       |

### Review Questions

1. Why separate the search unit from the generation unit?
2. How do you index parent-child relationships?
3. What document sizes benefit most from parent-child retrieval?

---

## Multi-Hop RAG

### Purpose

Iteratively retrieve information by following discovered relationships.

### Problem

Some information requires finding intermediate concepts:

```
Question: "What systems depend on Service X?"

Hop 1: Find Service X
Hop 2: Find services that depend on X
Hop 3: Find services that depend on those services
...
```

### Architecture

```
Initial Question
        ↓
Retrieve Relevant Information
        ↓
Analyze for Related Entities/Concepts
        ↓
Need More Information?
    ┌───┴────┐
   Yes       No
    ↓         ↓
Generate   Answer
New Search
    ↓
Retrieve Again
    ↓
(loop back)
```

### Real-World Example

**Question**: "What are the compliance impacts if we sunset the authentication service?"

**Hop 1**: Find authentication service
**Hop 2**: Find systems that depend on authentication
**Hop 3**: Find compliance requirements for those systems
**Hop 4**: Synthesize compliance impacts

### Key Decisions

| Decision Point                   | Considerations                         |
| -------------------------------- | -------------------------------------- |
| How many hops? (max depth)       | Cost, latency, diminishing returns     |
| When to stop?                    | Confidence threshold, no new entities  |
| How to reformulate each query?   | Use discovered context to improve     |
| How to avoid infinite loops?     | Track visited entities                |

### Trade-offs

| Approach         | Pros                        | Cons                      |
| ---------------- | --------------------------- | ------------------------- |
| Single retrieval | Low cost, low latency       | Misses relationships      |
| Two hops         | Discovers basic links       | Moderate cost/latency     |
| Multi-hop (3+)   | Deep relationship discovery | Higher cost, latency risk |

### When to Use

| Scenario                                  | Use Multi-Hop? |
| ----------------------------------------- | -------------- |
| Dependency analysis                       | Yes            |
| Root cause investigation                  | Yes            |
| Impact analysis across systems            | Yes            |
| Simple factual queries                    | No             |
| Cost-sensitive applications               | No             |

---

## Graph RAG

### Purpose

Pre-compute and traverse relationship graphs during retrieval.

### Difference from Multi-Hop

| Aspect          | Multi-Hop RAG                    | Graph RAG                      |
| --------------- | -------------------------------- | ------------------------------ |
| Relationship computation | At query time (runtime)    | At ingestion time (once)       |
| Performance     | Slower (multiple LLM calls)     | Faster (graph traversal)       |
| Flexibility     | Dynamic discovery               | Pre-defined relationships      |
| Cost            | Pay per query (LLM calls)        | Pay once (graph construction)  |

### Architecture

```
Ingestion Time:
Document → Extract Entities → Build Relationship Graph → Index

Query Time:
Question
    ↓
Identify Seed Entities
    ↓
Traverse Graph (BFS/DFS)
    ↓
Collect Connected Information
    ↓
LLM (with graph context)
```

### Example Graph Structure

```
Service A ─── depends_on ──→ Service B
  │                             │
  ├─ has_policy → Policy X      └─ has_compliance → Regulation Y
  │                                  │
  └─ owned_by → Team Alpha         └─ enforced_by → Audit Rule Z
```

### When to Use

| Scenario                           | Use Graph RAG? |
| ---------------------------------- | -------------- |
| Complex interconnected systems     | Yes            |
| Relationship queries are common    | Yes            |
| Relationships don't change often   | Yes            |
| Highly dynamic environment         | No             |
| Simple retrieval tasks             | No             |

### Trade-offs

| Aspect                  | Pros                          | Cons                        |
| ----------------------- | ----------------------------- | --------------------------- |
| No graph (simple RAG)    | Simple, low upfront cost      | Misses relationships        |
| Graph                   | Fast relationship queries     | Ingestion complexity        |
|                         | Lower query cost              | Requires entity extraction  |
|                         |                               | Graph maintenance burden    |

### Popular Graph Approaches

* **Neo4j + Vector Search**: Graph DB + semantic search
* **Knowledge Graphs**: Pre-built from domain ontologies
* **Dynamic Graphs**: Built from extracted entities at ingestion
* **Hybrid**: Combine static and dynamic relationships

---

## Agentic RAG

### Purpose

Enable LLMs to plan and execute multi-step retrieval strategies.

### Problem

Pre-defined retrieval patterns may not fit all questions:

```
Question: "What are the architecture implications of adopting Service X?"

Requires:
- Domain/capability lookup
- Architecture pattern matching
- Compliance requirement checking
- Performance implication analysis
- Cost impact assessment
```

No single retrieval pattern handles all aspects.

### Architecture

```
Question
    ↓
Agent (LLM with Tools)
    ↓
Plan Retrieval Steps
    ↓
Tool Selection (which retriever to use?)
    ↓
Execute Tool
    ↓
Observe Result
    ↓
Need More Information?
    ┌────┴────┐
   Yes        No
    ↓          ↓
  Loop      Generate Answer
    ↓
Return Answer
```

### Tool Examples

An agent might have access to:

| Tool              | Purpose                       |
| ----------------- | ----------------------------- |
| Vector Search     | Semantic concept search       |
| Graph Traversal   | Relationship discovery        |
| SQL Query         | Operational data lookup       |
| API Call          | External system integration   |
| Code Search       | Repository exploration        |

### When to Use Agentic RAG

| Scenario                               | Use Agentic RAG? |
| -------------------------------------- | ---------------- |
| Questions require multiple tools       | Yes              |
| Tool selection varies by question      | Yes              |
| Complex multi-step reasoning           | Yes              |
| Simple factual queries                 | No               |
| Cost-critical applications             | No               |

### Key Decisions

| Decision                    | Considerations                         |
| --------------------------- | -------------------------------------- |
| Which tools to expose?      | Capability vs. complexity trade-off    |
| Planning approach?          | Pre-planning vs. reactive               |
| How many steps max?         | Latency and cost limits                |
| Failure handling?           | Retry, escalate, or fallback           |

### Trade-offs

| Approach             | Pros                         | Cons                       |
| -------------------- | ---------------------------- | -------------------------- |
| Fixed patterns       | Predictable, low cost        | Inflexible                 |
| Simple agent         | Flexible, can adapt          | Higher cost and latency    |
| Advanced agent       | Full multi-step planning     | Complex, expensive, slower |

---

## Pattern Comparison Matrix

| Pattern        | Best For                    | Latency | Cost   | Complexity |
| -------------- | --------------------------- | ------- | ------ | ---------- |
| Parent-Child   | Large documents             | Low     | Low    | Low        |
| Multi-Hop      | Dependency discovery        | Medium  | Medium | Medium     |
| Graph RAG      | Relationship queries        | Low     | Low*   | High       |
| Agentic RAG    | Complex reasoning           | High    | High   | High       |

*High initial cost for graph construction, low per-query cost

---

## Pattern Evolution

```
Standard RAG
    ↓
(Need better precision?)
    ↓
Parent-Child RAG
    ↓
(Need relationships?)
    ↓
Multi-Hop RAG
    ↓
(Relationships are frequent?)
    ↓
Graph RAG
    ↓
(Need flexible planning?)
    ↓
Agentic RAG
```

---

## Combining Advanced Patterns

Most enterprise systems don't use a single pattern. They combine them:

### Example: Compliance Assistant

```
Question
    ↓
Router (Route to right compliance domain)
    ↓
Agentic RAG
    ├─ Tool 1: Policy Vector Search
    ├─ Tool 2: Regulation Graph Traversal
    ├─ Tool 3: Audit Trail SQL Query
    └─ Tool 4: Previous Risk Assessment Search
    ↓
Parent-Child Retrieval (for policy docs)
    ↓
Corrective Validation
    ↓
Multi-Model Analysis
    ├─ Judge LLM (compliance assessment)
    ├─ Judge LLM (risk scoring)
    └─ Judge LLM (recommendation confidence)
    ↓
Answer
```

---

## Review Questions

1. When would you use Parent-Child retrieval instead of standard chunking?
2. How does Graph RAG reduce query cost compared to Multi-Hop?
3. Design an agentic RAG system for an IT operations assistant.
4. What are the trade-offs between multi-hop and graph RAG?
5. How would you combine agentic RAG with corrective RAG?
6. When does the complexity of agentic RAG justify its benefits?

---

# Evaluation & Observability

## Purpose

Building a RAG system is only half the problem.

The real challenge is answering:

* Is retrieval working?
* Is the answer correct?
* Is the answer grounded?
* Is the system reliable?
* Is the system delivering business value?

---

## Why Evaluation Matters

### Traditional Software Testing

```
Input → Expected Output → Pass / Fail
```

Easy to test. 2 + 2 = 4, always.

### AI System Testing

```
Question → Retrieved Context → Answer
```

Multiple answers may be acceptable. This is **not a pass/fail problem**; it's a **measurement problem**.

---

## Evaluation Hierarchy

```
Evaluation
│
├── Retrieval Quality
│   └─ Precision, Recall, Context Precision, Context Recall
│
├── Generation Quality
│   └─ Groundedness, Faithfulness, Correctness, Relevance
│
├── Operational Quality
│   └─ Latency, Cost, Availability, Token Usage
│
└── Business Quality
    └─ User Satisfaction, Adoption, Task Success, ROI
```

---

## Retrieval Evaluation

### Precision

**Question**: Of the documents we retrieved, how many were actually useful?

**Formula**:
```
Precision = Relevant Retrieved / Total Retrieved
```

**Example**:
```
Retrieved: [Doc A, Doc B, Doc X, Doc Y]
Relevant:  [Doc A, Doc B, Doc C, Doc D]

Precision = 2/4 = 50%
```

**Mental Model**: Avoid junk in results.

### Recall

**Question**: Did we retrieve everything important?

**Formula**:
```
Recall = Relevant Retrieved / Total Relevant
```

**Example**:
```
Relevant in corpus:  [Doc A, Doc B, Doc C, Doc D]
Retrieved:          [Doc A, Doc B]

Recall = 2/4 = 50%
```

**Mental Model**: Avoid missing important information.

### Context Precision

**Question**: How much of the retrieved context was actually useful for answering?

**Example**:
```
Retrieved: [CPS 230, CPS 234, Holiday Policy, Cafeteria Menu]
Useful:    [CPS 230]

Context Precision = 1/4 = 25%
```

**Mental Model**: How much irrelevant context was retrieved?

### Context Recall

**Question**: Did we retrieve all information needed to answer the question?

**Example**:
```
Required for answer: [Requirement A, B, C, D]
Retrieved:          [Requirement A, B]

Context Recall = 2/4 = 50%
```

**Mental Model**: Did we retrieve enough information?

### Retrieval Metrics Summary

| Metric             | Measures                              | Formula                        |
| ------------------ | ------------------------------------- | ------------------------------ |
| Precision          | Useful results / Total results        | TP / (TP + FP)                 |
| Recall             | Found results / Total available       | TP / (TP + FN)                 |
| Context Precision  | Useful chunks / Total chunks          | Useful chunks / Retrieved      |
| Context Recall     | Complete context / Required context   | Retrieved required / All needed |

---

## Pattern Impact on Retrieval Metrics

| Pattern              | Improves                 |
| -------------------- | ------------------------ |
| Hybrid RAG           | Precision + Recall       |
| Query Transformation | Recall                   |
| Re-ranking           | Precision                |
| Parent-Child         | Context Recall           |
| Router RAG           | Precision                |
| Graph RAG            | Context Recall           |
| Multi-Hop            | Recall                   |
| Corrective RAG       | Precision + Recall       |

---

## Generation Evaluation

### Groundedness

**Question**: Is the answer supported by retrieved evidence?

**Example**:
```
Context: "Employees receive 20 days annual leave."

Answer: "Employees receive 20 days annual leave."
Grounded? YES ✓

Answer: "Employees receive 25 days annual leave."
Grounded? NO ✗
```

**Mental Model**: Supported by evidence?

### Faithfulness

**Question**: Did the model faithfully use the retrieved evidence without inventing facts?

**Example**:
```
Context: "20 days annual leave"

Answer: "20 days annual leave"
Faithful? YES ✓

Answer: "20 days annual leave plus 10 wellness days"
Faithful? NO ✗ (added information not in context)
```

**Mental Model**: No hallucinations?

### Correctness

**Question**: Is the answer objectively true in the real world?

**Critical Insight**: A response can be grounded and faithful but still be incorrect if the source is outdated.

**Example**:
```
Outdated Document: "Employees receive 20 days leave"
Actual Current Policy: "Employees receive 25 days leave"

Answer: "Employees receive 20 days leave"

Grounded?  YES ✓ (matches document)
Faithful?  YES ✓ (doesn't add facts)
Correct?   NO ✗  (policy changed)
```

**Mental Model**: Real-world truth?

### Relevance

**Question**: Did the answer address the user's question?

**Example**:
```
User: "How many leave days do employees receive?"

Answer: "Refer to HR Policy Section 5.2"

Relevant? POOR ✗
(User wanted the number, not a reference)
```

**Mental Model**: Answer the question?

### Generation Metrics Summary

| Metric       | Key Question                        | Measures                  |
| ------------ | ----------------------------------- | ------------------------- |
| Groundedness | Supported by evidence?              | Evidence backing          |
| Faithfulness | No hallucinated information?        | Consistency with context  |
| Correctness  | Objectively true?                   | Real-world accuracy       |
| Relevance    | Did it answer the question?         | Answer coverage           |

---

## Important Architect Insight

**Groundedness vs. Correctness**

A response can be:
* Grounded ✓
* Faithful ✓

And still be:
* Incorrect ✗

if the retrieved information itself is wrong or outdated.

This is why enterprises invest heavily in:
* Knowledge governance
* Data freshness
* Source validation
* Correctness evaluation beyond RAG metrics

---

## RAGAS Framework

### What Is RAGAS?

RAGAS = **Retrieval-Augmented Generation Assessment**

RAGAS is an **evaluation framework** (not a RAG architecture framework).

It implements and calculates RAG evaluation metrics.

### RAGAS Core Metrics

| Metric            | Measures                                |
| ----------------- | --------------------------------------- |
| Context Precision | Was retrieved context useful?           |
| Context Recall    | Was enough context retrieved?           |
| Faithfulness      | Did model avoid hallucinating?          |
| Answer Relevance  | Did answer address the question?        |

### RAGAS Evaluation Relationships

```
Question
   ↓
Retrieved Context
   ↓
Answer
```

RAGAS evaluates:

| Relationship       | Metric            |
| ------------------ | ----------------- |
| Question ↔ Context | Context Recall    |
| Context Quality    | Context Precision |
| Context ↔ Answer   | Faithfulness      |
| Question ↔ Answer  | Answer Relevance  |

### What RAGAS Does Not Measure

| Area              | RAGAS? |
| ----------------- | ------ |
| Latency           | ❌      |
| Cost              | ❌      |
| Security          | ❌      |
| Governance        | ❌      |
| User Satisfaction | ❌      |
| Business Value    | ❌      |

RAGAS focuses on retrieval and generation quality only.

---

## Operational Evaluation

Purpose: Determine whether the AI system is healthy.

| Metric       | Measures             |
| ------------ | -------------------- |
| Latency      | Response speed       |
| Throughput   | Requests per minute  |
| Availability | System reliability   |
| Token Usage  | Consumption patterns |
| Cost         | Financial efficiency |

---

## Business Evaluation

Purpose: Determine whether the system delivers value.

| Metric            | Measures              |
| ----------------- | --------------------- |
| User Satisfaction | User perception       |
| Adoption          | Usage levels          |
| Task Success Rate | Outcome quality       |
| Resolution Rate   | Business effectiveness |
| ROI               | Financial value       |

---

## Common Evaluation Failure Modes

### High Precision, Low Recall

```
Retrieved: Only one perfect chunk
Problem: Missing information
Result: Incomplete answers
```

### High Recall, Low Precision

```
Retrieved: Everything remotely related
Problem: Huge prompt, high cost, noise
Result: Lower quality answers
```

### High RAGAS Scores, Low Correctness

```
Problem: Source documents are wrong/outdated
Impact: Grounded and faithful answers that are incorrect
Solution: Validate source data quality independently
```

---

## Evaluation Strategy Framework

| Question Asked              | What to Measure              | Best Metrics                |
| --------------------------- | ---------------------------- | --------------------------- |
| Is retrieval working?       | Retrieval quality            | Context Precision, Recall   |
| Are answers grounded?       | Generation grounding         | Groundedness, Faithfulness  |
| Are answers correct?        | Real-world accuracy          | Correctness, Manual review  |
| Is the system fast enough?  | Performance                  | Latency, Throughput        |
| Is the system affordable?   | Cost efficiency              | Token usage, Cost per query |
| Are users satisfied?        | User perception              | Satisfaction scores, NPS    |
| Does it create business value? | Business impact              | Task success, adoption, ROI |

---

## When to Add Each Evaluation Layer

| When?                        | Add This Layer                    |
| ---------------------------- | --------------------------------- |
| MVP phase                    | RAGAS metrics only                |
| Production launch            | + Latency, availability metrics   |
| Post-launch optimization     | + User satisfaction surveys       |
| Business value assessment    | + ROI and business outcome data   |

---

## Review Questions

1. Explain the difference between precision and recall. When does each matter?
2. Can a response be grounded but incorrect? Give an example.
3. What are the 4 core RAGAS metrics?
4. Why doesn't RAGAS measure correctness?
5. Design an evaluation strategy for a compliance assistant.
6. When would you prioritize Context Precision over Context Recall?
7. What would you measure first if given a limited evaluation budget?

---

# Appendix: Reference Tables

## Quick Reference: AI Model Taxonomy

| Family            | Role                    | Cost  | Latency | Use Cases                              |
| ----------------- | ----------------------- | ----- | ------- | -------------------------------------- |
| Embedding         | Vectorization           | Low   | Low     | Search, clustering, similarity         |
| Classification    | Categorization/Intent   | Low   | Low     | Routing, categorization, detection    |
| Extraction        | Data mining             | Low   | Low     | Entity extraction, metadata, PII       |
| Re-ranking        | Relevance scoring       | Low   | Low     | Improving retrieval order             |
| Generative LLM    | Answer generation       | High  | Medium  | Reasoning, content creation, analysis |
| Agent LLM         | Planning/Orchestration  | High  | High    | Multi-step tasks, complex reasoning   |
| Safety Model      | Policy enforcement      | Low   | Low     | Moderation, compliance, protection    |
| Judge LLM         | Quality evaluation      | Low   | Low     | Scoring, assessment, validation       |

---

## Quick Reference: RAG Pattern Selection

### By Problem Type

**Problem: Retrieval precision is too low**
→ Hybrid RAG, Re-ranking, Router RAG

**Problem: Retrieval recall is too low**
→ Query Transformation, Multi-Hop, Graph RAG

**Problem: Generated answers are hallucinated**
→ Corrective RAG, Self-RAG, Judge LLM evaluation

**Problem: Need relationship discovery**
→ Multi-Hop RAG, Graph RAG

**Problem: Need flexible strategy selection**
→ Agentic RAG

**Problem: Large documents need better chunking**
→ Parent-Child RAG

---

## Quick Reference: Evaluation Metrics at a Glance

### Retrieval Metrics

| Metric             | Low Value Problem       | High Value Meaning          |
| ------------------ | ----------------------- | --------------------------- |
| Precision          | Lots of junk results    | Results are clean           |
| Recall             | Missing important info  | Found needed information    |
| Context Precision  | Irrelevant chunks       | Chunks are relevant         |
| Context Recall     | Incomplete context      | Complete context provided   |

### Generation Metrics

| Metric       | Low Value Problem              | High Value Meaning                |
| ------------ | ------------------------------ | --------------------------------- |
| Groundedness | Answers not backed by evidence | Answers supported by sources      |
| Faithfulness | Model hallucinating            | No invented information           |
| Correctness  | Answers factually wrong        | Answers are objectively true      |
| Relevance    | Not answering the question     | Addresses what user asked         |

---

## Decision Tree: Which Pattern to Use

```
START
  ↓
Need to improve what?
├─ Retrieval Quality?
│  ├─ Vector search misses synonyms? → Query Transformation
│  ├─ Need exact term matching too? → Hybrid RAG
│  ├─ Ranking is poor? → Re-ranking
│  ├─ Large documents? → Parent-Child RAG
│  └─ Done
│
├─ Retrieval Selection?
│  ├─ Question type matters? → Router RAG
│  ├─ Need flexible strategy? → Agentic RAG
│  └─ Done
│
├─ Retrieval Validation?
│  ├─ Want external validation? → Corrective RAG
│  ├─ Want model self-validation? → Self-RAG
│  └─ Done
│
└─ Relationship Discovery?
   ├─ Dynamic relationships? → Multi-Hop RAG
   ├─ Pre-built graph? → Graph RAG
   └─ Done
```

---

## Common Architect Interview Questions & Answers

**Q: Which LLM should we use?**

A: Start by asking "Which AI models are needed at each phase of our pipeline?" LLM selection is less important than retrieval quality, safety controls, and evaluation metrics.

---

**Q: Can we have grounded but incorrect responses?**

A: Yes. If source documents are outdated, a response can be grounded (supported by docs) and faithful (no hallucinations) but still factually incorrect.

---

**Q: When is retrieval good enough?**

A: When context precision + recall meet your business requirements. Use RAGAS to measure these, then decide if pattern improvements are needed.

---

**Q: Hybrid vs Vector-only retrieval?**

A: Vector search is better for semantic concepts; keyword search for exact terms. Most enterprises use hybrid to cover both.

---

**Q: How do we know if CRAG is working?**

A: Measure retrieval quality before and after CRAG validation. Success = improved context precision/recall when retrieval initially failed.

---

**Q: When is parent-child retrieval worth the complexity?**

A: When documents are large (>2000 tokens), when retrieval on small chunks is significantly more precise, and when generation context window supports parent size.

---

## Mental Models Checklist

- [ ] Enterprise AI = Pipeline of specialized models, not single LLM
- [ ] Retrieval quality determines answer quality more than LLM choice
- [ ] Safety is cross-cutting concern, not single component
- [ ] Query matters as much as knowledge base
- [ ] Grounded ≠ Correct
- [ ] Evaluation is measurement problem, not pass/fail
- [ ] Cost and latency matter more as you scale
- [ ] Patterns combine, not compete

---

## Quick Cost/Latency Decision Matrix

| Scenario                        | Use This Approach              |
| ------------------------------- | ------------------------------ |
| Cost-critical, good baseline    | Hybrid RAG + Router            |
| Cost-critical, best quality     | Hybrid + Router + Re-ranking   |
| Latency-critical               | Hybrid only, no re-ranking     |
| Quality-critical               | Add evaluation + feedback loop |
| Complex reasoning required     | Agentic RAG (cost acceptable)  |
| Compliance/governance required | Add Safety + Judge models      |

---

## Next Steps in Your Learning

After completing this handbook, deepen your knowledge in:

1. **Observability**: Traces, spans, debugging AI systems
2. **Security**: Prompt injection, jailbreaks, PII protection
3. **Governance**: Model governance, responsible AI, compliance
4. **Cost Optimization**: Token budgeting, model selection economics
5. **Production Patterns**: Caching, fallbacks, A/B testing

---

**End of Handbook**
