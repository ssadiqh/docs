# RAG Advanced Patterns Handbook

## Purpose of This Chapter

This chapter focuses on advanced retrieval architectures used when Standard RAG becomes insufficient.

This chapter intentionally focuses on:

- Retrieval architecture
- Retrieval orchestration
- Pattern selection
- Trade-offs
- Enterprise implications

Detailed discussions on:

- Security & Governance
- Evaluation & Observability
- Enterprise AI Platforms

are covered in dedicated chapters.

---

# One-View Comparison

| Pattern | Core Problem | Key Idea | Retrieval Behaviour | Best For | Main Trade-off |
|----------|----------|----------|----------|----------|----------|
| Standard RAG | Relevant information exists in retrieved chunks | Retrieve relevant chunks and generate | One semantic search | General knowledge retrieval | Can miss context and relationships |
| Parent-Child Retrieval | Small chunks lose context | Search small, generate large | Child search → Parent lookup | Large documents | More metadata and larger prompts |
| Multi-Hop Retrieval | Information spread across documents | Search, discover, search again | Multiple semantic searches | Distributed knowledge | Cost and latency |
| Graph RAG | Relationships repeatedly rediscovered | Store relationships during ingestion | Graph traversal + evidence retrieval | Relationship-heavy domains | Graph complexity |
| Agentic RAG | Different questions need different retrieval strategies | Let LLM choose retrieval path | Dynamic orchestration | Multi-system enterprise questions | Governance and cost |

---

# Pattern Selection Guide

| Requirement | Recommended Pattern |
|------------|------------|
| Better context from large documents | Parent-Child |
| Knowledge spread across documents | Multi-Hop |
| Relationship traversal | Graph RAG |
| Multiple tools and systems | Agentic RAG |
| Most enterprise implementations | Hybrid + Router + Agentic |

---

# Architecture Evolution

Standard RAG
↓
Parent-Child Retrieval
↓
Multi-Hop Retrieval
↓
Graph RAG
↓
Agentic RAG

Each pattern addresses a limitation of the previous pattern.

---

# Parent-Child Retrieval

## Key Idea

Search small. Generate large.

Search Unit ≠ Generation Unit

## Problem

Small chunks retrieve well but lose context.

Large chunks provide context but retrieve poorly.

## Retrieval Flow

Question
↓
Child Retrieval
↓
Parent Lookup
↓
LLM

## Key Insight

Search on small chunks.

Generate from larger parent sections.

## Architecture Implication

Relationship discovery is not required.

Parent lookup is typically:

- Metadata lookup
- Key lookup
- HashMap-style lookup

rather than another semantic search.

## When to Use

- Policies
- Procedures
- Manuals
- Large documents

## Trade-offs

Advantages:

- Better retrieval precision
- Better contextual completeness

Disadvantages:

- Additional metadata
- Larger prompts
- Higher token consumption

---

# Multi-Hop Retrieval

## Key Idea

Search → Discover → Search Again

## Problem

Answers are distributed across multiple documents.

## Retrieval Flow

Question
↓
Search #1
↓
Discover New Fact
↓
Search #2
↓
Answer

## Key Insight

Relationships are discovered during retrieval.

## Architecture Implication

Retrieval becomes a process.

Retrieve
↓
Learn
↓
Retrieve Again

## When to Use

- Dependency analysis
- Root-cause analysis
- Regulatory mapping
- Cross-domain knowledge discovery

## Trade-offs

Advantages:

- Finds distributed knowledge
- Supports complex questions

Disadvantages:

- Multiple semantic searches
- Higher latency
- Higher cost

---

# Graph RAG

## Key Idea

Discover relationships during ingestion, not retrieval.

## Problem

Multi-Hop repeatedly discovers relationships that are already known.

## Retrieval Flow

Question
↓
Entity Resolution
↓
Graph Traversal
↓
Evidence Retrieval
↓
LLM

## Key Insight

Multi-Hop discovers relationships at runtime.

Graph RAG discovers relationships at ingestion time.

## Critical Distinction

Multi-Hop:

Search
↓
Search
↓
Search

Graph RAG:

Traverse
↓
Traverse
↓
Traverse

Graph traversal may involve many graph hops while remaining a single graph operation.

## Typical Architectures

### Separate Platforms

Graph DB
+
Vector DB

Examples:

- Neo4j + Pinecone
- Neo4j + Azure AI Search
- Neptune + Vector Store

### Unified Platforms

- Neo4j
- Weaviate
- Cosmos DB

supporting:

Graph Traversal
+
Vector Search

## When to Use

- Enterprise architecture
- Application dependencies
- Fraud detection
- Supply chain networks
- Regulatory mapping

## Trade-offs

Advantages:

- Relationship-aware retrieval
- Explainability
- Efficient dependency traversal

Disadvantages:

- Graph construction complexity
- Entity extraction challenges
- Graph maintenance

---

# Agentic RAG

## Key Idea

Developer defines tools.

LLM defines tool sequence.

## Problem

Different questions require different retrieval strategies.

## Agent Model

Agent =
LLM
+ Tools
+ Execution Loop
+ State

## Retrieval Flow

Question
↓
Plan
↓
Choose Tool
↓
Execute
↓
Observe
↓
Choose Next Tool
↓
Answer

## What Is A Plan?

A plan is a dynamically generated sequence of information gathering steps required to answer a question.

## Important Clarification

Agentic RAG is not a retrieval algorithm.

It is retrieval orchestration.

## Architecture Insight

The intelligence remains inside the LLM.

The agent framework provides:

- Tool execution
- State
- Memory
- Control loop

## When to Use

- Multiple systems
- Multiple databases
- Multiple APIs
- Enterprise workflows

## Trade-offs

Advantages:

- Flexible
- Dynamic
- Enterprise-friendly

Disadvantages:

- More LLM calls
- More latency
- More cost
- More governance requirements

---

# Retrieval Enhancement Patterns

These patterns are commonly used alongside the major advanced patterns.

## Hybrid RAG

Combine:

- Keyword Search (BM25)
- Vector Search
- Metadata Filtering
- Re-ranking

Key Idea:

Use multiple retrieval techniques together to improve recall and precision.

Most enterprise RAG systems are Hybrid RAG systems.

---

## Router RAG

Question
↓
Router
↓
Choose Best Knowledge Source

Examples:

Policy Question
→ Policy Index

Architecture Question
→ Architecture Index

Dependency Question
→ Graph

Operational Question
→ SQL

External Question
→ API

Key Idea:

Route the question to the most appropriate retrieval system.

---

## Query Transformation

Techniques:

- Query Rewriting
- Query Expansion
- Query Decomposition
- HyDE
- Step-back Prompting

Key Idea:

Improve retrieval by improving the query.

---

## Corrective RAG

Key Idea:

Evaluate retrieval quality before generating an answer.

Typical Actions:

- Retry retrieval
- Reformulate query
- Retrieve additional evidence
- Escalate to another retrieval strategy

Note:

Detailed evaluation approaches are covered in the Evaluation & Observability chapter.

---

# Implementation Comparison

| Pattern | Storage | Runtime Behaviour | Cost | Latency | Complexity |
|----------|----------|----------|----------|----------|----------|
| Standard RAG | Vector DB | Search | Low | Low | Low |
| Parent-Child | Vector + Metadata | Search + Lookup | Low | Low | Medium |
| Multi-Hop | Vector DB | Search + Search | Medium-High | Medium-High | Medium-High |
| Graph RAG | Graph + Vector | Traverse + Evidence | Medium | Low-Medium | High |
| Agentic RAG | Multiple Systems | Dynamic Tool Selection | High | High | Very High |

---

# Architecture Summary

Parent-Child
→ Fixes Context Loss

Multi-Hop
→ Fixes Distributed Knowledge

Graph RAG
→ Fixes Relationship Discovery

Agentic RAG
→ Fixes Retrieval Strategy Selection

---

# Final Interview Summary

Parent-Child

Search small.
Generate large.

Multi-Hop

Search repeatedly to discover distributed knowledge.

Graph RAG

Store relationships beforehand and traverse them.

Agentic RAG

Allow the LLM to dynamically choose retrieval strategies and tools.

---

# What To Remember

Parent-Child

Search Unit ≠ Generation Unit

Multi-Hop

Relationship Discovery at Runtime

Graph RAG

Relationship Discovery at Ingestion Time

Agentic RAG

Developer Defines Tools
LLM Defines Tool Sequence


---

# Pattern Relationships & Composition

## Why This Matters

Advanced RAG patterns are not mutually exclusive.

Enterprise systems rarely choose a single pattern. Instead, multiple patterns are combined to address retrieval quality, relationship discovery, orchestration, and answer completeness.

The architectural challenge is typically:

How do these patterns work together?

rather than:

Which single pattern should I use?

---

## Example 1 — Enterprise Knowledge Assistant

Router RAG
      ↓
Hybrid Retrieval
      ↓
Parent-Child Retrieval
      ↓
Re-ranking
      ↓
LLM

Purpose:

Route query to correct knowledge source
↓
Improve retrieval recall
↓
Improve context completeness
↓
Improve ranking quality
↓
Generate answer

---

## Example 2 — Enterprise Architecture Assistant

Agentic RAG
      ↓
Graph RAG
      ↓
Hybrid Retrieval
      ↓
LLM

Purpose:

Choose retrieval strategy
↓
Traverse dependencies
↓
Retrieve supporting evidence
↓
Generate answer

---

## Example 3 — Compliance & Regulatory Assistant

Router RAG
      ↓
Agentic RAG
      ↓
Graph RAG
      ↓
Corrective RAG
      ↓
LLM

Purpose:

Route correctly
↓
Select tools
↓
Traverse regulatory relationships
↓
Validate retrieval quality
↓
Generate answer

---

## Relationship Summary

Parent-Child
    ↓
Improves Context Completeness

Hybrid
    ↓
Improves Retrieval Quality

Router
    ↓
Selects Knowledge Source

Multi-Hop
    ↓
Discovers Relationships Dynamically

Graph
    ↓
Stores and Traverses Relationships

Agentic
    ↓
Orchestrates Retrieval Strategies

Corrective
    ↓
Validates Retrieval Quality

---

## Key Architecture Insight

Graph RAG does not replace Hybrid RAG.

Agentic RAG does not replace Graph RAG.

Router RAG often sits above all retrieval patterns.

Corrective RAG can be applied to any retrieval pattern.

Parent-Child Retrieval can be combined with all other patterns.

---

## Typical Enterprise Retrieval Stack

User Question
      ↓
Router RAG
      ↓
Agentic RAG
      ↓
Hybrid Retrieval
      ↓
Graph / Multi-Hop Retrieval
      ↓
Parent-Child Retrieval
      ↓
Re-ranking
      ↓
LLM

---

## Final Takeaway

Standard RAG
    = Retrieve Information

Advanced RAG
    = Retrieve Information Intelligently

Enterprise RAG
    = Combine Multiple Retrieval Patterns
      To Deliver Accurate,
      Complete,
      Explainable Answers
