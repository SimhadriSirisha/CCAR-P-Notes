Every integration has two numbers: a latency requirement, such as a p95 response time, and a quality requirement.

#### Latency Budgets, Serial Dependencies, and Optimizing the Dominant Term

Latency Is a Sum

* Total request latency equals the sum of every stage along the request path:
* Model calls: Time-to-first-token (wait before the first token arrives) plus generation length, including any extended thinking.
* Retrieval and reranking: Every search and reranking step.
* Tool round-trips: Every external API or tool call.
* Agent-loop iterations: Every cycle the agent runs through.

Serial vs. Parallel Execution

* In a serial tool chain, latencies add together because each call must wait for the previous one to finish.
* Only genuinely independent calls can run in parallel to save time.
* Any step that needs the output of an earlier step keeps its serial latency, regardless of how it is scheduled.

Diagnose Before You Optimize

* Use traces (objective 3.4) to break total latency into its individual terms and locate the dominant term.
* Cutting a small, non-dominant term is a bad trade: you pay a cost (such as lower accuracy) and get no noticeable improvement in total speed.

Anthropic's Ordered Latency Options

1. Choose the right model tier for the task.
2. Reduce input and output tokens.
3. Use streaming for interactive user interfaces.

**EXAMPLE**

Scenario: A Health Insurer's Claim Status AI Assistant

A policyholder asks an insurer's chat agent, "Why was my MRI claim denied, and what is my remaining deductible?" The response takes 11 seconds total, which feels slow to the member.

* Checking the Serial Dependency: The agent must first run a retrieval step to find the member's `claim_id` (0.5s), and then call the `get_denial_reason(claim_id)` tool (0.7s). Because the tool needs the `claim_id` from retrieval, those two steps cannot run in parallel and stay serial (1.2s combined). However, fetching the member's remaining deductible (`get_deductible_balance`) only needs the logged-in user ID, so it is genuinely independent and can run in parallel with the claim lookup.
* Using Traces to Find the Dominant Term: Instead of guessing, the engineering team inspects the traces and sees the breakdown:
* Retrieval + reranking: 0.5s
* Tool round-trips: 0.7s
* Heavy frontier model with extended thinking generating a long 800-token explanation: 9.8s (the dominant term)


Wrong Fix vs. Right Fix:
* Wrong Fix: Removing the reranking step to save 0.2s. Total time drops from 11.0s to 10.8s (invisible to the user), while retrieval accuracy drops.
* Right Fix (Following Anthropic's Order): Target the 9.8s dominant term by (1) stepping down to a faster model tier without unnecessary extended thinking for routine lookups, (2) trimming the prompt and capping output tokens to a concise 150-token answer, and (3) enabling streaming in the chat UI so the member starts reading within a second.

---

#### Accuracy vs. Latency Trade-Offs, Selective Spending, and Model-Tier Validation

The Core Tension

* Every accuracy control adds extra work, which always adds latency.
* The five common accuracy controls are:
    * More retrieved context
    * A reranking stage
    * A verification pass
    * More reasoning
    * A bigger model

* Every latency cut carries the risk of lowering accuracy.

Spend Selectively (Tiered Routing)

* Apply expensive accuracy controls only where a wrong answer would be costly.
* Routing high-value or high-risk requests through a verified path and routine requests through a faster path routinely beats both all-or-nothing approaches:
* Verifying everything: Wastes cost and adds latency across all traffic.
* Removing verification everywhere: Violates or deletes a named quality requirement.

Validating a Smaller Model

* Switching to a smaller, faster model without checking accuracy is the same Domain 2 mistake showing up at the integration layer.
* A smaller model is only a valid latency fix after you re-measure its accuracy against your quality threshold.
* Before re-measuring, its error rate is unknown, so the switch is just a guess.

---

#### Streaming and Prefix Caching

**Streaming**

* How it works: Returns tokens to the user one by one as the model generates them instead of waiting for the full reply to finish.
* What it improves: Improves perceived latency because users see progress immediately.
* What it does NOT change: Total generation time remains unchanged.
* When to use it: Best suited for interactive interfaces (such as a live customer support chat).
* When NOT to use it: Irrelevant for a machine consumer (such as a nightly batch job or automated parser) that must wait for the final token and parse the complete output before taking action.

**Prefix Caching**

* Why it is special: A rare optimization that delivers two benefits at once on repeat requests: lower cost and faster time-to-first-token (TTFT).
    * Every LLM call has two separate phases: reading the input and writing the output. Prefix caching only saves work during the reading phase.
        * Phase 1: Reading the input (Prefill -> determines TTFT)

            Before the model can generate its very first word, it must read and process all your input tokens (system prompt, tool definitions, reference docs, and user question). The time this takes is your Time-to-First-Token (TTFT). Prefix caching saves the model's processed state for the stable input prefix. On repeat requests, the model skips re-reading those thousands of tokens and jumps straight to producing the first token much faster.
        * Phase 2: Writing the output (Decode -> determines generation time)

            Once the first token is out, the model generates the rest of the response one token at a time. Because the answer to each user's question is new, caching the input cannot pre-compute the output. The model still writes each new output token at its normal speed.
* What goes in the cached prefix (Stable Content): Content that stays identical across requests, such as:
    * System prompt
    * Tool definitions
    * Static reference corpus (for example, company policies or delivery terms)

* What goes AFTER the prefix (Volatile Content): Content that changes from request to request, such as:
    * Retrieved customer or shipment records
    * The user's current question

* Core rule: Never put volatile content in the cached prefix, because changing the prefix on every request prevents caching from working.

---

#### Exam Strategy for Latency vs. Quality Trade-Off Questions

The 3-Step Elimination Method
- Identify both requirements: Pinpoint the stated latency requirement (such as an SLA or p95 target) and the stated quality requirement (such as an error-rate threshold).
- Eliminate one-sided options: Cross out every choice that sacrifices a named requirement just to improve the other.
- Target the bottleneck: Among the surviving choices, pick the one that addresses the dominant budget term.

How to Spot Wrong Answers Quickly
- One-sided justifications: Any option that only solves for speed or only solves for accuracy when both are named is wrong.
- Deleting critical controls: The right answer never deletes a control that the use case names as critical (like removing verification everywhere to hit an SLA).
- Over-applying controls blindly: Applying an expensive control everywhere (like verifying every single request) and missing the SLA is also wrong.
- Streaming for machine consumers: If the consumer is a pipeline, parser, or batch job, any option proposing streaming is automatically wrong.

What the Right Answer Looks Like
- Meets and names both goals: Its justification explicitly addresses both sides (for example, "Meets the stated p95 while holding the measured wrong-shipment rate").
- Restructures the design instead of cutting corners: Valid design fixes include:
    - Routing by value or risk (sending high-stakes items through heavy verification and routine items through a fast path).
    - Caching the stable prefix (caching the unchanging start of the prompt).
    - Parallelising independent calls (running non-dependent tool or retrieval calls at the same time).