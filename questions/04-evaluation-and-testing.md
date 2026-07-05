# 4. Evaluation & Testing

Arguably the most important skill separating a hobbyist from a professional prompt engineer: building rigorous, repeatable evaluation rather than eyeballing a few outputs.

---

### Q: How would you build an evaluation set to measure whether a prompt is performing well? 🟡

**Answer:**
1. **Define the task's success criteria precisely** — what does a "good" output actually look like? Decompose vague notions like "helpful" into concrete, checkable dimensions (accuracy, completeness, tone, format adherence).
2. **Curate a representative test set**, including: typical/happy-path examples, known edge cases, and adversarial/ambiguous examples — not just easy cases the prompt already handles well.
3. **Include real production examples** where possible (anonymized/sampled from logs) rather than only hypothetical test cases, since real user inputs surface failure modes synthetic examples miss.
4. **Establish ground truth or a rubric** for each example, ideally reviewed by more than one person to catch labeling disagreements.
5. **Keep the eval set versioned and separate from the examples used to iterate on the prompt itself**, to avoid overfitting the prompt to the specific test cases (the prompt-engineering equivalent of training/test leakage).

**Follow-ups:**
- How large should an eval set be before you trust the results it gives you?
- How do you keep an eval set up to date as the product and user behavior evolve?

---

### Q: How do you evaluate outputs when there's no single "correct" answer (e.g., open-ended generation, summarization, creative writing)? 🔴

**Answer:**
- **Decompose into checkable sub-criteria** rather than a single holistic "good/bad" score — e.g., for summarization: factual accuracy (no hallucinated claims), coverage (key points included), conciseness, and coherence, each scored separately.
- **Use human evaluation with a clear rubric**, ideally with multiple raters and a measured inter-rater agreement score, since a single rater's subjective judgment is noisy.
- **Use pairwise/comparative evaluation** (which of two outputs is better) rather than absolute scoring where possible — humans are generally more reliable at relative judgments than assigning absolute scores.
- **Use LLM-as-a-judge for scale**, but validate the judge model's scores against a sample of human judgments periodically to catch judge drift or bias, and be aware of known judge biases (e.g., preferring longer outputs, or outputs that match the judge's own stylistic preferences).
- **Track implicit production signals** (user edits, regenerations, thumbs-down rate, copy/use rate) as a complement to offline evaluation, since these reflect real-world usefulness that a rubric might miss.

**Follow-ups:**
- What are known failure modes of LLM-as-a-judge, and how do you mitigate them (e.g., position bias, verbosity bias)?

---

### Q: How do you set up regression testing for prompts so that changes don't silently break existing behavior? 🟡

**Answer:**
- **Maintain a fixed eval set with expected behavior/scores**, run automatically against every prompt change before it ships — treat this exactly like a CI test suite for code.
- **Track scores over time/versions**, not just pass/fail at a single point, so gradual quality drift is visible even if no single change crosses a hard failure threshold.
- **Separate "hard" regressions** (must never happen — e.g., safety violations, broken output format) from **"soft" regressions** (a slight dip in a subjective quality score that might be an acceptable tradeoff for a gain elsewhere).
- **Test against multiple model versions** if the underlying model is likely to be updated by the provider, since prompt behavior can shift even without any prompt changes on your end — this needs the same regression discipline as code changes.
- **Include the adversarial/edge case examples** in the regression suite, not just typical happy-path cases, since regressions often show up first in edge cases.

**Follow-ups:**
- A prompt change improves your eval set's average score but a specific important edge case now fails. Do you ship it?

---

### Q: What metrics would you use to evaluate a summarization prompt specifically? 🟡

**Answer:**
- **Factual consistency/faithfulness:** does the summary contain claims not supported by the source? This is usually the most important and hardest-to-automate dimension — often requires human review or a specialized faithfulness-checking model, since standard overlap metrics don't catch hallucination well.
- **Coverage/informativeness:** does the summary capture the key points a human would expect?
- **Conciseness:** is it appropriately short relative to the source and the use case?
- **Automatic overlap metrics** (ROUGE, BLEU) can provide a cheap, scalable signal but correlate imperfectly with human judgment of quality — useful for regression tracking, not as the sole quality bar.
- **Task-specific downstream metrics** where available — e.g., if summaries are used to make a decision, does a person who reads only the summary make the same decision as one who reads the full source?

**Follow-ups:**
- How would you specifically test for hallucinated content in generated summaries at scale?

---

### Q: How would you design an evaluation for a prompt used in a customer-facing chatbot, where quality is inherently subjective? 🟡

**Answer:**
- **Break "quality" into measurable dimensions**: task success (did it actually resolve the user's need?), tone/appropriateness, factual accuracy (especially for anything grounded in company policy/data), safety (no inappropriate content), and format/UX fit.
- **Use a mix of offline and online evaluation**: offline rubric-based scoring on a curated test set before launch, and online implicit/explicit signals (CSAT, escalation rate, thumbs-down rate, conversation abandonment) after launch.
- **Segment evaluation by intent/topic category**, since chatbot quality often varies significantly by conversation type — an aggregate score can hide poor performance in a specific, important category (e.g., refund requests).
- **Include multi-turn evaluation**, not just single-turn quality, since chatbot failures often emerge from context loss or contradiction across turns rather than any single bad response.

**Follow-ups:**
- How would you evaluate whether the chatbot handles conversation history correctly across many turns?

---

### Q: How do you avoid overfitting a prompt to your evaluation set? 🔴

**Why it's asked:** A senior-level concern — iterating a prompt purely to maximize a fixed eval score can produce a prompt that's brittle in the real world.

**Answer:**
- **Hold out a portion of the eval set** that's never used during iterative prompt development, only for a final check before shipping — analogous to a true test set in ML.
- **Periodically refresh the eval set** with new real-world examples so the prompt can't simply be tuned to memorized quirks of a static set.
- **Watch for prompts that become overly long and specific** with rules seemingly added just to fix individual eval failures — this is a sign of overfitting; a healthier fix usually addresses the underlying pattern, not the individual failing example.
- **Validate gains against live production signals**, not just the offline eval score, before fully trusting an improvement — an offline win that doesn't show up in real user outcomes is a red flag that the eval set doesn't represent reality well.

**Follow-ups:**
- Describe a time (real or hypothetical) where a prompt looked great on your eval set but failed in production. What would you investigate?
