# VECTOR DB

Embeddings convert meaning into vectors; similarity metrics determine how semantic closeness is measured; vector indexes (HNSW, IVF, ANN) determine how similar vectors are found efficiently at scale.

## 1. Two Different Problems

| Layer                       | Question Being Solved                    | Examples                                           |
| --------------------------- | ---------------------------------------- | -------------------------------------------------- |
| Similarity Mathematics      | "How similar are these two vectors?"     | Cosine Similarity, Euclidean Distance, Dot Product |
| Search Algorithms & Indexes | "How do I find similar vectors quickly?" | ANN, HNSW, IVF                                     |

Think of it as:

```text
Step 1: Find candidate vectors quickly
Step 2: Measure similarity
```

---

## 2. Similarity Mathematics

| Metric             | Mental Model         | What It Measures          |
| ------------------ | -------------------- | ------------------------- |
| Cosine Similarity  | Same direction       | Semantic similarity       |
| Euclidean Distance | Physical distance    | Absolute closeness        |
| Dot Product        | Direction + strength | Similarity plus magnitude |

For AI/RAG:

```text
Cosine Similarity
```

is the one you will hear most often.

---

## 3. Search Algorithms & Indexes

| Term | Mental Model    | Purpose                                    |
| ---- | --------------- | ------------------------------------------ |
| ANN  | Fast shortcut   | Search approximately instead of exactly    |
| HNSW | Road network    | Navigate through vectors using graph links |
| IVF  | Library shelves | Search only the most relevant cluster      |

The important relationship:

```text
ANN
 ├── HNSW
 └── IVF
```

ANN is the category.

HNSW and IVF are implementations.

---

## 4. How They Work Together

The process is:

```text
Documents
    ↓
Embeddings
    ↓
Stored as Vectors
```

At query time:

```text
User Question
      ↓
Embedding
      ↓
ANN Search
(HNSW or IVF)
      ↓
Candidate Vectors
      ↓
Cosine Similarity
      ↓
Top Matches
```

---

## 5. The Simplest Mental Model

Imagine a library.

### IVF

```text
Which shelf should I search?
```

### HNSW

```text
What is the fastest path through the library?
```

### Cosine Similarity

```text
How relevant is this book to my question?
```

---

## 6. The Interview Answer

If someone asks:

> Explain how vector retrieval works.

A concise architect-level answer is:

> Documents are converted into embeddings and stored as vectors. When a user asks a question, the question is also converted into an embedding. The vector database uses an ANN index such as HNSW or IVF to efficiently locate candidate vectors. Similarity is typically calculated using cosine similarity. The most relevant chunks are returned, optionally filtered by metadata, re-ranked, and then supplied to the LLM as context.

That's about 95% of what an AI Solution Architect typically needs to know about vector search.


# Vector Database Comparison

## Product Comparison

| Product | Good 👍 | Bad 👎 | Ugly ⚠️ | Choose When | Avoid When |
| :---- | :---- | :---- | :---- | :---- | :---- |
| Azure AI Search | Excellent hybrid search, RBAC, security, Azure integration | Not a pure vector DB | Can get expensive at scale | Azure ecosystem, enterprise RAG | Need specialized vector features |
| Amazon OpenSearch | Search \+ vector \+ logging in one platform | More operational complexity | Configuration can become messy | AWS-centric enterprise environments | Small teams wanting simplicity |
| Pinecone | Easiest managed vector DB | Proprietary | Cost can grow rapidly | Fastest path to production | Heavy self-hosting requirements |
| Qdrant | Excellent open source, easy to operate | Smaller ecosystem than Pinecone | Fewer enterprise integrations | Self-hosted enterprise RAG | Need fully managed cloud experience |
| Weaviate | Rich feature set, hybrid search, GraphQL | More moving parts | Can become complex quickly | Advanced retrieval requirements | Teams wanting minimal operational overhead |
| pgvector | Reuse existing PostgreSQL skills | Not built for massive vector workloads | Performance ceiling | Small/medium RAG solutions | Millions to billions of vectors |
| Milvus | Extremely scalable | Operationally heavier | Steeper learning curve | Large-scale AI platforms | Small enterprise projects |
| Chroma | Very easy for prototypes | Limited enterprise capabilities | Not ideal for large production systems | Learning and PoCs | Enterprise production workloads |

## Enterprise Ranking

| Rank | Product | Why |
| :---- | :---- | :---- |
| 1 | Azure AI Search | Most enterprise-friendly in Azure |
| 2 | OpenSearch | Most common AWS enterprise choice |
| 3 | Qdrant | Best self-hosted/open-source balance |
| 4 | Pinecone | Simplest managed vector DB |
| 5 | Weaviate | Powerful but more complex |
| 6 | pgvector | Excellent when PostgreSQL already exists |
| 7 | Milvus | Great scale, more operational effort |
| 8 | Chroma | Primarily development/prototyping |

## Common Deployment Patterns

| Cloud | LLM | Vector DB | Use Case |
| :---- | :---- | :---- | :---- |
| Azure | Azure OpenAI | Azure AI Search | Most common Azure pattern |
| AWS | Bedrock | OpenSearch | Most common AWS pattern |
| Self-Hosted | OpenAI / Claude / Llama | Qdrant | Increasingly common |

## Key Considerations

### Nuances to Keep in Mind

**pgvector ranking** — Ranked 6th, but if you already have PostgreSQL in place, the operational simplicity and zero additional infrastructure make it competitive for medium-scale RAG until you hit millions of vectors.

**Qdrant vs. Weaviate** — Qdrant (ranked 3\) is simpler to operate than Weaviate and has a good managed cloud option. Weaviate's GraphQL and hybrid search are nice but add complexity.

**Azure AI Search** — Noted as "not a pure vector DB," but that's because it's hybrid search focused. If you need keyword \+ semantic search in one system, that's a strength, not a weakness.

**Cost at scale** — Managed options (Pinecone, Azure, OpenSearch) have different pricing models. For a 6-month projection, cost-per-query at scale matters more than initial ease. Qdrant self-hosted \+ your own compute can dramatically undercut managed options at 10M+ vectors.

**Chroma positioning** — While ranked last, it's genuinely solid for small production systems (\< 1M vectors, single-machine or small cluster). It's more than just development-focused.  



# Similarity Search Metrics

## 1\. L2 Distance (Euclidean Distance)

### Definition

The L2 distance between two vectors `a` and `b` is the square root of the sum of the squared differences between corresponding elements:

L2(a,b) \= √(∑(aᵢ \- bᵢ)²)

### Properties

- The L2 distance measures the straight-line distance between two points in Euclidean space, following the principles of the Pythagorean theorem.  
- It is sensitive to both the magnitude and direction of the vectors, meaning that changes in either can significantly affect the calculated distance.  
- This distance metric is commonly used in applications involving spatial or geometric data, such as image analysis, computer vision, and geographic mapping.

### Use Case

- A common use case for L2 distance is determining the closest point to a given location in two-dimensional (2D) or three-dimensional (3D) space.  
- This approach is frequently used in computer vision tasks, where spatial proximity between features or objects plays a critical role.

---

## 2\. Dot Product (Inner Product) Similarity and Distance

### Definition

The dot product of two vectors is:

a · b \= ∑(aᵢ × bᵢ)

### Properties

- The dot product of two vectors can be positive, negative, or zero, depending on the angle between them.  
- Larger dot product values typically indicate a higher degree of similarity between the vectors, especially when they are pointing in similar directions.  
- If a distance-like metric is needed, the negative of the dot product can be used. In this case, larger (more negative) values correspond to greater dissimilarity or distance.  
- The dot product is sensitive to both the magnitude and direction of the vectors, meaning that changes in either will affect the result.  
- It is frequently used in machine learning models, such as in neural networks for computing activations or in matrix factorization for recommendation systems.

### Use Case

When the length of the vector (representing something such as relevance, confidence, or quantity) is meaningful. For instance, in recommendation systems, the direction of vectors typically indicates two products are about the same topic, but larger magnitudes might indicate that a product is more popular. In this case using dot product similarity makes sense, because one would want to recommend the products that are not only about the same topic, but also popular.

---

## 3\. Cosine Similarity and Distance

### Definition

Cosine similarity measures the cosine of the angle between two vectors:

cosine\_similarity(a,b) \= (a · b) / (||a|| × ||b||)

**Note:** Cosine similarity is a similarity metric—the greater the cosine similarity, the more similar the vectors are. To convert cosine similarity to cosine distance:

cosine\_distance(a,b) \= 1 \- cosine\_similarity(a,b)

The greater the cosine distance, the less similar the vectors are.

### Alternative Calculation

Cosine similarity can be calculated more efficiently if the vectors are normalized. To normalize a vector, divide it by its L2 norm:

norm(a) \= a / ||a||

### Properties

- Cosine similarity focuses on the orientation of vectors rather than their magnitude, measuring the cosine of the angle between them.  
- It is particularly well-suited for high-dimensional, sparse data, such as text embeddings or term-frequency vectors in natural language processing.  
- This metric is invariant to vector length, meaning that scaling a vector up or down does not affect the similarity score.

### Use Case

A common use case for cosine similarity is measuring document similarity in natural language processing (NLP), where it helps identify texts with similar content regardless of their length.

---

## Choosing the Right Metric

| Metric | Sensitive to Magnitude | Normalized | Best For |
| :---- | :---- | :---- | :---- |
| L2 Distance | ✅ Yes | ❌ No | Spatial data, clustering |
| Cosine Distance | ❌ No | ✅ Yes | Text, embeddings, NLP |
| Dot Product | ✅ Yes | ❌ No | Neural networks, recommender systems |

---

## Practical Considerations

### Normalization

Normalize vectors if you expect to only need to calculate cosine similarity. For many natural language processing tasks and text embedding models, this is the default option.

### High-Dimensional Data

- L2 distance can suffer from the 'curse of dimensionality.' Consider using a different metric or using a dimensionality reduction technique if the data exhibits high dimensionality.  
- Cosine distance often performs better in high dimensions, and is the default choice in many natural language processing and text-based tasks.  
- Dot product can be computed efficiently using matrix operations, and can be used to calculate cosine similarity if the vectors are normalized.

---

## Conclusion

Choosing the right distance metric is crucial for effective similarity search. L2 distance works well for continuous, lower-dimensional data where magnitude matters. Cosine distance excels with high-dimensional, sparse data where direction is more important than magnitude. Dot product offers computational efficiency and is useful when both magnitude and direction contribute to similarity. Understanding these trade-offs helps in selecting the most appropriate metric for your specific use case.  



# Collection vs Index

Collection (Chroma, Qdrant, Milvus), Index (Pinecone), Class (Weaviate), and Namespace (some platforms) are vendor-specific names for the logical container that stores vectors and metadata, typically backed by an internal ANN search index such as HNSW for fast retrieval.