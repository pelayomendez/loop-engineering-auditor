# Further reading — peer-reviewed grounding for the rubric

The Loop Engineering working note (`../README.md`) is a practitioner synthesis. The claims it
makes about *why* a loop fails — especially the generator/evaluator separation, the limits of
self-correction, and the human-oversight costs — are each backed by published research. This
file maps the framework's load-bearing claims to that literature so an auditor can cite a
mechanism, not just an opinion, and so reviewers can check the basis for each scoring rule.

Each entry: the framework claim it supports → the source(s). All links verified June 2026.

## 1. "An agent grading its own work tends to praise it" (the Nodding loop; Verification)

The single most important scoring rule — do not credit Verification to a self-grading agent —
rests on documented **self-preference / self-enhancement bias** in LLM judges.

- Zheng et al., *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*, NeurIPS 2023 — names and measures **self-enhancement bias** (judges favor their own style), alongside position and verbosity biases. The original LLM-as-a-judge reference. https://arxiv.org/abs/2306.05685
- Wataoka et al., *Self-Preference Bias in LLM-as-a-Judge*, 2024 — gives a quantitative metric; finds GPT-4 shows significant self-preference, correlated with lower perplexity (the judge prefers text *familiar* to it — exactly the "chain of self-persuasion" the note describes). https://arxiv.org/abs/2410.21819
- Wataoka et al. / follow-ups, *Do LLM Evaluators Prefer Themselves for a Reason?*, 2025 — separates harmful bias from legitimate preference; flags the risk specifically for self-improvement cycles (i.e. loops). https://arxiv.org/html/2504.03846v3

## 2. "Making the generator self-critical works poorly; tune a separate skeptic" (Verification)

- Huang et al. (Google DeepMind), *Large Language Models Cannot Self-Correct Reasoning Yet*, ICLR 2024 — without external feedback, intrinsic self-correction does not reliably help and can *degrade* performance. Direct support for "don't fix a modest author; install an independent check." https://arxiv.org/abs/2310.01798
- Madaan et al., *Self-Refine: Iterative Refinement with Self-Feedback*, NeurIPS 2023 — self-feedback helps on some open-ended tasks but uses one model as generator+critic; read alongside Huang et al. as the boundary of what self-review can and cannot catch. https://arxiv.org/abs/2303.17651

## 3. "Use a separate, adversarial agent — borrowed from GANs" (Sub-agents; the generator/evaluator split)

- Goodfellow et al., *Generative Adversarial Networks*, 2014 — the generator-vs-discriminator min-max game the note explicitly ports to "an agent that writes and an agent that reviews." https://arxiv.org/abs/1406.2661
- Du et al., *Improving Factuality and Reasoning in Language Models through Multiagent Debate*, ICML 2024 — multiple independent model instances debating reduces hallucination and improves factuality vs. a single self-judging pass; evidence that independence (not a bigger model) is what buys reliability. https://arxiv.org/abs/2305.14325

## 4. "The evaluator should act, not just read" (Verification by acting)

- Yao et al., *ReAct: Synergizing Reasoning and Acting in Language Models*, ICLR 2023 — interleaving reasoning with tool actions reduces hallucination vs. reasoning alone; grounds the rule that an evaluator wired to *run tests / click the page* beats one that only reads diffs. https://arxiv.org/abs/2210.03629

## 5. "Memory must live outside the conversation" (Persistence; the Amnesiac loop)

- Shinn et al., *Reflexion: Language Agents with Verbal Reinforcement Learning*, NeurIPS 2023 — agents that write reflections to an **episodic memory buffer** and read them back on the next trial improve over time; the mechanism behind "the agent forgets, the repo does not." https://arxiv.org/abs/2303.11366

## 6. "Keep a human capable of saying no" (Cognitive surrender; the human checkpoint)

- Bai et al. (Anthropic), *Constitutional AI: Harmlessness from AI Feedback*, 2022 — even heavily AI-driven training keeps human judgment in the loop via an explicit set of principles; a model for "the human sets the boundary, the loop executes within it." https://arxiv.org/abs/2212.08073
- Parasuraman & Manzey, *Complacency and Bias in Human Use of Automation: An Attentional Integration*, Human Factors, 2010 — the human-factors basis for cognitive surrender: operators under-monitor reliable automation and make commission errors by following wrong recommendations. https://www.researchgate.net/publication/47792928
- Goddard et al., *Automation bias: a systematic review of frequency, effect mediators, and mitigators*, JAMIA, 2012 — automation bias affects experts and novices alike and resists training; argues the guard must be **structural** (a built-in checkpoint), echoing the note's "build the pause in." https://pmc.ncbi.nlm.nih.gov/articles/PMC3240751/

## 7. "Real-world autonomous coding is hard; reliability comes from the scaffold" (whole-loop realism)

- Jimenez et al., *SWE-bench: Can Language Models Resolve Real-World GitHub Issues?*, ICLR 2024 — real GitHub-issue resolution requires multi-file coordination, execution, and long context; early solve rates were low, underlining that an unscaffolded model is not enough and that the constraints (Stripe's lesson) carry the reliability. https://arxiv.org/abs/2310.06770

---

## How to use these in an audit

When you flag a gap, cite the mechanism, not just the rule. Examples:

- Flagging a Nodding loop: "The same agent writes and grades. Self-enhancement bias (Zheng et al. 2023; Wataoka et al. 2024) means it will systematically over-rate its own output, so 'never said no across N runs' is evidence of *no real check*, not of quality."
- Recommending an independent evaluator over a self-critique prompt: "Intrinsic self-correction does not reliably improve reasoning and can degrade it (Huang et al. 2024) — tune a separate skeptical evaluator rather than asking the generator to be harder on itself."
- Justifying the permanent human checkpoint: "Automation bias is robust to training and affects experts (Goddard et al. 2012; Parasuraman & Manzey 2010), so the guard against cognitive surrender must be structural — a checkpoint the loop cannot route around."

Keep the practitioner framing primary (it's what the user's loop is built on); use these as the
*why* when a builder pushes back on a finding.
