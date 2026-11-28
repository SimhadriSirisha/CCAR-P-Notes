#### RAG Pipeline Stages, Search Types, and Chunking Trade-offs

RAG Pipeline Stages and Core Terms

- A RAG pipeline answers questions from a document collection in 7 stages: ingest -> chunk -> embed -> index -> retrieve -> optionally rerank -> generate.
- Each stage has its own failure mode.
- Chunk: A passage cut from a document; it is the exact unit that retrieval finds.
- Embed: Converting a chunk into a list of numbers that represents its meaning.
- Embedding search: Compares chunks by meaning.
- Keyword search (such as BM25): Matches exact tokens.
- Rerank: Reorders the retrieved chunks so the most relevant ones appear first.

The Chunking Trade-off
- Chunks too small: Lose necessary context. Splitting a table from its caption or a clause from its heading makes the passage unfindable or misleading.
- Chunks too large: Lower relevance and waste tokens on unneeded text. A whole-document chunk matches everything weakly.

Choosing a Chunking Strategy
- Structure-aware chunking (with modest overlap): Respects sections, headings, and tables, while using a small overlap to protect against losing meaning at chunk boundaries. Beats fixed-size splitting when document structure is reliable (for example, supplier contracts with consistent headings where each clause stays with its heading).
- Fixed-size splitting with overlap: Often the default choice in production for real-world documents that lack reliable structure (such as plain text files or PDFs without clear headings).

---
#### Contextual Retrieval

Contextual Retrieval (Fixing Lost Chunk Context)
- Root cause of many retrieval failures: Context is lost at the chunk boundary when a passage is cut away from its parent document or section.
- How it works: Before embedding and indexing, prepend a short, chunk-specific explanation to the start of each chunk that places it inside its document (such as naming the specific procedure).
- Core rule: Restore each chunk's lost context before you index it.
- Benchmark results: Anthropic's benchmark shows contextual retrieval measurably reduces retrieval failures, and the gains grow even larger when combined with contextual BM25 and reranking.

---

#### Hybrid Indexing
- Embedding search (Semantic similarity): Great for meaning-based questions (for example, "why does the pump alarm sound at night"), but often misses exact strings.
- Keyword search / BM25 (Exact token match): Great for exact strings like part numbers, IDs, or codes.
- Hybrid indexing (Embeddings + Keyword): Required for any corpus containing identifiers, part numbers, error codes, or clause references.
- Exam trap: Pure embedding search on an identifier-heavy corpus returns passages that are thematically similar but wrong, and no chunk tuning fixes this.
- How Hybrid Search Works at Retrieval Time ?
    - At Indexing Time (Hybrid Indexing): Every chunk is stored in two ways: as a vector embedding (for meaning) and in a BM25 keyword index (for exact tokens).
    - At Retrieval Time (Both Run Every Time): When a user asks a question, the retrieval stage sends that exact same query to both search engines at the same time:
        - Embedding search returns its top chunks based on semantic meaning.
        - BM25 keyword search returns its top chunks based on exact token matches.
    - Merging the Results: The system combines both lists into one unified candidate set (using rank fusion or weighted scoring).
        - If a query is purely conceptual, embedding results naturally rise to the top of the merged list.
        - If a query is an exact code, BM25 results naturally rise to the top.
        - If a query has both meaning and a code, chunks found by both methods get boosted to the very top.
    - Optional Reranking: A reranker then takes that merged shortlist and reorders the chunks so the single most relevant passages go to the LLM first.

    **EXAMPLE**

    Scenario: A Telecoms Company's Technical Support RAG Assistant

    A technician asks a single mixed question: "Why does router FH-900B drop Wi-Fi every evening with error ERR-4092?"
    
    - Step 1 (Both Searches Run Together):
        - Embedding search searches by meaning and returns Chunk A (about "nightly wireless disconnects caused by automatic channel scans") and Chunk C (about general router overheating).
        - BM25 keyword search searches by exact tokens and returns Chunk B (which explicitly mentions FH-900B and ERR-4092) and Chunk D (a parts list mentioning FH-900B).
    - Step 2 (Merge and Rerank): The pipeline merges both result sets into one shortlist (Chunks A, B, C, and D) and passes them to the reranker, which pushes the exact troubleshooting match for ERR-4092 on FH-900B and evening disconnects to the top.
    - Why "Only One" Would Have Failed: If the system had only run embedding search, it might have missed the exact ERR-4092 fix. If it had only run BM25, it might have missed Chunk A because the manual wrote "nightly wireless disconnects" instead of the technician's exact words "drop Wi-Fi every evening." Running both on every query solves both halves at once.

---

#### Metadata Filters
- Use stored document facts (such as date, source, or product) to narrow the pool of candidate chunks before similarity ranking scores them
- Thinking "Before" and "After" Give the Same Result: It is easy to assume that retrieving the top 10 chunks first and throwing away the wrong ones afterward gives you the same answer. It does not. Filtering after ranking can leave you with almost zero usable chunks because wrong or outdated documents hogged all the top spots.
- Confusing Metadata Filtering With Reranking:
    - Metadata filtering happens before similarity ranking to eliminate ineligible documents from the candidate pool.
    - Reranking happens after retrieval to reorder the surviving chunks from most relevant to least relevant.
- Why Filtering BEFORE Ranking Works (Pre-Filtering):
    - First, the metadata filter throws out every document that does not match the required facts (creating a smaller, strictly valid candidate pool).
    - Next, similarity ranking runs only on that valid pool.
    - Result: Every single slot in your top-K results (for example, top 5 chunks) is guaranteed to be a valid, in-scope chunk.
- Why Filtering AFTER Ranking Fails (Post-Filtering):
    - First, similarity ranking searches the entire database and grabs the top 5 matches overall.
    - Outdated documents or wrong products often have very similar wording and take up 4 or 5 of those top slots.
    - Next, the metadata filter deletes those invalid chunks afterward.
    - Result: You are left with only 1 chunk (or zero chunks) to send to the LLM, even though dozens of great, valid chunks existed further down the list.

---
#### Complete End-to-End RAG Pipeline Flow (All 7 Stages + Enhancements)

Phase A: Offline Pipeline (Preparing the Documents - Stages 1 to 4)
- Ingest: Pull in the raw documents (PDFs, policies, manuals, contracts) and extract both their text and their metadata (stored facts like date, source, or product).
- Chunk: Cut each document into retrievable passages.
    - Structured docs (clear headings, sections, tables): Use structure-aware chunking with modest overlap.
    - Unstructured docs (plain text, messy PDFs): Use fixed-size splitting with overlap (the production default).
    - Enhancement - Contextual Retrieval (runs right here, before embedding and indexing): Prepend a short, chunk-specific explanation to the start of each chunk that places it in its document so context is not lost at chunk boundaries.
- Embed: Turn each contextualized chunk into a list of numbers (vectors) that represents its semantic meaning.
- Index: Save the processed chunks into the search database using Hybrid Indexing:
    - Store the embeddings (for meaning-based search).
    - Store the BM25 keyword index (for exact token matching, built on the contextualized chunks).
    - Store the metadata tags alongside each chunk so they can be filtered later.

Phase B: Online Pipeline (Answering a User Question - Stages 5 to 7)
- Retrieve: When a user asks a question, find the best candidate chunks in two internal steps:
    - Sub-step A - Metadata Filter: Use stored facts (date, source, product) to narrow the candidate pool before any similarity ranking runs.
    - Sub-step B - Hybrid Search: Run both embedding search (semantic meaning) and BM25 keyword search (exact tokens like part numbers or error codes) on the filtered pool, then merge the two result lists together.
- Optionally Rerank: Take the merged shortlist of retrieved chunks and reorder them so the single most relevant chunks sit at the very top.
- Generate: Send the user's question plus the top reranked chunks to the LLM so it can generate a grounded final answer.

---

#### RAG Operational Freshness

4 Data-System Operations Required for RAG
- Atomic refresh: Any document update must trigger re-embed and re-index atomically (all or nothing). Skipping this causes the index to silently serve outdated content with full confidence.
- Pin the embedding-model version everywhere: An embedding model turns text into coordinates (lists of numbers) on a specific map. If your documents were turned into numbers using Model v1, and your live user queries are turned into numbers using Model v2, the two maps do not match. The system throws no errors and looks healthy, yet retrieval silently breaks and pulls irrelevant chunks. You must lock (pin) the exact same embedding-model version for both indexing and querying.
- Monitor retrieval quality (Recall): Recall is the percentage of truly relevant passages that your retriever actually manages to find and return. By running a labelled query set (a set of test questions where you already know which exact chunks should be returned), you can measure recall continuously. If a bad update breaks the index, recall drops and triggers an automated alert before end users ever discover the bad answers.
- Evaluate retrieval separately from generation: If you only test the final answer (end-to-end), you cannot tell whether the retriever fetched the wrong chunks or the LLM misread the right chunks. Debugging end-to-end is slow and expensive. Measuring retrieval quality and generation quality separately isolates any failure to one exact stage immediately.

The Pattern After a Document Refresh
- Symptom: Quality breaks immediately after a document refresh, while model and latency remain unchanged.
- First suspect: The retrieval/indexing stage is serving stale or irrelevant chunks.
- Wrong exam options: Model upgrades, temperature changes, and context window sizes.

When RAG Is Not the Answer
- Retrieval is not the default.
- If a corpus is small and stable enough to fit in a cached prompt, serve it whole: it is faster, cheaper, and has no retrieval failure.

