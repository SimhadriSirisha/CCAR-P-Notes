### Service Levels, Envelopes, and the Six Elements of a Defensible Commitment

- Service Level: A formal promise about how fast, available, or accurate a system is.
- Envelope: The actual performance that the real pipeline measurably achieves on the real corpus (the actual documents and data the system uses).
- Clients love to demand a service level (like "under 2 seconds and 99.9% uptime") before anything is built. The golden rule here is `engineer first, then promise. Never promise first.` You must measure the envelope before committing to a number.

- Six Elements That Make a Commitment Defensible (Checkable and Actionable):
    - The Metric: Exactly what is being measured and how it is calculated.
    - The Percentile or Threshold: Never rely on the mean (average), because nobody experiences the mean. Use a percentile (like p95) or a hard threshold.
    - The Measurement Point: Where the clock starts and stops. This separates a narrow promise about the LLM model from a real promise about what the user experiences (e.g., starting the clock when the customer sends a message).
    - The Window: The time period over which compliance is judged. A good window prevents one bad hour from breaching an entire year, and prevents one good week from hiding a bad month.
    - The Population: Exactly who and what traffic/requests the commitment covers. Must be defined honestly before measurement, never adjusted afterward.
    - The Breach Behaviour: What concretely happens when the commitment is missed.

**EXAMPLE**

Scenario: A Telecoms Company Committing to a Service Level for Its Claude Billing-Support Assistant

First, the architect follows engineer first, then promise. They run the full RAG + Claude pipeline against 5,000 real historical billing chats and real tariff PDFs (the real corpus) to measure the true performance envelope. Only after seeing real numbers do they write a defensible 6-part commitment:

- The Metric: End-to-end response latency (total elapsed seconds) and billing-answer accuracy (exact match on tariff/fee rules).
- The Percentile or Threshold: p95 latency under 4.5 seconds (meaning 95% of replies finish within 4.5 seconds, instead of hiding slow outliers inside a "2.1-second average"), plus a quality floor of 93% accuracy and a 99.5% safety rate on a named held-out set of 600 real billing chats.
- The Measurement Point: The latency clock starts the millisecond the customer hits "Send" in the mobile app and stops when the full formatted reply renders on their screen. This measures the user experience (including RAG retrieval, billing API calls, and guardrails), not just the raw Claude API call.
- The Window: Evaluated over a rolling 30-day calendar month. One 15-minute cloud outage does not blow up an annual contract, and one quiet weekend cannot mask three bad weeks.
- The Population: Covers all authenticated postpaid mobile customers asking English-language billing questions (defined upfront before testing, excluding unauthenticated browsing and enterprise accounts).
- The Breach Behaviour: If p95 latency exceeds 4.5 seconds for 15 consecutive minutes, or monthly accuracy drops below the 93% quality floor on weekly audits, the system automatically routes new chats to live human agents, pages the on-call engineer, and pauses automated replies until the root cause is fixed.

---

### Managing Dependencies, Impossible Latency Asks, Segment Quality, Feedback, and Scope Changes

**Example Scenario consider for this sections is "A Retail Bank Running a Claude Loan-Support Assistant"

#### Dependencies Limit Your Promises
- Check the underlying platform's guarantees before writing availability or tail-latency clauses.
- Anthropic's standard tier is best-effort availability.
- A prioritised tier(premium API service) has an explicit uptime target, requires a committed capacity arrangement (no longer generally purchasable), and any excess traffic spills onto the standard tier(pay-as-you-go).
- Never promise an uptime or latency envelope higher than the platform underneath you.

    **EXAMPLE**

    The bank's sponsor asks for a 99.99% uptime SLA. The architect checks the dependency: the bank runs on Anthropic's standard tier (best-effort availability). The architect explains they cannot legally promise 99.99% uptime above a best-effort dependency and instead designs a fallback to human agents when the API is unavailable.

#### Negotiating an Impossible Latency Ask
- Show the latency decomposition (where every millisecond goes) so the achievable number is clearly backed by data, not arbitrary.
- State the real envelope the pipeline achieves.
- Improve perceived responsiveness instead of faking speed:
    - Anthropic separates baseline latency (total time) from time to first token (TTFT).
    - Use streaming, progressive disclosure (showing results in stages), partial results, or an asynchronous path with notification.
    - Remember: none of these techniques reduce total completion time; they only make the app feel more responsive.
- Never accept an impossible number, and never delete retrieval or verification steps to meet a speed demand (that turns a measured problem into an unmeasured problem).

    **EXAMPLE**

    The sponsor demands full answers in under 2 seconds. The architect shows the latency decomposition: 1.2s for policy RAG search + 2.5s for Claude generation + 0.8s for compliance verification = 4.5 seconds total.
    - The architect refuses to delete the RAG lookup or compliance check (which would make answers fast but silently wrong).
    - Instead, they state the real 4.5-second envelope and add streaming and progressive disclosure (showing "Checking mortgage policy..." at 0.5s and streaming words at 1.5s) to improve perceived responsiveness.

#### Quality Commitments and the Carve-Out Trap
- Accuracy commitments require the exact same six elements as latency (vague statements like "very accurate" fail).
- If failures cluster in a specific segment (like clinic orders vs. individual customers), state it openly: commit on the measured population, and give the weak segment its own improvement plan, evidence, and owner.
- The Trap That Looks Like Rigour: Narrowing the contract population to make the main SLA number look good leaves excluded cases live in production with no threshold, no review path, and no accountable owner. Only narrow the population when the exclusion is visible and nobody is left uncovered by accident.

    **EXAMPLE**
    
    Testing shows 95% accuracy on standard personal loans, but only 79% accuracy on complex commercial business loans. Instead of secretly hiding a carve-out in the contract while leaving commercial loans live and unmonitored, the architect makes the exclusion visible: the SLA covers personal loans, commercial loans are routed straight to human specialists for now, and commercial loans get their own improvement plan and named owner.

#### Feedback Is Instrumentation
- A feedback channel must produce two things: evidence (a trace, case ID, or reproducible input) and a disposition (a decision on what happens to the report).
- Triage must show a visible outcome; otherwise users stop reporting, and silence is mistaken for quality.
- Route confirmed failures into Domain 4's regression set.
- Never turn raw user anecdotes directly into prompt edits (fixing one reported case silently breaks unmentioned cases).

    **EXAMPLE**

    A loan officer complains, "The bot messes up interest rates sometimes." The architect adds a "Flag Error" button that automatically attaches the exact trace and case ID (evidence). When confirmed, the triage team logs a visible disposition ("Confirmed bug, ticket #412, added to test suite") so staff keep reporting instead of going silent. The bug is added to Domain 4's regression set before any prompt change is made, ensuring the fix does not break other loan calculations.

#### Reporting and Scope Discipline
- Status reports must start with the success criteria agreed at discovery, supported by engineering measurements.
- For any scope change: weigh it against agreed criteria and baseline, show the sponsor its cost, date, and risk before committing, and reset the plan with the sponsor's signature.

    **EXAMPLE**

    Reporting and Scope Change: Weekly reports open with the discovery success criteria (accuracy floor, safety rate, latency). Mid-project, a manager asks to add automated tax-return parsing. Before writing any code, the architect weighs it against the baseline, presents the extra $45,000 cost, 4-week delay, and higher error risk to the sponsor, and only proceeds after the sponsor signs off on the reset plan.