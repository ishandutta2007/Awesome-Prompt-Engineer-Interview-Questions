# 8. Behavioral & Case Studies

Real-world scenarios and debugging exercises. Many prompt engineering interviews include a live exercise ("here's a broken prompt, fix it") — use these to practice narrating your reasoning aloud.

---

### Q: Tell me about a time a prompt you wrote worked perfectly in testing but failed in production. What happened? 🟡

**What a strong answer includes:**
- Honesty that this is extremely common in prompt engineering — interviewers are wary of candidates who claim everything they've shipped worked flawlessly on the first attempt.
- A specific root cause (distribution shift between test examples and real user inputs, an edge case not covered, a silent model version update, a context-window truncation issue).
- The diagnostic process used to identify the cause (log analysis, reproducing the failure, isolating variables).
- The fix, and — importantly — a systemic change made afterward (e.g., adding the failure pattern to a regression suite, expanding the eval set to include a previously-missed input category) so the same class of failure is caught earlier next time.

**Follow-ups:**
- How did you communicate this failure and fix to the rest of the team or to stakeholders?

---

### Q: Walk me through how you'd approach fixing this prompt: it's supposed to extract structured data from customer emails but sometimes returns malformed JSON or misses fields. 🟡

**Live exercise structure — a strong candidate would:**
1. **Ask to see actual failing examples** rather than guessing blindly — real failure patterns matter more than hypothetical fixes.
2. **Check whether the failures cluster around specific input types** (very long emails, emails with unusual formatting, ambiguous field values, multiple items per email).
3. **Consider structural fixes first**: does the API support structured output / JSON mode constraints, which would eliminate malformed JSON at the decoding level rather than relying purely on prompt wording?
4. **Review the field-extraction instructions for ambiguity** — are optional/missing fields handled explicitly (e.g., "use null if the field isn't present" rather than leaving the model to guess or omit unpredictably)?
5. **Add validation and retry logic downstream** — programmatically validate the JSON against a schema, and re-prompt with the specific validation error if it fails, rather than assuming single-shot perfection.
6. **Build a regression eval set from the failing examples** before considering the fix complete, to confirm the fix generalizes and doesn't just patch the one example shown.

**Follow-ups:**
- What would you do differently if this were a lower-stakes internal tool versus a customer-facing, revenue-affecting workflow?

---

### Q: Tell me about a time you had to push back on a stakeholder who wanted a prompt to do something you didn't think was achievable through prompting alone. 🟡

**What a strong answer includes:**
- A specific example distinguishing what prompting can and can't reliably fix (e.g., a stakeholder wanting "100% accuracy" or wanting the model to reliably do precise arithmetic/counting that prompting alone can't guarantee).
- How the pushback was communicated constructively — ideally by demonstrating the limitation with concrete failing examples rather than just asserting it, and by proposing an alternative (a tool-call/calculator integration, a different architecture, a human-in-the-loop step, or a recalibrated expectation).
- Evidence of collaborative problem-solving rather than just saying no.

**Follow-ups:**
- How do you build credibility with stakeholders so your technical judgment about prompting limitations is trusted?

---

### Q: Describe a time you had to significantly reduce the cost or latency of a prompt-driven feature while a stakeholder insisted quality couldn't drop at all. 🟡

**What a strong answer includes:**
- A structured approach to the cost/quality tradeoff (see section 7) — testing smaller models, trimming unnecessary prompt content, exploring caching — grounded in actual eval data rather than guesswork.
- Transparent communication with the stakeholder about what was tested and the actual quality impact measured (or lack thereof) — not just a unilateral decision.
- A specific outcome with numbers (e.g., "reduced cost by X% with a measured Y-point quality difference on our eval set, which we agreed was an acceptable tradeoff for this use case").

**Follow-ups:**
- What would you have done if the cost reduction genuinely required an unacceptable quality tradeoff?

---

### Q: You're given a chatbot transcript where a user was clearly frustrated and the model's responses seem repetitive/unhelpful across several turns. How would you diagnose and fix the underlying prompt? 🔴

**Live exercise structure — a strong candidate would:**
1. **Read the full transcript carefully**, not just the last failing turn — multi-turn failures often stem from something that went wrong several turns earlier (e.g., a misunderstanding that compounded, or the model losing track of information given earlier in the conversation).
2. **Check whether conversation history/context is being passed correctly** — a common root cause of "repetitive" responses is that the model isn't actually seeing its own prior responses or key facts established earlier.
3. **Look for whether the system prompt gives clear escalation instructions** — e.g., does it tell the model what to do if the user expresses frustration or repeats a question (offer a human handoff, acknowledge the repetition explicitly) rather than looping the same unhelpful response?
4. **Consider whether the underlying issue is even fixable by prompting** — sometimes repetitive unhelpfulness reflects a retrieval/tool failure (the model has nothing new/useful to say because retrieval keeps returning the same unhelpful result), which needs a fix outside the prompt itself.
5. **Propose a fix and describe how you'd validate it** — e.g., testing against a small set of similarly frustrated/repetitive conversation patterns, not just the one transcript shown.

**Follow-ups:**
- How would you build automated detection for "the user is getting frustrated" so this pattern can be caught and escalated in future conversations, not just fixed reactively for this one?

---

### Q: Tell me about a time you collaborated with an ML engineer or researcher on a task where the line between "prompt fix" and "model/fine-tuning fix" was unclear. 🔴

**What a strong answer includes:**
- A specific example showing genuine collaborative diagnosis (e.g., testing whether better prompting alone closed the gap before escalating to a fine-tuning conversation, since fine-tuning is a bigger investment).
- Evidence of understanding both sides of the tradeoff (prompting is fast/cheap/reversible; fine-tuning is slower/costlier but can bake in behavior more reliably and reduce per-request prompt overhead at scale).
- A clear articulation of how the decision was ultimately made and what data/evidence drove it, rather than an arbitrary call.

**Follow-ups:**
- In hindsight, do you think the decision (prompting vs. fine-tuning) was the right one? What would you check differently next time?
