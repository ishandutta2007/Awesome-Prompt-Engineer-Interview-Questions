# 1. Prompting Fundamentals

Core techniques every prompt engineer must have fluent, hands-on command of.

---

### Q: What's the difference between zero-shot, one-shot, and few-shot prompting? When would you choose each? 🟢

**Why it's asked:** Baseline vocabulary check, but also tests whether you know *when* to reach for each — not just definitions.

**Answer:**
- **Zero-shot:** the model is given only an instruction, no examples. Works well for tasks the model has strong general capability in (e.g., "summarize this paragraph").
- **One-shot:** a single example is provided to anchor format or style.
- **Few-shot:** multiple examples (typically 2-10) are provided to establish a pattern — especially useful for tasks with a specific, non-obvious output format, tone, or edge-case handling the model wouldn't infer from an instruction alone.

Choice depends on: task ambiguity (more examples reduce ambiguity), output format complexity (structured/unusual formats benefit from examples), and token budget (each example costs context and money at scale). I'd start zero-shot, and only add examples once I see the model's zero-shot failures — adding few-shot examples "just in case" wastes tokens and can even hurt performance if examples are unrepresentative.

**Follow-ups:**
- What's the risk of using too many or poorly chosen few-shot examples?
- How would you select which examples to include in a few-shot prompt?

---

### Q: How do you decide what should go in a system prompt versus a user prompt? 🟢

**Answer:**
- **System prompt:** persistent instructions that define the assistant's role, tone, constraints, and behavior boundaries across the entire interaction — things that shouldn't change per-request (persona, safety guardrails, output format rules, refusal policies).
- **User prompt:** the specific, per-request task or question — the variable part of the interaction.

Practically: anything that should apply uniformly across many different user inputs belongs in the system prompt; anything that varies per request belongs in the user prompt (or is templated in). I'd also keep the system prompt focused and avoid overloading it with so many rules that the model starts ignoring or conflating them — clarity and priority-ordering of instructions matters more than exhaustiveness.

**Follow-ups:**
- What happens if a user's prompt directly contradicts an instruction in the system prompt? How should the system be designed to handle that?

---

### Q: What are common techniques for controlling the output format of an LLM response (e.g., getting valid JSON reliably)? 🟡

**Answer:**
- **Explicit schema instructions:** describe the exact structure expected (field names, types, nesting) rather than a vague "return JSON."
- **Few-shot examples of the exact desired format**, including edge cases (empty lists, optional fields).
- **Structured output / JSON mode features** offered by many model APIs, which constrain generation to valid JSON at the decoding level — more reliable than prompting alone.
- **Delimiters and explicit start/end markers** (e.g., asking the model to wrap output in specific tags) to make parsing more robust even if surrounding text is generated.
- **Post-hoc validation and retry logic:** validate the output against the schema programmatically, and re-prompt (with the validation error included) if it fails, rather than assuming one-shot success.
- **Lower temperature** for tasks requiring strict format adherence, since format-breaking is often correlated with higher-randomness sampling.

**Follow-ups:**
- What would you do if structured output constraints are supported by the model API, but you also need creative/varied text within one of the JSON fields?

---

### Q: How do you write a prompt that needs to work reliably across a wide range of unpredictable user inputs (not just the happy path)? 🟡

**Answer:**
- **Explicitly enumerate edge cases and expected behavior for them** in the prompt (empty input, off-topic input, ambiguous input, adversarial input) rather than only describing the happy path.
- **Test against a deliberately adversarial/edge-case eval set**, not just typical examples, before shipping (ties into section 4, Evaluation).
- **Give the model explicit fallback instructions** — e.g., "If the user's request is ambiguous, ask a clarifying question rather than guessing," so failure modes are graceful and predictable rather than silent guesses.
- **Constrain scope explicitly** — tell the model what it should *not* attempt to do, since models will often try to be maximally helpful even outside intended scope unless told otherwise.

**Follow-ups:**
- Give an example of an edge case that broke a prompt you wrote, and how you fixed it.

---

### Q: What is prompt "temperature," and how do you decide what value to use for a given task? 🟢

**Answer:**
Temperature controls the randomness of token sampling — lower values (near 0) make the model's output more deterministic and focused on the highest-probability tokens; higher values increase diversity and creativity at the cost of consistency and sometimes coherence.

Guidance:
- **Low temperature (0–0.3):** factual Q&A, classification, code generation, structured extraction — tasks where consistency and correctness matter more than variety.
- **Higher temperature (0.7–1.0):** creative writing, brainstorming, generating diverse options — tasks where variety is the goal.
- I'd tune temperature empirically against an eval set rather than picking a value theoretically, since the "right" value is task- and model-specific.

**Follow-ups:**
- What other sampling parameters (top-p, top-k, frequency/presence penalty) do you commonly adjust, and why?

---

### Q: How do you structure a long, complex prompt so it stays maintainable and debuggable? 🟡

**Answer:**
- **Use clear sections with headers or delimiters** (e.g., `# Role`, `# Task`, `# Constraints`, `# Output Format`, `# Examples`) so both the model and future editors can parse intent easily.
- **Keep instructions ordered by priority** — put the most important/non-negotiable constraints first or make them unambiguous, since models can deprioritize instructions buried deep in a long prompt.
- **Avoid redundant or conflicting instructions** that accumulate over iterative editing — periodically audit and prune a prompt rather than only ever adding to it.
- **Externalize variable content** (user data, retrieved context) into clearly marked, templated slots separate from static instructions, so the reusable "logic" of the prompt is easy to version and test independently of any specific input.
- **Version control prompts like code** — track changes, and evaluate each version against a regression test set before deploying (see section 4).

**Follow-ups:**
- How do you handle a prompt that's grown to include 15+ rules over several iterations and has started producing inconsistent behavior?

---

### Q: What's the difference between an instruction-based prompt and a role-based (persona) prompt, and when does persona-setting actually help? 🟢

**Answer:**
An instruction-based prompt directly tells the model what task to do ("Summarize this article in 3 bullet points"). A role/persona-based prompt frames the model as a character or expert ("You are a senior financial analyst reviewing this report") to shape tone, vocabulary, and implicit knowledge assumptions.

Persona-setting genuinely helps when: it shifts the *style and framing* of a response in a way that matters for the use case (e.g., "explain like I'm five" vs. "write as a legal expert"), or when it helps the model apply domain-appropriate conventions (a "senior code reviewer" persona nudging toward more rigorous code critique).

It's often overused as a superstition, though — persona alone doesn't reliably improve factual accuracy or reasoning quality, and shouldn't be relied on as a substitute for explicit task instructions or constraints.

**Follow-ups:**
- Have you seen a case where persona-setting actually hurt output quality? What happened?
