# Additional Resources

A short list of resources for going deeper on prompt engineering. (Not exhaustive — PRs welcome.)

## Foundational Reading
- Anthropic's prompt engineering documentation — practical, example-heavy guidance on structuring prompts, using XML tags, and chain-of-thought techniques.
- OpenAI's prompt engineering guide — covers similar fundamentals with a different provider's conventions and API features.
- *Prompt Engineering Guide* (promptingguide.ai) — a broad, well-organized reference covering most major techniques (CoT, ReAct, self-consistency, etc.) with links to original papers.

## Key Papers (read abstracts first, then dive into ones relevant to your target role)
- "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" (Wei et al.)
- "ReAct: Synergizing Reasoning and Acting in Language Models" (Yao et al.)
- "Self-Consistency Improves Chain of Thought Reasoning in Language Models" (Wang et al.)
- "Tree of Thoughts: Deliberate Problem Solving with Large Language Models" (Yao et al.)
- "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (Lewis et al.) — the original RAG paper.

## Practicing Technical Fundamentals
- Any major model provider's interactive API playground/console — the best way to build intuition is direct, hands-on experimentation with real prompts and parameters.
- Hugging Face's tokenizer playground tools — useful for building intuition about how different tokenizers split text.

## Evaluation & Tooling
- Familiarize yourself with at least one prompt evaluation/observability framework (many exist — LangSmith, Braintrust, and others) even if you don't use it day-to-day, since many interviews assume familiarity with the general category of tooling.

## Communities & Newsletters
- r/PromptEngineering and r/MachineLearning for discussion threads.
- Latent Space (newsletter/podcast) for staying current on applied LLM engineering trends.

## Staying Current
- Follow release notes and technical/system reports from major model providers (Anthropic, OpenAI, Google DeepMind, Meta) — prompting best practices shift with each new model generation, and interviewers often probe whether you're current on recent model capabilities.
- This field moves fast: techniques and best practices from even a year ago can be stale. Treat any single source (including this repo) as a snapshot, and cross-check against current provider documentation before an interview.
