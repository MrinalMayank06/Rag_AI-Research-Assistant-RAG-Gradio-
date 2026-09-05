# RAG --- Retrieval-Augmented Generation

> **How RAG actually works: from raw documents to vectors, retrieval,
> context, and LLM generation.**

RAG (Retrieval-Augmented Generation) is an architecture that combines
**information retrieval** with **large language models (LLMs)**.

Instead of expecting an LLM to know every piece of private,
domain-specific, or frequently changing information, RAG retrieves
relevant information from an external knowledge base and provides it to
the LLM as context.

------------------------------------------------------------------------

## 🧠 The Core Idea

A simple RAG system can be understood as:

``` text
Documents
    ↓
Text Extraction & Cleaning
    ↓
Chunking
    ↓
Embeddings
    ↓
Vector Database / Index
    ↓
        ┌─────────────────────┐
User → Query Embedding → Similarity Search
        └──────────┬──────────┘
                   ↓
              Top-K Chunks
                   ↓
            Context Formation
                   ↓
                  LLM
                   ↓
             Final Answer
```

The LLM is only one component.

The real challenge is retrieving the **right information at the right
time**.

------------------------------------------------------------------------

# 🔄 RAG Workflow

## 1. Data Ingestion

RAG starts with information from different sources:

-   PDF / DOCX files
-   Web pages
-   Databases
-   APIs
-   Knowledge bases
-   Emails and notes
-   Internal company documents

The first step is converting these sources into usable text.

``` text
Raw Data → Extraction → Clean Text
```

------------------------------------------------------------------------

## 2. Data Processing

Extracted information is cleaned and prepared.

Typical operations include:

-   Text extraction
-   OCR for scanned documents
-   Cleaning unwanted characters
-   Normalization
-   Metadata extraction
-   Language detection

Good preprocessing directly affects retrieval quality.

------------------------------------------------------------------------

## 3. Chunking

Large documents are divided into smaller pieces called **chunks**.

Example:

``` text
100-page document
       ↓
 ┌───────────────┐
 │    Chunk 1    │
 ├───────────────┤
 │    Chunk 2    │
 ├───────────────┤
 │    Chunk 3    │
 ├───────────────┤
 │      ...      │
 └───────────────┘
```

Why chunk?

Sending an entire document for every query is inefficient and can
introduce irrelevant information.

Common strategies:

-   Fixed-size chunking
-   Sliding-window chunking
-   Sentence-based chunking
-   Semantic chunking
-   Parent-child retrieval

### The trade-off

**Chunks too small:** context can lose meaning.

**Chunks too large:** retrieval becomes less precise and more irrelevant
information may enter the context.

------------------------------------------------------------------------

# 🔢 4. Embeddings

An embedding model converts text into a numerical vector representing
its semantic characteristics.

Example:

``` text
"RAG retrieves relevant information"

              ↓

[0.21, -0.42, 0.87, 0.13, ...]
```

A query is embedded using the same or compatible embedding space.

The goal is that semantically related text has vectors that are close
according to the chosen similarity metric.

This enables **semantic search** instead of relying only on exact
keyword matching.

------------------------------------------------------------------------

# 🗄️ 5. Vector Database / Vector Index

After embedding the chunks, the vectors need to be stored and searched
efficiently.

Examples:

-   Pinecone
-   Qdrant
-   Weaviate
-   Milvus
-   Chroma
-   FAISS

A stored record can conceptually contain:

``` text
Vector
+
Original Text
+
Document ID
+
Page Number
+
Metadata
```

The vector index makes it practical to search a large collection of
embeddings.

------------------------------------------------------------------------

# 🔍 6. Query Processing

When a user asks:

``` text
"How does RAG reduce hallucinations?"
```

the query goes through processing and is converted into an embedding.

``` text
User Query
    ↓
Query Processing
    ↓
Query Embedding
    ↓
[0.19, -0.39, 0.91, ...]
```

Now the system has a query vector that can be compared against stored
document vectors.

------------------------------------------------------------------------

# 📐 7. Similarity Search

The retrieval system compares the query vector with document vectors.

One commonly used metric is **Cosine Similarity**.

Conceptually:

``` text
             Document Vector
                    ↗
                   /
                  /   small angle
                 /
                ↗
          Query Vector
```

Cosine similarity is:

``` text
cos(θ) = (A · B) / (||A|| ||B||)
```

The important intuition:

**Smaller angle → greater directional similarity**

For normalized vectors, cosine similarity is closely related to the dot
product.

Example:

``` text
Chunk A → 0.94
Chunk B → 0.87
Chunk C → 0.76
Chunk D → 0.31
```

The system can rank chunks according to their similarity to the query.

------------------------------------------------------------------------

# 🎯 8. Top-K Retrieval

A RAG system normally does not send millions of chunks to the LLM.

Instead, it retrieves the most relevant `K` chunks.

Example:

``` text
Top-K = 5

Chunk 1 → 0.94
Chunk 2 → 0.91
Chunk 3 → 0.88
Chunk 4 → 0.84
Chunk 5 → 0.81
```

These chunks become the candidate context for generation.

------------------------------------------------------------------------

# 🧩 9. Context Formation

The retrieved chunks are combined with the user's query.

Conceptually:

``` text
Instructions
+
Retrieved Context
+
User Question
        ↓
      LLM
```

The quality of this context is critical.

If retrieval returns irrelevant information, the LLM receives poor
evidence.

That leads to a fundamental RAG principle:

> **Bad retrieval → Bad context → Bad answer**

------------------------------------------------------------------------

# 🤖 10. Generation

The LLM receives the query along with the retrieved context and
generates the response.

``` text
Retrieved Context
        +
    User Query
        ↓
       LLM
        ↓
   Final Answer
```

The LLM can now answer using information retrieved from the external
knowledge source.

This makes RAG especially useful for:

-   Private knowledge bases
-   Enterprise search
-   Documentation assistants
-   Research assistants
-   Customer support
-   Frequently changing information
-   Domain-specific applications

------------------------------------------------------------------------

# 🔁 Advanced RAG

Basic RAG is only the beginning.

Production systems often introduce additional retrieval and generation
techniques.

### Retrieval

-   Dense retrieval
-   Sparse retrieval
-   Hybrid search
-   Metadata filtering
-   Query expansion
-   Multi-query retrieval
-   Re-ranking

### Chunking

-   Semantic chunking
-   Parent-child retrieval
-   Context-aware chunking

### Context

-   Context compression
-   Duplicate removal
-   Context prioritization
-   Citation/source tracking

### Generation

-   Grounded generation
-   Structured output
-   Citation generation
-   Prompt optimization

### Evaluation

A production RAG system should measure more than whether an answer
"looks good".

Important questions include:

-   Did we retrieve the correct information?
-   Was the retrieved context relevant?
-   Did the answer use the retrieved evidence?
-   Was the final answer factually correct?
-   Did the system introduce unsupported information?

------------------------------------------------------------------------

# ⚠️ Common RAG Failure Points

RAG can fail even when the LLM itself is powerful.

### Poor chunking

Important information may be separated across chunks.

### Weak embeddings

The embedding model may not represent domain-specific meaning
effectively.

### Bad retrieval

The correct document exists but isn't retrieved.

### Too much context

Irrelevant chunks can distract the model.

### Too little context

The retrieved chunks may not contain enough information to answer the
question.

### Metadata problems

Incorrect or missing metadata can make filtering unreliable.

### Generation errors

The LLM may still produce unsupported claims even with retrieved
context.

------------------------------------------------------------------------

# 🏗️ Production Mental Model

Think of RAG as two major pipelines.

## Offline Pipeline

``` text
Documents
    ↓
Extraction
    ↓
Cleaning
    ↓
Chunking
    ↓
Embedding
    ↓
Vector Index / Database
```

This prepares the knowledge base.

## Online Pipeline

``` text
User Query
    ↓
Query Processing
    ↓
Query Embedding
    ↓
Retrieval
    ↓
Top-K Results
    ↓
Context Formation
    ↓
LLM
    ↓
Answer
```

This happens when the user asks a question.

------------------------------------------------------------------------

# 🛠️ Example Technology Stack

A RAG application can combine tools such as:

  Layer                Examples
  -------------------- ----------------------------------------------------
  Document ingestion   Unstructured, Apache Tika, BeautifulSoup
  Embeddings           OpenAI Embeddings, Hugging Face models, Instructor
  Vector search        FAISS, Pinecone, Qdrant, Weaviate, Chroma
  LLM                  GPT, Llama, Mistral, Gemini
  Orchestration        LangChain, LlamaIndex
  Storage              PostgreSQL, S3, Blob Storage, document databases
  API                  FastAPI, Flask, Node.js
  Evaluation           Retrieval and generation metrics, human evaluation

The exact stack depends on the use case, data volume, latency
requirements, cost, and infrastructure.

------------------------------------------------------------------------

# 📊 RAG in One Sentence

> **RAG retrieves relevant external knowledge and gives it to an LLM as
> context so the model can generate a more grounded response.**

Or, even simpler:

``` text
Find the right information
          ↓
Give it to the model
          ↓
Generate the answer
```

------------------------------------------------------------------------

# 💡 Key Takeaway

RAG is not just:

``` text
Vector DB + LLM
```

It is an information pipeline.

The quality of the final answer depends heavily on what happens **before
the LLM sees the prompt**.

If you want better RAG systems, don't only ask:

> "Which LLM should I use?"

Also ask:

> **"How accurately can I retrieve the information the LLM needs?"**

That is where the real engineering begins.

------------------------------------------------------------------------

## 📌 Related Concepts to Learn Next

If you're going deeper into RAG, study these in order:

1.  Embeddings
2.  Vector indexing
3.  Cosine similarity
4.  Approximate Nearest Neighbor (ANN) search
5.  Chunking strategies
6.  Hybrid retrieval
7.  Re-ranking
8.  Metadata filtering
9.  Context compression
10. RAG evaluation
11. Agentic RAG
12. Graph RAG

------------------------------------------------------------------------

## 👨‍💻 Author

**Mrinal Mayank**

Computer Science & Engineering \| AI / GenAI Enthusiast

Focused on understanding AI systems from the **engineering layer up**
--- not just using APIs, but understanding what happens underneath.

------------------------------------------------------------------------

## ⭐ If this helped

Star the repository, share the article, and keep exploring.

**Don't just build RAG. Understand RAG.**
