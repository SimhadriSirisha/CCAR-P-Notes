#### Production Tracing and Why Non-Determinism Breaks Traditional Debugging

Traces vs. Traditional Server Debugging
- Trace: A complete record of one request from start (input) to finish (response), including the full context of what happened along the way.
- Traditional server debugging habits fail on LLM systems because LLMs are non-deterministic: the exact same input can produce different outputs.
- A green uptime dashboard only proves the service is running; it tells you nothing about whether the model's answers are actually right.

Why Non-Determinism Breaks Anecdote Debugging
- Agents make dynamic decisions and can take different paths between runs even when given identical prompts.
- Anthropic production lesson: When multi-agent users saw agents failing to find obvious information, engineers could not diagnose the root cause until full production tracing was in place.
- A single anecdote proves very little, and re-running the prompt proves even less.

The Anti-Method vs. The Right Approach
- The Anti-Method (Avoid): Re-running a failing input to "see if it happens again."
- Without the original trace, neither reproducing the bug nor failing to reproduce it proves anything. Getting the right answer on a re-run teaches you nothing about the original failure.
- The Right Approach:
    - Reason about distributions of results across traffic rather than single anecdotes.
    - Capture and keep the full context of each request in a trace so any individual failure can be diagnosed and explained systematically.

---
#### Per-Request Trace Fields, Correlation IDs, and Trace-Based Diagnosis

What Every Request Trace Must Capture
- A failure you cannot trace is a failure you cannot diagnose.
- Record these 8 items for every request:
    - Prompt version
    - Model ID
    - Retrieved context
    - Tool calls and their results
    - Token usage
    - Cost
    - Latency
    - The response

Correlation IDs
- A correlation ID is a single unique identifier recorded by every stage for the same user request.
- It stitches together all stages of a multi-call pipeline (or all iterations of an agent loop) into one navigable record.
- Tracing each call in isolation across a multi-stage pipeline fails because you cannot connect a bad final output back to the exact upstream stage that broke it.
 
Using Traces to Diagnose Root Causes
- Use the trace to see what each stage actually received and returned, rather than guessing.
- If wrong outputs suddenly start after an upstream data change (such as a product-page refresh) while the model and latency have not changed, inspect what retrieval returned in the trace and start at retrieval, not at the model or temperature.

---
#### Two-Tier Observability at Scale (Metrics, Sampling, Retention, and Redaction)

Why Logging Everything Forever Fails
- Nobody can read a million traces manually.
- Storing every full trace forever fails in two ways:
    - Cost: Massive storage and processing bills at high volume.
    - Privacy: Full prompts and outputs capture sensitive user data.

The Two-Tier Architecture:
- Tier 1 (Cheap, 100% of traffic): Lightweight aggregate metrics recorded on every single request:
    - Latency percentiles (such as p50, p95)
    - Token spend
    - Error rates
    - Refusal rates (how often the model declines to answer)
    - Tool-failure rates
- Tier 2 (Expensive, selective traffic): Complete full payloads saved only for:
    - A sample of normal requests
    - Every detected failure (so you always have the full trace when something visibly breaks)

Upfront Architecture Decisions (Make Them at the Start)
- Do not wait until after launch to decide these three controls:
    - Sampling rates: What percentage of normal requests get full Tier 2 logging.
    - Retention windows: How long full traces are stored before automatic deletion. (eg: Tier 2 payloads after 30 days to reduce privacy risk and storage cost, while keeping cheap Tier 1 aggregate numbers for 12 months)
    - Redaction of sensitive fields: Masking or stripping personal data (such as customer addresses) before traces are written to storage.

- Cross-Domain Link: Domain 5 defines the compliance requirements these controls satisfy; this objective covers designing them into the architecture from day one.

---
#### Quality Instrumentation, Automated Production Evaluation, and Privacy-Safe Behavioural Monitoring

Why Infrastructure Metrics Are Not Enough
- Uptime, error rate, and latency can all be green while the AI confidently outputs rubbish.
- A hallucinating system still registers as "up."
- When user complaints happen alongside healthy service dashboards, quality instrumentation is missing.
- Exam trap: Adding more uptime or infrastructure alerts does not fix bad answers; it only adds more metrics that are already green.

Instrumenting Quality in Production
* Quality requires its own dedicated signal: automated evaluation on sampled production traffic.
- Two ways to score sampled answers:
    - Rubric: A written list of scoring criteria.
    - LLM judge: A separate model that scores the answer.
- Calibration (Domain 4 link): Check the automated scorer's results against Domain 4's formal eval set so you know the scorer is accurate.
- Operational value: Converts vague complaints ("it sometimes makes things up") into a measurable trend with automated alerts running continuously between formal eval runs.

Coexisting With Privacy Constraints
- Behavioural monitoring works at the pattern level (tracking score distributions, error trends, and behavioral shifts across traffic) without requiring anyone to read individual conversations.
- This allows teams to detect quality regressions while honoring strict data-privacy rules.

**EXAMPLE**

Scenario: A Retail Bank's Mortgage Advisory Chatbot

A retail bank runs an AI chatbot that answers customer questions about mortgage eligibility and early-repayment fees. One week, customers start complaining that the bot is inventing non-existent "zero-fee early payoff" rules.
- Green Dashboard, Bad Answers: The bank's infrastructure dashboard shows 99.99% uptime, 0.01% API errors, and a fast 1.5-second latency. Adding stricter server uptime alerts would be useless because the hallucinating bot is already "up" and green.
- Adding Quality Instrumentation: The engineering team adds an automated LLM judge paired with a written rubric (checking factual grounding against official mortgage terms) to score a 5% sample of live production chats. They calibrate this LLM judge against their Domain 4 formal eval set to verify its scores match human mortgage experts.
- Measurable Trends + Privacy-Safe Pattern Monitoring: Because mortgage chats contain private salaries and debt details, engineers are banned from reading individual customer transcripts. Instead, the behavioural monitoring dashboard tracks scores strictly at the pattern level. When the aggregate grounding score drops from 96% to 82% right after a policy update, an automated alert fires between formal eval runs, letting the team catch and fix the hallucination trend without ever reading a single customer's private conversation.

---
#### Detecting Drift Early With Cheap Signals and Tail-Percentile Alerts

Why Cheap Signals Catch Drift First
- Distribution shifts appear in cheap Tier 1 metrics(token spend, input length, refusal rate, tool-failure rate) before user quality complaints arrive because these metrics count every request (100% coverage).

    Think of "Distribution" as the Normal Spread: Normally, your traffic follows predictable ranges (for example, most inputs are 400 to 600 tokens, refusals happen 1% of the time, and tool failures happen 0.5% of the time). When those ranges visibly slide upward or downward across traffic, that is a distribution shift.

- Key cheap signals to monitor for drift:
    - Token spend per request slowly climbing.
    - Refusal rates rising.
    - Tool-failure rates rising.
    - Input lengths drifting (getting unusually longer or shorter).

- Three Common Causes in LLM Systems on which we can add alerts:
    - Changed user behaviour: Customers suddenly start asking new types of questions or pasting much longer text.
    - Broken document refresh: Updated knowledge-base files accidentally inject bloated, corrupted, or missing chunks into retrieval.
    - Prompt regression: A prompt edit accidentally makes the model write longer answers, refuse too often, or call tools incorrectly.

- Alert on Tail Percentiles, Not Averages
    - p99 definition: The value that only the highest 1% of requests exceed (meaning 99% of requests are at or below this number).
    - Why averages fail: A stable mean (average) can completely hide a doubled p99 because the 99% of fast requests mathematically drown out the slowest 1%.
        - If 99 banking chatbot requests take 1 second and the slowest 1 request takes 5 seconds, the average is 1.04s and the p99 is 5s.
        - If that slowest 1% doubles to 10s (p99 = 10s), the average barely moves to 1.09s, hiding a 2x slowdown behind a stable-looking mean.
    - Operational impact: The users sitting in that worst 1% experience the worst failures or delays and are the ones who file the support tickets.
