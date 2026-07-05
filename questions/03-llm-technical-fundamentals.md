# 3. LLM Technical Fundamentals

You don't need to train models, but you need to understand the mechanics that directly shape prompting decisions.

---

### Q: What is tokenization, and why does it matter for prompt engineering? 🟢

**Answer:**
Tokenization breaks text into sub-word units (tokens) that the model actually processes — not necessarily whole words. A word can be one token, multiple tokens, or share a token with part of another word, depending on the tokenizer's vocabulary.

This matters for prompt engineering because:
- **Cost and context budget** are measured in tokens, not characters or words — the same instruction can cost meaningfully different amounts depending on phrasing and language.
- **Certain tasks are harder for models due to tokenization quirks** — e.g., character-level tasks like counting letters in a word, or precise arithmetic on numbers, can fail because numbers/characters aren't tokenized the way a human would naturally decompose them.
- **Non-English languages** often tokenize less efficiently (more tokens per unit of meaning), affecting both cost and sometimes quality.

**Follow-ups:**
- Why might an LLM struggle to reliably count the number of a specific letter in a word, and how would you prompt around that limitation?

---

### Q: What is a context window, and how does it constrain prompt design in practice? 🟢

**Answer:**
The context window is the maximum number of tokens (input + output combined, depending on the model) the model can process in a single call. Everything — system prompt, conversation history, retrieved documents, user input, and the generated response — must fit within it.

Practical implications:
- **Long conversations need a memory strategy** (summarization of older turns, selective retrieval of relevant history) once they exceed the window.
- **RAG systems must prioritize which retrieved chunks to include**, since you usually can't fit "everything potentially relevant."
- **Cost scales with tokens used**, so a bigger context window isn't free — stuffing more context than necessary "just in case" wastes money and can also dilute the model's attention on the truly relevant parts (sometimes called the "lost in the middle" problem, where models attend less reliably to information buried in the middle of a very long context).

**Follow-ups:**
- What is the "lost in the middle" phenomenon, and how would you structure a long prompt to mitigate it?

---

### Q: What are the key sampling parameters (temperature, top-p, top-k) and how do they interact? 🟡

**Answer:**
- **Temperature** rescales the probability distribution over next tokens before sampling — lower values sharpen the distribution toward the most likely tokens (more deterministic), higher values flatten it (more random/diverse).
- **Top-k** restricts sampling to only the k most probable next tokens, discarding the long tail regardless of their probability mass.
- **Top-p (nucleus sampling)** restricts sampling to the smallest set of tokens whose cumulative probability exceeds p, which adapts dynamically to how "peaked" or "flat" the distribution is at each step (unlike a fixed top-k cutoff).

These interact: e.g., a high temperature with a low top-p can still stay fairly focused because top-p prunes the extreme tail even as temperature flattens the remaining distribution. In practice, most production systems tune one or two of these (commonly temperature and top-p together) rather than all three, and I'd tune them empirically against a task-specific eval set rather than by theory alone.

**Follow-ups:**
- If a model's outputs are technically correct but feel repetitive/monotonous, which parameters would you adjust first?

---

### Q: What is the difference between the model's "knowledge cutoff" and its ability to use information provided in the prompt? Why does this distinction matter for prompt design? 🟢

**Answer:**
The knowledge cutoff refers to the latest data the model was trained on — anything after that date, the model has no inherent knowledge of unless it's provided in the prompt itself (e.g., via retrieval/RAG or directly pasted context). Within a single conversation, though, the model can use any information given to it in-context regardless of when that information originated, because it's processing that text at inference time, not relying on memorized training data.

This matters because prompt engineers often need to explicitly ground time-sensitive or post-cutoff information in the prompt rather than assuming the model "knows" it, and should design prompts that make clear which information source (training knowledge vs. provided context) should take priority when they might conflict.

**Follow-ups:**
- How would you prompt a model to prioritize provided context over its own potentially outdated training knowledge?

---

### Q: What is an attention mechanism, at a level useful for a prompt engineer (not a researcher) to reason about prompt structure? 🟡

**Answer:**
At a practical level: transformer-based LLMs use attention to weigh how much each token in the input should influence the representation of every other token, allowing the model to relate distant parts of a long prompt to each other rather than only processing text strictly left-to-right in isolation.

For a prompt engineer, the useful takeaway isn't the math — it's the practical implication: **placement and repetition of key information affects how strongly the model "attends" to it.** Important instructions placed at the very beginning or very end of a long prompt tend to be attended to more reliably than instructions buried in the middle (see the "lost in the middle" phenomenon), which is why critical constraints are often restated near the end of a long prompt just before the actual task/question.

**Follow-ups:**
- How would you test whether a specific instruction is being reliably "attended to" versus effectively ignored in a long prompt?

---

### Q: What's the difference between an open-weight model and a closed/API-only model, and how does that affect prompt engineering work? 🟡

**Answer:**
Open-weight models release their trained parameters publicly, allowing self-hosting, direct inspection, and fine-tuning without vendor dependency. Closed/API-only models are accessed only through a hosted API, with the underlying weights and often much of the training/architecture details undisclosed.

For prompt engineering specifically:
- **Closed models** are typically stronger at the frontier but introduce risk of silent behavior changes when the vendor updates the model version — prompt engineers must maintain regression eval suites to catch these shifts.
- **Open-weight models** give more control (you can inspect tokenization, log-probabilities, sometimes even attention patterns) which can aid debugging, and you can pin an exact model version indefinitely.
- Prompting techniques often transfer reasonably well across models but not perfectly — a prompt tuned carefully for one model's quirks may need re-tuning when switching model families, which is a real operational cost prompt engineers need to budget for.

**Follow-ups:**
- How would you design your prompt library to minimize the cost of migrating between model providers or versions?

---

### Q: What are logprobs (log probabilities), and how can a prompt engineer use them practically? 🔴

**Answer:**
Logprobs are the log-probability the model assigned to each token it generated (or to specific candidate tokens), which some APIs expose alongside the generated text.

Practical uses:
- **Confidence estimation:** low logprob on a generated answer can be a proxy signal for model uncertainty, useful for triggering fallback behavior (e.g., escalate to human review) without needing a separate classifier.
- **Debugging classification-style prompts:** comparing the logprob assigned to each candidate label directly, rather than relying only on the generated text, can give a more precise and stable signal than free-text generation, especially for close calls.
- **A/B testing prompt variants:** comparing how confidently a model produces a target completion across prompt versions.

Caveat: logprob-based confidence doesn't necessarily correlate with actual factual correctness — a model can be highly "confident" (high logprob) while still being wrong, so this signal should be validated against ground truth, not trusted blindly.

**Follow-ups:**
- How would you use logprobs to build a lightweight, cheap classifier without a separate model, using just prompting?
