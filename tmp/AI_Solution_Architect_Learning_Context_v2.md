# AI Solution Architect Learning Context v2

_Last Updated: 2026-05-30_

# Student Profile

- Solution Architect with 25+ years experience.
- Goal: Become an AI Solution Architect.
- Focus: Architecture, governance, patterns, trade-offs, operating models and enterprise implementation.
- Not pursuing ML Engineer or Data Scientist path.

# Learning Preferences

1. Mental models before technical details.
2. Architecture analogies.
3. Enterprise examples.
4. Understand "why" before "how".
5. Build a long-term handbook while learning.

# Foundation Progress

Estimated Completion: ~75%

Covered:
- Tokens
- Embeddings
- Attention
- Transformers
- Neurons
- Weights
- Parameters
- Context Windows
- Chunking
- Vector Database Foundations
- Model Knowledge vs Enterprise Knowledge

---

# Topic: Tokens

## My Understanding
Tokens are numerical identifiers representing pieces of text.

## Key Insight
Tokens contain no meaning.

## Mental Model
Token = Customer ID

## Detailed Example
"mortgage" may become a token ID such as 2201.

## Architecture Implications
Every LLM workflow starts with tokenisation.

## Common Misconceptions
Tokens are not words.
Tokens are not embeddings.

## Interview Answer
Q: What is a token?
A: A token is the numerical unit processed by an LLM.

## Confidence
85%

---

# Topic: Embeddings

## My Understanding
Embeddings are numerical representations of meaning.

## Key Insights
- Embeddings represent meaning.
- Embeddings do not represent knowledge.

## Questions Asked
- Are embeddings stored in the model?
- Why do vector databases exist?

## Mental Model
Embedding = Customer Profile

## Detailed Example
loan, mortgage and finance cluster together in semantic space.

## Architecture Implications
Embeddings enable semantic search, RAG and recommendations.

## Common Misconceptions
Embeddings are not knowledge.
Embeddings are not weights.

## Interview Answer
Q: What is an embedding?
A: A numerical representation of meaning used for similarity operations.

## Confidence
75%

---

# Topic: Attention

## My Understanding
Attention determines which information matters most in context.

## Mental Model
Reading a 100-page document and focusing only on relevant pages.

## Detailed Example
In "mortgage application", the word application pays more attention to mortgage than to "the".

## Architecture Implications
Attention quality impacts reasoning and retrieval quality.

## Common Misconceptions
Attention is not retrieval.
Attention is not memory.

## Interview Answer
Q: What problem does attention solve?
A: It allows the model to focus on relevant context rather than treating all words equally.

## Confidence
80%

---

# Topic: Transformers

## My Understanding
Transformers use attention to process relationships between tokens.

## Key Insight
Transformers are computational architectures, not databases.

## Detailed Example
Every token can consider every other token.

## Architecture Implications
Transformers enabled modern LLMs such as GPT, Claude, Gemini and Llama.

## Common Misconceptions
Transformers do not store enterprise knowledge.

## Interview Answer
Q: Why did transformers replace RNNs?
A: Better context handling, scalability and parallel processing.

## Confidence
80%

---

# Topic: Neurons, Weights and Parameters

## My Understanding
Neurons are pattern detectors.
Weights are learned connection strengths.
Parameters are learned values; most are weights.

## Mental Model
Neuron = City
Weight = Road
Signal = Traffic

## Key Insight
Knowledge primarily resides in weights.

## Questions Asked
- Are weights stored in vector space?
- What is a neuron computing?

## Architecture Implications
Model knowledge changes through training, not through vector databases.

## Common Misconceptions
Weights are not embeddings.
Parameters are not tokens.

## Interview Answer
Q: Where does model knowledge live?
A: Primarily in the learned weights of the model.

## Confidence
80%

---

# Topic: Context Windows

## My Understanding
The context window is the model's working memory.

## Mental Model
Context Window = RAM

## Detailed Example
A model with a 128K context window can only consider information inside that limit.

## Architecture Implications
Drives chunking, retrieval design, cost and latency.

## Common Misconceptions
Knowledge is not stored in the context window.

## Interview Answer
Q: What is a context window?
A: The amount of information the model can consider at one time.

## Confidence
70%

---

# Topic: Chunking

## My Understanding
Large documents are broken into smaller retrievable sections.

## Mental Model
Chunk = Chapter of a book

## Detailed Example
A 500-page policy becomes hundreds of smaller chunks.

## Architecture Implications
Essential for RAG and efficient retrieval.

## Common Misconceptions
Entire documents should not usually be sent to an LLM.

## Interview Answer
Q: Why use chunking?
A: To improve retrieval precision and fit information into context limits.

## Confidence
75%

---

# Topic: Vector Database Foundations

## My Understanding
Vector databases store embeddings and enable similarity search.

## Mental Model
Vector Database = Semantic Search Engine

## Detailed Example
Question embedding is compared against document embeddings to find the most relevant chunks.

## Architecture Implications
Core component of most enterprise RAG solutions.

## Common Misconceptions
Vector databases do not store model knowledge.
Vector databases do not replace LLMs.

## Interview Answer
Q: Why use a vector database?
A: To efficiently retrieve semantically similar information at scale.

## Confidence
60%

---

# Biggest Breakthroughs

1. Tokens are IDs, not meaning.
2. Embeddings represent meaning.
3. Weights represent learned knowledge.
4. Context Window = RAM.
5. Model Knowledge and Enterprise Knowledge are separate.
6. RAG exists because enterprise knowledge changes.

---

# Current Architecture Mental Model

Prompt
→ Tokens
→ Embeddings
→ Attention
→ Transformer Layers
→ Weights
→ Probability Calculation
→ Next Token Prediction
→ Response

Enterprise Knowledge

Documents
→ Chunks
→ Embeddings
→ Vector Database

---

# Current Weak Areas

- Cosine Similarity
- Metadata Filtering
- Hybrid Search
- Re-ranking
- RAG Architecture
- Agents
- MCP
- AI Governance
- AI Security

---

# Next Topic

RAG Architecture End-to-End

Topics:
- Retrieval-Augmented Generation
- Retrieval Pipeline
- Query Embeddings
- Similarity Search
- Chunk Retrieval
- Prompt Augmentation
- Hallucination Reduction
- Enterprise RAG Patterns

---

# Instructions For Future ChatGPT Sessions

Use this document as the baseline.
Do not reteach basic token, embedding, attention and transformer concepts.
Continue at Solution Architect level.
Focus on architecture patterns, trade-offs, governance and enterprise implementation.
Start with RAG Architecture unless otherwise requested.
