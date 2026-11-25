1. A human gate is a checkpoint where a person must review or approve an AI's action before it actually happens.
    
    When designing one, follow two steps in strict order:
    
    - Placement first (Where): Find the exact point in the workflow where a mistake would cause serious harm and be hard to undo (high blast radius). Put the gate right before that action.
    - Pattern second (How): Once you know where the gate sits, design how the human will review it.

2. 5 patterns of Human gate:

    | Pattern | How It Works | Best Used For | Key Catch / Rule | Example (How One Bank Uses All Five)
    | --- | --- | --- | --- | --- |
    | **1. Approve-before-execute** | AI proposes an action → human approves → action runs. | Irreversible external actions (payments, deletions, customer emails). | Gate sits *before* execution. | AI prepares a $500 fraud refund → staff clicks approve → money moves. | 
    | **2. Review-before-release** | AI drafts content → human reviews & signs off → content is published. | Consequential content (regulated advice, clinical docs, official commitments). | The human reviewer owns **accountability** for the output. | AI drafts a formal letter explaining a loan denial → compliance officer edits and signs their name to it → letter is sent. | 
    | **3. Triggered escalation** | AI handles routine work; sends uncertain, out-of-policy, or high-value cases to a human. | Mixed workloads where most cases are safe and a few are risky. | **Validate the trigger**—an unvalidated confidence score is not a real control. Because if the trigger is on the basis of confidence score, AI can completely hallucinate and still give itself a 95% confidence score which is risky | AI auto-approves routine $10 fee waivers, but routes anything over $100 (or flagged as unusual) to a manager—*after* the bank validates that the routing rule actually catches edge cases. |
    | **4. Post-hoc sampled audit** | No gate upfront; humans check a random sample *after* the fact. | High-volume, reversible, low-severity tasks (e.g., account tagging). | Also used to **detect rubber-stamping** in the other patterns. | AI tags 10,000 overdue accounts overnight; QA checks 100 random tags the next morning (and also audits 5% of the refund approvals from Pattern #1 to make sure staff aren't rubber-stamping). | 
    | **5. Automate with reversibility guarantee** | Fully automated, no human gate—backed by an easy undo path. | Low-friction updates where mistakes are cheap to reverse. | Must have an **audit trail**, **undo path**, or **compensating transaction**. | AI updates a customer's mailing address automatically, while logging the old address so a single click can restore it if the AI got it wrong. |


    > **rubber-stamping means Over time, automation bias kicks in. Because the AI is right most of the time, reviewers stop reading the receipts and just click "Approve" on everything

3. You don't always get to choose whether to add a human gate, Anthropic's Usage Policy requires one in 11 High-risk Areas (such as legal/medical advice, credit, insurance, housing, and employment).
    
    To know if the policy forces a human gate, ask two questions:
    - To whom? Is this about a specific person (advice they'll rely on, or a decision about them)?
    - What does it do? Is it a final recommendation in one of the 11 High-risk Areas?

    If YES to both, you must apply two controls:
    - A qualified human with authority to change it must review it first.
    - The person must be clearly told AI was used.
    
    If NO (e.g., general educational info, wellness tips, internal drafts/summaries, or general business operations not about one person), the policy does not force a gate.
    
    Finally, under Anthropic's commercial terms, your organization not Anthropic is responsible for deciding whether outputs are appropriate and where human review belongs.

    **EXAMPLE**

    1. Excluded (No policy gate required): The bank's AI writes an internal summary for branch managers showing how many loan applications were rejected this month.
        
        Why: It is an internal summary and decides nothing about any individual customer.
    
    2. Policy applies (Mandatory gate + disclosure): The bank's AI recommends denying a specific customer's credit card application.

        Why: It is a decision about one person in a High-risk Area (credit).
        
        What's required: A qualified loan officer with the power to override the decision must review it first, and the customer must be clearly told that AI was used in the process.

4. Control v/s Control gate v/s Control Surface

    They are nested concepts: a **control surface** is *where* you can defend, a **control** is *any* safeguard you put there, and a **control gate** is a specific type of control that *pauses or blocks* an action until a check passes.

    | Term | What It Means | Bank Refund Agent Example |
    | --- | --- | --- |
    | **Control Surface** | The architectural layer *where* you can intervene (Input, Output, Capability, etc.). | **Capability surface** (the tools and permissions the agent can use). |
    | **Control** | Any specific mechanism—preventive *or* detective—that reduces or spots a risk. | A 5-step limit, read-only ledger access, immutable trace logging, or a human approval step. |
    | **Control Gate** | A specific **blocking** control that stops an action or output from proceeding until a human or automated check approves it. | A mandatory **human checkpoint** that pauses `issue_refund` until a staff member clicks Approve, or an **output gate** that blocks an email with a wrong dispute date. |

    How They Fit Together
    * Every **control gate** is a **control**, and every **control** sits on a **control surface**.
    * **Not every control is a gate.** Trace logging, post-hoc sampled audits, and narrow read-only permissions are all **controls**, but they are not **gates**—because they don't pause a workflow mid-flight to wait for a pass/fail decision.

5. A human gate only counts as real oversight if the reviewer is effective, not decorative.
    
    Under the UK ICO's test, a human reviewer must:
    - Actively check the AI's recommendation instead of routinely clicking "Approve."
    - Have the competence and authority to overrule the AI.

    To stop reviewers from just repeating the AI's logic, give them:
    - The trace and sources the AI used.
    - Any flagged uncertainty.
    - At least one input the AI never saw (so the human weighs new factors).
    - Enough time to actually read it all.

    **EXAMPLE**

    Suppose a company's AI recommends canceling a customer's policy:
    - Decorative reviewer (fails the ICO test): The manager's screen shows only the AI's recommendation ("Cancel policy") and the exact same 3 billing lines the AI read, with a 15-second target per ticket. The manager clicks "Approve" every time. Legally, this is still a solely automated decision.
    - Effective reviewer (passes the ICO test): Before canceling, the manager sees the AI's reasoning trace, the billing sources, where the AI was uncertain, plus the customer's last phone call notes (which the AI never had)—and has 5 minutes and full authority to overrule the cancellation if the call notes show a billing mistake.

6. A human gate isn't a real control unless you track three performance numbers:
    1. Override rate: How often reviewers change or reject the AI's recommendation.
    2. Reviewer agreement: How consistently different reviewers make the same call on similar cases.

        If a refund decision requires reading a 2-page receipt and a call log, an average time of 4 seconds proves mathematically that nobody is reading them.
    3. Time per item: Whether reviewers are actually spending enough seconds/minutes to read the case.
        
        If a refund decision requires reading a 2-page receipt and a call log, an average time of 4 seconds proves mathematically that nobody is reading them.
    
    If the override rate is near 0% at high volume, reviewers are rubber-stamping there are usually two root causes:
        
    - The gate is in the wrong place: You are forcing humans to review 295 boring, routine $10 refunds just to catch 5 risky $5,000 ones. Fatigue guarantees they will zone out and miss the 5 bad ones.

    -  Reviewers can't see enough: If the screen only shows "AI recommends: Approve $500" without the raw receipt, policy rule, or customer call notes, the reviewer has no basis to disagree—so they default to trusting the AI.

    
    You can't fix automation bias just by training reviewers to "pay closer attention." Instead, you have to divide the job between the AI and the human before a single line of code is written and that takes three teams working together at the scoping and design stage:
    
    1. The Business (Operations / Product): Knows the real-world context, what exceptions actually happen with customers, and how much time staff realistically have to review a case.
    2. Data Scientists / Engineers: Know what structured data the AI can reliably process and where the model is blind or prone to error.
    3. The Oversight Function (Risk / Compliance / Legal): Knows the regulatory rules (like the ICO's test) and ensures the human reviewer is actually weighing independent factors not just acting as a decorative rubber stamp.
    
    Finally, whoever clicks "Approve" owns the outcome. So they must always see the evidence behind it.

    **EXAMPLE**

    - Broken gate: A bank manager approves 300 AI-drafted refunds a day with a 0.1% override rate and an average time per item of 4 seconds.
    
        Diagnosis: The gate is in the wrong place (gating routine $15 refunds alongside $5,000 ones) and the manager can't see enough to challenge the AI. Telling the manager to "be more careful" in a training session won't fix this.

    - Fixed at scoping & design: Business, data science, and compliance redesign the workflow together:

        What the AI considers: Receipt amounts, transaction dates, and policy rules.

        What the human weighs: Customer call notes and fraud-history flags—and only on refunds over $500 (moving the gate to the right place).

        Because the manager signs off and owns the outcome, the screen displays the AI's calculation basis alongside the call notes, and the team monitors override rate, reviewer agreement, and time per item every week.

7. Automate with a reversibility guarantee is a legitimate right answer wherever the question says the action is low-impact and reversible

8. An added reviewer still sees only the drafted output, so two reviewers give two rubber stamps, and seniority adds authority, not information. Give the reviewer the trace, the sources, an input the model never had and time per item, then measure the override rate.