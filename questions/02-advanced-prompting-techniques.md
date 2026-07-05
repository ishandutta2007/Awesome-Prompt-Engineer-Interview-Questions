# 2. Advanced Prompting Techniques

Techniques that go beyond basic instruction-following — reasoning strategies, decomposition, and agentic patterns.

---

### Q: Explain chain-of-thought (CoT) prompting. Why does it improve performance on certain tasks, and where does it not help? 🟡

**Answer:**
Chain-of-thought prompting asks the model to generate intermediate reasoning steps before producing a final answer (e.g., "think step by step"), rather than jumping straight to a conclusion.

It helps most on tasks requiring **multi-step reasoning** — arithmetic, logic puzzles, multi-hop question answering — because it gives the model "space" to work through intermediate sub-problems token by token, rather than needing to compute a complex answer in a single forward pass.

It helps less (or can even hurt) on:
- **Simple, single-step tasks** where reasoning just adds latency/cost without improving accuracy.
- **Tasks with strict output format requirements**, where free-form reasoning text can interfere with clean structured output unless the reasoning is separated from the final answer (e.g., using a scratchpad the user never sees).
- **Highly subjective/creative tasks**, where "reasoning" doesn't map cleanly onto a right-answer structure.

**Follow-ups:**
- How would you get the benefits of chain-of-thought while still returning clean structured output to the end user?
- What is "self-consistency" and how does it build on chain-of-thought?

---

### Q: What is self-consistency prompting, and when would you use it despite its added cost? 🟡

**Answer:**
Self-consistency samples multiple independent chain-of-thought reasoning paths for the same question (using non-zero temperature) and takes the majority/most common final answer across samples, rather than trusting a single generation.

It's useful when: the task has a genuinely correct answer (math, logic, factual reasoning) and the cost of being wrong outweighs the added compute cost of multiple samples — since single reasoning chains can go down an incorrect path early and compound the error, while majority voting across several independent attempts tends to cancel out these idiosyncratic errors.

Tradeoff: N times the cost and latency of a single generation, so it's best reserved for high-stakes or high-value queries rather than applied universally.

**Follow-ups:**
- How would you decide the right number of samples to use for self-consistency in a cost-sensitive production system?

---

### Q: What is the ReAct (Reason + Act) prompting pattern, and how does it differ from plain chain-of-thought? 🔴

**Answer:**
ReAct interleaves reasoning steps with concrete actions (typically tool calls, like a search query or calculator invocation) and observations from those actions, in a loop: **Thought → Action → Observation → Thought → ...** until a final answer is reached.

The key difference from plain CoT: CoT reasons entirely from the model's internal knowledge in one pass, while ReAct lets the model gather new, real-world information mid-reasoning and adjust its plan based on what it learns — making it foundational to how most agentic/tool-using systems work today.

**Follow-ups:**
- What failure modes are specific to ReAct-style agents (e.g., getting stuck in unproductive action loops), and how would you mitigate them?

---

### Q: What is prompt decomposition, and when would you break a complex task into multiple smaller prompts/calls instead of one large prompt? 🟡

**Answer:**
Prompt decomposition splits a complex task into a pipeline of smaller, more constrained sub-tasks, each handled by its own (often simpler) prompt/call, rather than asking one prompt to do everything at once.

Favor decomposition when:
- **The task has clearly separable stages** (e.g., extract → classify → generate) where each stage benefits from focused instructions and its own evaluation.
- **Different stages need different models** — e.g., a cheap model for a simple classification step, a stronger model only for the harder generation step, optimizing cost.
- **Debuggability matters** — a multi-step pipeline lets you isolate exactly which stage is failing, whereas a single mega-prompt's failures are hard to diagnose.
- **The single-prompt version shows the model "juggling too much"** — quality degrades as instruction complexity increases in one shot.

Tradeoff: more calls means more latency and cost, and errors can still compound across stages, so it's not automatically better — I'd validate the decomposed pipeline against the single-prompt baseline on real eval data rather than assume decomposition is always superior.

**Follow-ups:**
- How would you handle error propagation in a multi-step prompt pipeline (e.g., stage 2 fails because stage 1's output was malformed)?

---

### Q: What is meta-prompting or prompt-to-prompt generation (using an LLM to write or optimize prompts)? When is it useful? 🔴

**Answer:**
Meta-prompting uses an LLM to generate, critique, or iteratively refine prompts — for example, asking a model to propose several prompt variants for a task, or to critique why a given prompt produced a bad output and suggest a fix.

Useful when:
- **Rapidly generating a first draft** of a prompt for a new task, which a human then refines rather than starting from a blank page.
- **Automated prompt optimization loops**, where a model iterates on a prompt against an eval set and proposes improvements based on failure analysis — useful at scale when manually iterating dozens of prompt variants isn't practical.
- **Explaining unexpected model behavior** — using a model to help diagnose why a specific prompt produced a specific unwanted output.

Caution: meta-prompting outputs still need human review and evaluation against real data — an LLM's confident suggestion for "a better prompt" isn't guaranteed to actually improve real-world performance without testing.

**Follow-ups:**
- How would you validate that a meta-prompt-optimized prompt is actually better, not just different?

---

### Q: What's the difference between in-context learning and fine-tuning from a prompt engineer's perspective, and how do you decide which approach to recommend? 🟡

**Answer:**
In-context learning (via prompting, including few-shot examples) teaches the model a task at inference time through the prompt itself, with no changes to model weights. Fine-tuning updates the model's weights based on a training dataset, baking the behavior in more permanently.

As a prompt engineer, I'd recommend starting with prompting/in-context learning first because it's faster to iterate, requires no training infrastructure, and can be updated instantly. I'd escalate to recommending fine-tuning only when: the task requires behavior that's hard to specify reliably through instructions (deeply stylistic or highly specialized domain behavior), the prompt has grown so large with examples that it's hurting cost/latency at scale, or the task needs to work with zero (or near-zero) prompt overhead in the production system.

**Follow-ups:**
- Give an example where you initially thought fine-tuning was necessary but solved the problem with prompting instead (or vice versa).

---

### Q: Explain "tree of thoughts" or other search-based prompting strategies, and what problem they solve that basic chain-of-thought doesn't. 🔴

**Answer:**
Tree-of-thoughts generalizes chain-of-thought by exploring multiple reasoning branches at each step (rather than one linear chain), evaluating intermediate states, and backtracking or pruning unpromising paths — similar to a search algorithm over possible reasoning trajectories.

It solves the problem that plain CoT commits to a single reasoning path early and can't recover if that path turns out to be wrong, while tree-based search allows exploring alternatives and selecting the most promising one based on some evaluation criterion at each branch point.

Tradeoff: significantly higher compute/latency cost since it explores multiple paths, so it's reserved for high-value, complex reasoning tasks (e.g., difficult planning or puzzle-solving) rather than everyday production use cases.

**Follow-ups:**
- Would you use tree-of-thoughts in a latency-sensitive, real-time production feature? Why or why not?
