### The Seven Parts of Complete Architectural Communication

A complete communication lets others evaluate a decision rather than blindly accepting it. It always contains these seven parts:
- The Decision: Clearly state what is being decided.
- Genuinely Considered Options: Present real alternatives, explicitly including the option the audience expected.
- Criteria: State what the options were judged on so the weighting is visible and arguable.
- Evidence: Provide real proof. For a Claude system, this means Domain 4 measurements, not assertions.
- Costs and Risks: Show what each option costs and what risks it introduces.
- A Recommendation: State your clear architectural choice. Declining to recommend is a failure of responsibility, not neutrality.
- What Would Change the Recommendation (Revisit Condition): Name the exact measurement, volume, price change, or regulatory event that would reverse your recommendation.

**EXAMPLE**

Scenario: A Telecoms Company Deciding on Automated Billing Refunds

The VP of Customer Support wants Claude to automatically issue roaming fee refunds up to $50 with no human review. The architect writes a complete 7-part communication:

- The Decision: Whether the Claude billing assistant will issue roaming refunds up to $50 automatically or require human agent approval.
- Options Genuinely Considered:
    - Option A (what the VP expected): Fully automated refunds up to $50 with no human check.
    - Option B: Claude calculates the refund and drafts the reply, and a human agent clicks "Approve" after a quick check.
    - Option C: A deterministic rule-based script with no LLM.
- Criteria (visible weighting): Refund calculation accuracy (50%), regulatory compliance (30%), and average handling time (20%).
- Evidence (Domain 4 measurements): Tested on 500 held-out historical billing disputes. Option A issued incorrect refund amounts on 4.5% of cases (misreading legacy family-plan add-ons). Option B caught 99.3% of errors during human review while still cutting handling time by 45%. Option C failed to parse 35% of free-text customer complaints.
- Costs and Risks: Option A saves $18,000/month in agent time but creates ~$40,000/month in erroneous payouts and telecom regulator complaints. Option B saves $11,000/month with minimal payout risk.
- Recommendation: I recommend Option B (human 1-click approval). It delivers nearly half the speed gain while keeping payout errors within our risk limit. (The architect takes responsibility and recommends, rather than staying neutral.)
- What Would Change the Recommendation (Revisit Condition): Switch to Option A if either: (a) a future model or pipeline update brings our held-out refund error rate below 0.5% across 1,000 test cases, or (b) monthly roaming disputes double past 80,000 tickets/month where human review causes SLA breaches.

---

### Tailoring Communication by Audience
- The Golden Rule: Every audience gets the same decision and the same facts (such as the exact same error rate). Only two things change:
    - The lead: What you say first.
    - The level of technical detail.
- How to Lead for Each of the 4 Audiences:
    - Executives: Lead with business consequence and risk.
    - Engineers: Lead with mechanism and constraint (because that is what they actually build against).
    - Legal and Compliance: Lead with the regulatory/legal obligation and the residual risk (the risk that still remains even after safeguards and controls are put in place).
    - The Sponsor: Lead with cost, date, and precisely what they are being asked to accept.
- Three Communication Failures to Avoid (Exam Traps):
    - Giving an executive too much technical detail.
    - Giving a specialist vague generalities they cannot act on.
    - Simplifying a statement until it becomes untrue (the worst failure). The person who relied on that simplified statement will always discover the problem later. eg: for example, telling a VP "the model does not hallucinate" when it actually has a 2% error rate

---

### Communicating Error Rates, Demo Risks, and Evaluation Uncertainty

- State the Error Rate Before Sign-Off:
    - Every AI system has an error rate, and the architect must state it clearly before anyone signs off.
    - Always include three things together:
        - The error rate itself.
        - The population it was measured on (what test data produced that number).
        - The handling path for residual errors: who catches the remaining mistakes, how quickly, and at what cost.
        - eg: On our held-out test set of 200 historical fraud alerts, our best setup has a 6% error rate (94% accuracy, with a 95% confidence interval of 90.7% to 97.3%). The remaining 6% of misrouted alerts are caught by Tier-2 fraud analysts during their 2-hour audit sweep, at an operational cost of roughly $1,800 per month in analyst review time.

    - Why you must say the error rate out loud before sign-off: LLM systems are probabilistic, so they will make mistakes. If you hide or downplay the error rate to get a project approved, stakeholders will treat the first production mistake as a disaster. Saying the exact error rate upfront (along with who catches the mistakes) makes the risk explicit before anyone signs the contract.

- Why Demos Are a Communication Risk:
    - A live demo is a communication risk, not a communication tool.
    - Demos sample from the best end of possible outcomes. Showing five perfect answers in a row tricks the audience into believing the system is 100% reliable, because a demo only shows the best end of the outcome range.

- Communicating Uncertainty (Anthropic's Evaluation Format):
    - Express uncertainty as a range (interval), not a single point number.
    - Always state what work would narrow that range (for example, collecting a larger held-out test set).
    - Single candidate: Report the interval with each score, never the score alone.
    - Comparing two candidates: Also report an interval for the pairwise difference (the difference between their two scores).
    
        eg: 
        - Prompt A scored 94.0% (interval: 90.7% to 97.3%).
        - Prompt B scored 91.5% (interval: 87.6% to 95.4%).
        - Pairwise difference: Prompt A is ahead by +2.5 percentage points, with a pairwise difference interval of -2.1% to +7.1%.
        - What the architect tells the bank: "Because that difference interval crosses zero (-2.1% to +7.1%), this 2.5% gap could just be random variation, not a real win for Prompt A. To narrow this range and prove which prompt is actually better, we need 3 days of work to expand our held-out set from 200 alerts to 1,000 alerts."
    - Why it matters: Without these intervals, a comparison can falsely present random variation as a real finding.

---
### Handling Stakeholder Disagreements (Business Trade-offs vs. Legal/Safety Obligations)

- The Standard Disagreement Pattern (When evidence contradicts the owner):
    - Follow three steps: Measure, explain the trade-off, and recommend.
    - Then leave the decision with the accountable owner.
    - Give the owner five specific things so they can decide properly:
        - The quality difference as a number.
        - Its downstream cost in the resources the owner controls.
        - The exposure to the organisation.
        - The saving (what the owner gains from their preferred choice, such as hitting a deadline).
        - A clear recommendation.
    - Three Responses That Always Fail:
        - Silent non-compliance (quietly doing something else without telling them).
        - Compliance without comment (agreeing without laying out the numbers and risks).
        - Escalation above the owner as a first step (going over their head instead of giving them the facts first).

- The Core Rule (Judge by Consequence, Not Loudness):
    - Margin, date, or quality level: Owner decides with all the facts.
    - Unlawful data path or unsupervised action with a safety consequence: Hard obligation (cannot be traded off). The owner does not have the authority to override the law or safety rules. You must state that plainly and bring in the responsible department (like Legal, Compliance, or Safety).
    - How loud, urgent, or senior the stakeholder is is not evidence either way.