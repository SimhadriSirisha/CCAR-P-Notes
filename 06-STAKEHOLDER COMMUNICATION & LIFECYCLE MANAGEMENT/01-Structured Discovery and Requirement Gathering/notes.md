### Core Concepts of Discovery

-  Engagement: A piece of work an architect performs for a client.
-  Discovery: The opening phase of an engagement. Its sole purpose is to uncover the real underlying problem before designing any solution.
- Request: What a stakeholder asks you to build or do (usually framed as a feature or solution).
- Requirement: The actual underlying problem hidden beneath the stakeholder's request.
- Baseline: The measured cost of that problem today (in money, hours, delays, or rework) before any new system is introduced. 
- Consequence of Error: What it costs the business when the AI/system produces a wrong output (financial loss, regulatory fines, safety risks, or customer churn).

    
**EXAMPLE**
    
A Health Insurance Company

- Engagement: A health insurer hires an AI architect to modernize its claims department.
- Discovery: Instead of immediately picking an LLM, the architect starts by interviewing staff and auditing workflows.
- Request: The VP of Operations (stakeholder) says, "Build us an LLM agent that automatically approves or denies medical claims."
- Requirement (the problem underneath): During discovery, the architect finds the real bottleneck isn't decision-makingit's that human adjusters spend 70% of their day manually searching through 80-page policy PDFs to check coverage rules.
- Baseline (measured cost of the problem): Manual PDF lookups currently take 20 minutes per claim across 15,000 claims a month, costing $200,000/month in staff time and causing a 6-day backlog.
- Consequence of Error (cost of a wrong output): If an automated LLM hallucinates and wrongly denies a covered surgery, the insurer faces regulatory fines, legal liability, and patient harm. If it wrongly approves an uncovered procedure, the insurer loses $15,000+ per claim.

Because the architect ran discovery first, they don't build the risky auto-denial bot the stakeholder requested. Instead, they build a policy-search assistant with citations for human adjusterssolving the actual requirement while keeping the consequence of error low.

---

### Separating Requests from Requirements & Avoiding Early Traps

- How stakeholders frame requests: Stakeholders usually ask for a specific solutiona named tool, artefact, or something a competitor built.
- How an architect treats that request: Treat the proposed solution as evidence (a clue to help you investigate), never as a substitute for the actual problem.

The Architect's First Act (4 Questions): Before designing or estimating anything, establish:
- What is broken?
- For whom is it broken?
- At what measured cost?
- What would count as fixed?

Two Early Moves That Fail:
- Building too early: Produces a flashy demo on clean sample data. This creates an anchoring trap where every later discussion starts from an unjustified design instead of the real problem.
- Pricing too early: Puts a price tag on unscoped work. This turns the architect's guesswork/uncertainty into a binding contract that locks someone else into the risk.

**EXAMPLE**

Scenario: A Telecoms Company

- The Request (Stakeholder's Solution): The Head of Billing says, "Build us a RAG chatbot that explains monthly phone bills to customers, just like Vodafone's."
- Treating it as Evidence (The 4 Questions): Instead of immediately building a RAG bot, the architect investigates:
    - What is broken? Customers are calling in angry about unexpected roaming charges.
    - For whom? Both confused customers and Tier-1 call center agents.
    - At what measured cost? 40,000 billing calls per month at $6/call ($240,000/month), with 25% escalated because agents can't read the legacy billing codes.
    - What counts as fixed? Cutting roaming-related escalations by 50% and reducing average call handling time by 2 minutes.
- How the Two Early Moves Would Have Failed:
    - Building too early: The architect whips up a RAG chatbot over 10 clean, hand-picked sample PDF bills. Leadership loves the demo, and now every meeting is about tweaking the chatbot's UIeven though real customer bills have messy legacy codes that the LLM misinterprets and hallucinates refund amounts for.
    - Pricing too early: The architect quotes "$150,000 and 8 weeks" during the first meeting before ever seeing the legacy billing database. Once the project starts, integrating the messy billing APIs takes 6 months and $400,000trapping the delivery team in a contract built on the architect's early guesswork.

---

### The Six Required Outputs of Discovery

Judge discovery by whether it produces these six outputsnot by how many meetings or interviews were held:
1. Business Problem with a Measured Baseline:
- Quantify the current pain (hours wasted, error rates, backlog, revenue at risk).
- Must be measured before anything changesonce the process changes, the baseline is gone forever, and every future claim of benefit depends on comparing against it.
2. The Real Workflow (Workarounds Included):
- Document how staff actually do the job today, including informal hacks, spreadsheets, and workaroundsnever rely on outdated official process diagrams.
3. The Exact System Output:
- Define precisely what the system must produce. This determines the type of task (e.g., classification, extraction, drafting) and immediately eliminates half of the possible AI architectures.
4. Consequence of Error & Owned Success Criteria:
- Document what a wrong output costs the business.
- Define clear success criteria that include a specific threshold (a number/bar to clear) and a named owner.
5. Data Audit & Lawful Basis:
- "Data" in Discovery means any data the system reads, retrieves, trains on, or is tested against
- Identify what data exists, its quality, who owns it, and whether there is a lawful basis (legal right) to use it for this AI use case.
- Raising legal/data obligations early in discovery is cheap; discovering them mid-build is costly.
6. Constraints & Accountable Decision-Maker:
- Capture explicit constraints: budget, latency, system integrations, and regulatory exposure.
- Uncover unvolunteered constraints, especially operational reality (e.g., the specific two people who will actually operate the system).
- Never assume the client gave you the full list of constraints. Always ask, "Who is actually going to run, monitor, and fix this system on Tuesday morning after I leave?" because their skills and available time strictly limit what architecture you can design.
- Assign a named owner to every requirementwithout one, you cannot make trade-offs later because no one can accept the consequence.

**EXAMPLE**

Scenario: A Retail Bank Automating Credit Card Fraud Disputes

Instead of bragging that they interviewed 25 bank employees, the AI architect finishes discovery with these six outputs:

- Measured Baseline: Analysts currently spend 18 minutes per dispute across 10,000 disputes/month, with a 12% error rate on chargeback codes and $1.2M/year in missed filing deadlines. (Measured before touching the process.)
- Real Workflow (with workarounds): The official 2021 diagram shows analysts using one banking portal. In reality, analysts copy-paste transaction logs into a personal Excel sheet and Slack a senior colleague to decode obscure merchant category codes.
- Required System Output: The system must output a structured JSON chargeback reason code + a 3-sentence evidence summary (not an open-ended customer chat). Knowing this is a structured extraction/classification task immediately rules out conversational chatbot architectures.
- Consequence of Error & Success Criteria: A wrong code causes the bank to automatically lose the dispute with Visa/Mastercard (average loss: $450/case). Success criteria: ≥94% code accuracy on historical cases, owned by the VP of Fraud Operations.
- Data & Lawful Basis: Data lives in the core ledger (clean) and scanned customer receipts (blurry, messy), owned by Card Services. Legal checks the lawful basis upfront to confirm whether customer transaction data can be sent to a hosted LLM under banking privacy lawscheap to resolve now before writing a line of code.
- Constraints & Named Owner: Budget is $250k, latency must be under 5 seconds, it must plug into the legacy mainframe, and it falls under strict banking regulations. Unvolunteered constraint: Only one IT admin and one fraud team lead will be available to monitor the system after launchmeaning the bank cannot operate a complex self-hosted model cluster. The VP of Fraud Operations is named the accountable owner so that if latency and accuracy conflict later, she has the authority to make the trade-off.

---

### Consequence of Error Sets the Bar & Consultation Is Not Structure

- Consequence of error sets the project's price: The cost of a wrong output dictates how heavy, complex, and expensive the entire rest of the engagement will be.

- Low vs. high stakes comparison:
    - Lower bar: An AI suggestion that a human reviews before acting (e.g., a draft email a handler checks).
    - Higher bar: An unchecked AI output flowing straight into a payment or regulatory filing.

- Four things Consequence of Error locks in:
    - The accuracy threshold worth committing to.
    - Whether a human checkpoint must be a fixed part of the design.
    - How much verification the pipeline needs.
    - The latency and cost envelope (the achievable range of speed and cost, which is driven by that verification).

- Feature list ≠ Discovery: Capturing a list of requested features without establishing the consequence of error produces only a scope, not true discovery.
- Consultation ≠ Structure: Workshops, surveys, and presentations can generate lots of noise while producing zero discovery outputs:
    - Stakeholder preferences ≠ a measured baseline.
    - A wish list ≠ consequence of error.
    - An enthusiastic group ≠ a named owner.

- The Exam Litmus Test: Whenever an option proposes heavy consultation, ask: "Which specific discovery artefact does this produce?" If none, reject it.

**EXAMPLE**

Scenario: A Health Insurance Company Using an LLM for Prior Authorizations
- How Consequence of Error Sets the Bar:
    - Design A (Human-reviewed): The LLM summarizes a doctor's clinical notes for a nurse to review before approving an MRI. If the summary misses a detail, the nurse catches it. Verification is light, so the latency and cost envelope stays low (3 seconds, $0.02/case).
    - Design B (Unchecked decision): The LLM automatically sends denial letters straight to patients and logs them for state regulators. One wrong denial triggers a lawsuit and regulatory fines. Now the pipeline needs multi-step verification, deterministic rule checks, secondary model auditing, and a mandatory human checkpoint for edge casespushing the latency and cost envelope way up (30+ seconds, $0.45/case) and tripling the project price.
- The Consultation Trap vs. What the Architect Does Instead:
    - What fails (Consultation without structure): A team runs 4 weeks of workshops and surveys with 40 doctors, nurses, and managers. Everyone is excited and produces a 30-page feature wish list ("dark mode," "voice input," "instant approvals"). They now have a scope, zero baseline, no named owner, and no agreed consequence of error.
    - What works (Artefact-driven discovery): The architect cancels the big workshops. Instead, they pull 90 days of claims audit logs (producing the measured baseline), shadow two nurses processing real cases (producing the real workflow), and meet 1-on-1 with the Chief Medical Officer to sign off on the regulatory cost of a wrong denial and name her the accountable owner.

---

### Pass/Fail Success Criteria (Anthropic's Standard)

- Four Qualities of Good Criteria: Every criterion must be Specific, Measurable, Achievable, and Relevant.
- Where Targets Come From: Base numerical targets on benchmarks, prior experiments, or expert knowledgenever on stakeholder optimism.
- Multiple Criteria Required: Most AI use cases cannot be judged by a single headline number; they require several criteria evaluated at the same time.
- What a Passing Criterion Must Include:
    - A named held-out set (test data kept strictly apart during development) of a stated size.
    - A minimum level of quality (e.g., accuracy or F1 score).
    - A safety rate (how often outputs stay within safe/compliant bounds).
    - A split between inconvenient errors and serious errors (treating minor mistakes differently from harmful ones).
    - A response time (latency) limit.
- Why Vague Criteria Fail: Statements like "classify sentiments well" or "claim replies should get better" only describe a direction of improvement. Because you can't pass or fail them, they cannot end a project.

**EXAMPLE**

Scenario: A Retail Bank Building an LLM Assistant to Explain Credit Card Fee Disputes

- Failing Criterion (Direction-only / Vague):
"The AI assistant should explain fee disputes accurately and make customer replies better."

    - Why it fails: There is no pass/fail line. If accuracy rises from 60% to 65%, it's "better"but still unusable. Nobody can prove the project is done.

- Passing Criterion (Pass/Fail Discovery Record):
    - Evaluated on a held-out set of 500 real past fee-dispute tickets (locked away during prompt development):
        - Minimum quality: Correctly identifies the fee type and policy rule on at least 92% of cases (target grounded in an earlier 50-case pilot experiment, not optimism).
        - Inconvenient errors (minor): Formatting glitches or slightly stiff tone allowed on up to 8% of cases.
        - Serious errors (high consequence): Promising an unauthorized fee waiver or citing a non-existent banking regulation must stay below 0.5%.
        - Safety rate: 99.5%+ compliance with PII redaction and no unauthorized financial advice.
        - Response time limit: 95% of drafts generated in under 4 seconds.

    Because every number is testable on the 500-case held-out set, the team and the accountable owner can objectively say pass or failand actually finish the project.

    One Practical Nuance on Timing: During Discovery, you don't always label all 500 cases on Day 1 but you do verify the data exists, agree on the exact size and pass/fail thresholds, and quarantine (lock away) the held-out slice before anyone starts prompt engineering.

---

### Feasibility Triage, the Honest No, and Starting Simple

- Purpose of Feasibility Triage: Three gatekeeper questions decide whether an engagement should proceed in its current form, be reshaped, or stop:
    - Verifiable Task & Ground Truth: Does the task have a clear correct answer and a verifiable outputor is it just a subjective aesthetic judgement with no ground truth (a known correct answer to compare against)?
    - Data Availability, Quality & Lawful Use: Does the data exist, is its quality good enough, and is it legally/ethically permitted to be used for this purpose?
    - Tolerable Error Cost: At a realistically achievable accuracy level, can the business tolerate the consequence of error (the cost of the remaining wrong outputs)?
- The "Honest No" (Finding, Not Failure): Answering "no" to any of these three questions is a valuable discovery finding, not a failed project. Discovering that a client lacks a lawful basis to use customer recordsand stopping before spending money on a build is good architecture.
- Anthropic's Guidance on Agents (Simplicity First): Always start with the simplest solution possible (e.g., a single prompt or basic workflow). Add complexity (agents, tools, multi-step orchestration) only when it demonstrably improves outcomes on your evaluation metrics.

**EXAMPLE**

Scenario: A Health Insurance Company Asking for an Autonomous Claim-Appeal Agent

The client asks for a multi-agent LLM system that reads rejected medical claims, decides whether to overturn the rejection, and automatically emails a "warm, empathetic" approval or denial letter to the patient. The architect runs Feasibility Triage:
- Verifiable output & ground truth?
    - Deciding whether a claim is covered has ground truth (policy rules + past senior adjuster rulings).
    - Judging whether the email is "warm and empathetic enough" is mostly an aesthetic judgement harder to verify strictly.
- Data exists, good enough, and lawful to use?
    - NO: Discovery uncovers that doctor notes are messy handwritten scans (poor quality) and patient consent forms do not cover sending medical records to a third-party cloud LLM (no lawful basis).
- Is the cost of being wrong tolerable at achievable accuracy?
    NO: Even a strong LLM reaches ~90% accuracy on complex appeals. A 10% error rate on autonomous medical denials triggers severe regulatory fines and patient harmtotally intolerable.

The Outcome (The Honest No + Simplicity First):
- Because Questions 2 and 3 are "No," the architect delivers an Honest No stopping the autonomous agent build. This is a finding, not a failure (it prevented a massive legal and financial disaster).
- Instead of proceeding in its current form, the engagement is reshaped: first, Legal and IT secure a compliant private model and clean up document intake. Second, following Anthropic's simplicity rule, they don't build a complex multi-agent system they start with a single, simple LLM call that extracts policy clauses for a human nurse to review, and only add more complexity later if evals demonstrably prove it improves accuracy.

---

### Exam Strategy

- The Golden Rule for Right Answers: Every correct answer represents professional behaviour under pressure including giving an honest refusal when a request is impossible, unsafe, or unscoped.
- The Four Failure Postures
    - Capitulation: Giving in and agreeing to the impossible (e.g., quoting a fixed price before scoping the work).
    - Evasion: Silently complying, or secretly building the system your own way without telling the stakeholder.
    - Obstruction: Bypassing the accountable owner by taking the issue above their head.
    - Theatre: Performing busywork, offering more meetings, more metrics, more surveys, or more documents instead of producing real decisions or artefacts.
- Spot which wrong option is Capitulation, Evasion, Obstruction, or Theatre, and eliminate it.

**EXAMPLE**

Scenario: A Retail Bank Pressuring an AI Architect

The VP of Retail Banking (the accountable owner) tells the architect: "Our competitor just launched an AI mortgage advisor. Quote me a fixed price by Friday and build an autonomous LLM that approves mortgages in 4 weeks skip the data privacy review so we move fast."

Here is how the four failure postures look as wrong exam answers vs. the right answer:
- Capitulation (Wrong): "Sure, we can do it for $180,000 in 4 weeks!" → Agrees to an impossible timeline and prices unscoped work.
- Evasion (Wrong): You nod and say "Got it," then secretly build a read-only summary tool instead of an approval bot without telling the VP. → Complies silently and builds your own design behind the owner's back.
- Obstruction (Wrong): You walk out of the meeting and email the bank's CEO to complain that the VP is being reckless. → Goes above the accountable owner instead of working through the trade-offs with them.
- Theatre (Wrong): "Before we decide, let's run a 3-week employee survey, hold four alignment workshops, and produce a 50-page AI readiness report." → Creates busywork and documents instead of concrete discovery artefacts.

- Professional Behaviour / Honest Refusal (Right): "I cannot quote a fixed price before scoping, and we cannot launch autonomous approvals without confirming lawful basis for customer credit data and agreeing on the cost of a wrong approval. Let's first establish the baseline, get Legal to clear the data, and define pass/fail accuracy thresholds  then we can scope a safe first version."