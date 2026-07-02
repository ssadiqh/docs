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


---

# Week 2 Progress – Adaptation Layer

Estimated Completion: ~80%

Covered:
- In-Context Learning (ICL)
- Fine-Tuning
- Instruction Fine-Tuning
- PEFT / LoRA

## Topic: In-Context Learning (ICL)

### My Understanding
ICL allows an LLM to perform tasks by observing examples provided within the prompt, without changing model weights.

### Key Insights
- ICL changes behaviour, not knowledge.
- Model weights remain unchanged.
- Examples act as temporary guidance inside the context window.
- Context Window = Temporary Learning Workspace.

### Mental Model
ICL = Teaching by Examples

### Architecture Implications
Use ICL for:
- Output formatting
- Response style
- Rapid prototyping
- Frequently changing requirements

Use RAG instead when knowledge is large, dynamic, or enterprise-scale.

### Interview Answer
Q: What is In-Context Learning?
A: In-Context Learning allows an LLM to perform tasks using examples supplied within the prompt without modifying model weights.

### Confidence
85%

---

## Topic: Fine-Tuning

### My Understanding
Fine-tuning changes model weights using task-specific training data to permanently improve behaviour for specialised tasks.

### Key Insights
- Fine-tuning modifies weights.
- Fine-tuning adapts behaviour.
- Dynamic enterprise knowledge should usually be handled through RAG.

### Mental Model
Fine-Tuning = Formal Training

### Architecture Implications
Good for:
- Classification
- Structured outputs
- Stable domain-specific tasks

Poor for:
- Frequently changing policies
- Dynamic enterprise knowledge

### Confidence
80%

---

## Topic: Instruction Fine-Tuning

### My Understanding
Instruction Fine-Tuning teaches a model how to behave like an assistant by training it on instruction-response pairs.

### Key Insights
- Creates assistant behaviour.
- Improves instruction following.
- Does not provide current enterprise knowledge.

### Mental Model
Pre-Training = Learning Language
Instruction Fine-Tuning = Learning How To Help

### Confidence
80%

---

## Topic: PEFT / LoRA

### My Understanding
LoRA is a Parameter Efficient Fine-Tuning technique that customises model behaviour using small adapter layers rather than retraining the entire model.

### Key Insights
- LoRA modifies a small subset of parameters.
- Much cheaper than full fine-tuning.
- Often combined with RAG.

### Mental Model
Full Fine-Tuning = Rewrite the Handbook
LoRA = Add a Supplement

### Confidence
75%

---

# Major Breakthroughs From Week 2

1. ICL changes behaviour.
2. RAG provides knowledge.
3. Fine-Tuning changes weights.
4. Instruction Fine-Tuning creates assistant behaviour.
5. LoRA enables low-cost model customisation.
6. Most enterprise AI systems use Instruction-Tuned Models + RAG.
7. Behaviour Problems and Knowledge Problems require different solutions.

---

# AI Architecture Patterns

## Behaviour Patterns
- Prompt Engineering
- Prompt Templates
- In-Context Learning
- Instruction Fine-Tuning
- Fine-Tuning
- LoRA / PEFT

## Knowledge Patterns
- RAG
- Hybrid Search
- Re-ranking
- Knowledge Graph RAG

## Reasoning Patterns
- Tool Calling
- Agents
- Multi-Agent Systems

## Integration Patterns
- MCP
- Workflow Orchestration

## Governance Patterns
- Guardrails
- RLHF
- Constitutional AI

---

# Architect Decision Framework

Behaviour Problem?
→ Prompt Engineering
→ ICL
→ Instruction Fine-Tuning
→ Fine-Tuning / LoRA

Knowledge Problem?
→ RAG

Reasoning Problem?
→ Agents

Integration Problem?
→ MCP

Governance Problem?
→ Guardrails

---

# Updated Current Weak Areas

- Cosine Similarity
- Metadata Filtering
- Hybrid Search
- Re-ranking
- Enterprise RAG Architecture
- Agents
- MCP
- AI Governance
- AI Security

---

# Next Topic

RAG Architecture End-to-End

Topics:
- Query Embeddings
- Cosine Similarity
- Similarity Search
- Metadata Filtering
- Hybrid Search
- Re-ranking
- Prompt Augmentation
- Enterprise RAG Patterns
- RAG Trade-offs




No.

The file I generated contains the **new Week 2 learning content**, but it does **not yet include the larger structural recommendation** I mentioned afterwards.

Specifically, it does **not** include:

```text
Foundation Layer
Adaptation Layer
Knowledge Layer
Reasoning Layer
Integration Layer
Governance Layer
Platform Layer
```

nor does it reorganize the existing content into that structure.

### Current State (v3)

Contains:

✅ Original Foundation Topics
✅ Week 2 Adaptation Topics
✅ AI Architecture Patterns
✅ Updated Weak Areas
✅ Next Topic (RAG)

---

### Recommended State (v4)

I would restructure the entire document into:

```text
# AI Solution Architect Learning Context

## Progress Dashboard

Foundation Layer
Adaptation Layer
Knowledge Layer
Reasoning Layer
Integration Layer
Governance Layer
Platform Layer

## Foundation Layer
- Tokens
- Embeddings
- Attention
- Transformers
- Weights
- Parameters
- Context Windows

## Adaptation Layer
- Prompt Engineering
- ICL
- Fine-Tuning
- Instruction Fine-Tuning
- LoRA

## Knowledge Layer
- RAG
- Vector Databases
- Hybrid Search
- Re-ranking
- Knowledge Graph RAG

## Reasoning Layer
- Tool Calling
- Agents
- Multi-Agent Systems

## Integration Layer
- MCP
- Workflow Orchestration

## Governance Layer
- Alignment
- Guardrails
- AI Governance
- AI Security

## Platform Layer
- Azure AI Foundry
- Amazon Bedrock
```

This becomes a **living handbook context file** rather than a collection of session notes.

For your long-term goal of becoming an AI Solution Architect, I would strongly recommend creating **v4** because once we start RAG, Agents, MCP, Governance, and Security, the current structure will become increasingly difficult to navigate.

My recommendation would be:

```text
v3 = Session Notes

v4 = Master Learning Context
```

and then we keep updating v4 after every major learning session.
