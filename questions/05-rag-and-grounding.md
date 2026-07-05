# 5. Retrieval-Augmented Generation (RAG)

Grounding model outputs in external, retrieved data is one of the most common real-world prompt engineering tasks.

---

### Q: Explain how a RAG pipeline works end to end, and where prompt engineering fits into it. 🟡

**Answer:**
A typical RAG pipeline: (1) a user query is embedded into a vector; (2) that vector is used to search a vector database/index for the most semantically similar document chunks; (3) the retrieved chunks are inserted into the prompt as context; (4) the LLM generates a response grounded in that context, ideally citing which chunk(s) it used.

Prompt engineering touches multiple stages:
- **The retrieval query itself** may need to be rewritten/expanded from the raw user input to retrieve better results (query rewriting).
- **The generation prompt** must clearly instruct the model how to use the retrieved context (e.g., "only answer using the provided documents; say you don't know if the answer isn't in them").
- **Handling retrieval failure gracefully** — instructing the model what to do when no relevant chunk was retrieved, rather than letting it fall back on ungrounded (and potentially hallucinated) knowledge.

**Follow-ups:**
- What would you check first if a RAG system starts producing plausible-sounding but incorrect answers?

---

### Q: How would you prompt a model to avoid hallucinating when using retrieved context, and how reliable is this in practice? 🟡

**Answer:**
Prompting techniques that help:
- **Explicit grounding instructions:** "Answer only using the information in the provided documents. If the answer isn't present, say you don't know" — reduces but doesn't eliminate hallucination.
- **Requiring citations/quotes:** asking the model to reference which specific chunk supports each claim makes fabrication more visible and checkable (and can be validated programmatically against the source chunks).
- **Explicitly separating context from instructions** with clear delimiters, so the model doesn't confuse retrieved text as containing new instructions to follow (this also matters for prompt injection defense — see section 6).

Reliability caveat: these techniques meaningfully reduce hallucination rate but don't eliminate it — a rigorous system still needs evaluation (checking generated claims against source documents, e.g., via a faithfulness-checking step) and, for high-stakes use cases, human review, rather than trusting grounding instructions alone.

**Follow-ups:**
- How would you programmatically verify that a model's citations actually support its claims?

---

### Q: How does chunking strategy (chunk size, overlap) affect prompt and retrieval quality? 🟡

**Answer:**
- **Chunk size:** too small, and chunks lose context needed to answer questions correctly (a fact might be split across chunk boundaries); too large, and irrelevant content dilutes the retrieved context, wasting tokens and potentially confusing the model about what's actually relevant to the query.
- **Overlap between chunks:** helps prevent important information from being awkwardly split at a chunk boundary, at the cost of some redundancy in the index.
- **Semantic/structure-aware chunking** (splitting along natural document boundaries like sections or paragraphs, rather than a fixed token count) usually retrieves more coherent, useful context than naive fixed-length splitting.

As a prompt engineer, I'd treat chunking strategy as something to tune empirically against the evaluation set (measuring downstream answer quality, not just retrieval relevance in isolation), since the "right" chunk size is task- and document-type-specific.

**Follow-ups:**
- How would you handle a document type (e.g., a large table or code file) that doesn't chunk well using standard paragraph-based splitting?

---

### Q: How would you prompt-engineer around the case where multiple retrieved chunks contain conflicting information? 🔴

**Answer:**
- **Instruct the model explicitly on how to handle conflicts** — e.g., prioritize the most recent document (if timestamps are available and included in context), prioritize a designated authoritative source, or explicitly surface the conflict to the user rather than silently picking one side.
- **Include metadata in the context** (source, date, authority level) so the model has the information needed to make this judgment, rather than expecting it to resolve ambiguity with no signal.
- **Consider resolving conflicts at the retrieval/data layer** rather than relying purely on prompting — e.g., deduplicating or flagging outdated documents in the underlying knowledge base, since prompting is a weaker lever than fixing the underlying data quality issue.

**Follow-ups:**
- Should the model ever be allowed to silently pick one source over another without telling the user? When might that be acceptable?

---

### Q: How do you decide how many retrieved chunks (top-k) to include in the prompt? 🟡

**Answer:**
Tradeoff between:
- **Recall (more chunks increases the chance the truly relevant information is included)** vs. **precision/attention dilution (too many chunks, especially irrelevant ones, can bury the useful information and cause the "lost in the middle" issue, or lead the model to synthesize from irrelevant material)**.
- **Cost and latency**, since each chunk consumes context tokens.

I'd tune top-k empirically against the evaluation set, testing several values and measuring downstream answer quality (not just retrieval relevance scores), and would also consider a **re-ranking step** — retrieve a larger candidate set cheaply, then use a more precise re-ranking model to select the best few chunks to actually include in the generation prompt, rather than relying purely on the initial retrieval's ranking.

**Follow-ups:**
- What is re-ranking, and why might a system use both an initial retriever and a separate re-ranker?

---

### Q: A user asks a question, but the RAG system retrieves no relevant documents. How should the prompt handle this gracefully? 🟢

**Answer:**
The prompt should explicitly instruct the model on this exact scenario rather than leaving it to guess: e.g., "If none of the provided documents are relevant to the user's question, tell the user you don't have information on this topic rather than attempting to answer from general knowledge." This prevents the model from silently falling back on potentially outdated or hallucinated general knowledge when the whole point of the system is to answer from a specific, controlled knowledge base.

I'd also design the retrieval step to expose a relevance/confidence signal (e.g., a similarity score threshold) so the generation prompt can be conditioned on "no relevant context found" explicitly, rather than passing empty or weakly-relevant chunks silently.

**Follow-ups:**
- How would you test that this "no relevant context" fallback actually triggers reliably, rather than the model still attempting a guess?
