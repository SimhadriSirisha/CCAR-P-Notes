1. A risk register is a table where each row describes one specific way your AI system can fail. It is not just paperwork—it directly shapes how engineers build the system. It is an architecture artefact because each row actively changes how you build the system. 

    Every row must answer five questions:

    |Field|Explanation|Example|
    |-|-|-|
    |Mechanism|Exactly how does it break technically?|A hidden instruction inside an uploaded claim PDF tricks the model into calling the cancel_policy tool.
    |Blast radius|How bad is the damage, who gets hurt, and can it be undone?|Customer loses active insurance coverage immediately; customer bears the harm; cannot be undone cleanly.
    |Detection signal|What will spike or change in your logs and metrics while the failure is happening?|% of cancel_policy approval requests rejected by staff (a sudden jump means the model is trying to cancel policies it shouldn't).
    |Owning control|Which specific guardrail stops this failure, and which of the 5 control surfaces does it sit on?|Mandatory staff approval gate before cancel_policy executes (Capability control).
    |Residual & who accepted it| What risk is left over? If no control covers it, it is an accepted risk, and a real person must sign their name to accept it. | Residual risk: a tired staff member accidentally clicks "Approve." Accepted and signed off by the Head of Claims Operations.

2. Risk Register Real usecases :
    - If a high-impact row (like "agent could issue an unauthorized refund") has no owning control, the launch is blocked until engineers either build the control or a senior leader formally signs off to accept the residual risk.
    - Each row describes a technical failure mechanism, teams turn rows directly into sprint tickets.
    - The Detection signal column is handed to the platform or observability team. So, the team configure their logging and AI observability tools to watch for those exact signals
    - As Proof for Audits and Regulators

3. When an AI makes something up (confident fabrication), it doesn't crash or throw an error message it just writes a smooth, believable answer that happens to be wrong. Because nothing technically "errored," your normal error-rate metrics will show 0% errors.
    
    Prompting techniques can cut down hallucinations a lot, but they can never drop them to zero. Because of that, any important fact the AI outputs must be checked against a real, verifiable source—not just accepted because it sounds right. And simply saying "we tested it a lot and it worked" is never enough to count as a real risk assessment.

    **risk registry entry example: "How many days do I have to dispute a fraudulent charge?"**:

    |Field|Example Entry
    |-|-
    |Mechanism|"The model fabricates a plausible but incorrect dispute deadline (e.g., 90 days instead of 60) when answering a customer question without a grounded source."
    |Blast radius|Customer misses the real deadline and loses the right to recover fraudulent charges; customer bears the financial loss (and the bank faces regulatory/legal exposure); hard or impossible to undo.
    |Detection signal|"Spike in customer escalations or complaints citing wrong policy dates, or a drop in citation-match rates during automated output checks."
    |Owning control|Mandatory citation/retrieval grounding paired with an output validation check against the official policy database (Output control).
    |Residual & who accepted it|Residual risk: an outdated policy document in the database causes a wrong answer. Accepted and signed off by the Head of Retail Banking Compliance.

4. Risk register tells you what can fail and how you'll prove it's controlled, and non-determinism explains why normal software testing and debugging don't count as proof for AI systems.

    Because LLMs are non-deterministic, giving the exact same input twice can produce two different answers. That breaks three normal software assumptions:
    - Testing: Passing a test once doesn't prove it will always pass.
    - Debugging: Trying to reproduce a bug once and seeing a good answer doesn't mean the bug isn't real.
    - Auditing: If you need to investigate a past decision months later, you cannot just re-run the prompt—you must look at the saved trace (the recorded conversation and tool calls) from when it actually happened.

    Here is how the **non-determinism** risk looks as a row in the risk register:

    use-case example : **"Last month, your chatbot told me I had 90 days to dispute a charge, so I missed the real 60-day deadline."**

    | Field | Example Entry |
    | --- | --- |
    | **Mechanism** | Because the LLM is non-deterministic, re-running a customer's prompt months later produces a fresh answer—making it impossible to reproduce or verify what the assistant actually told the customer. |
    | **Blast radius** | Bank cannot prove what advice was given during a customer complaint or regulatory audit; bank bears legal/regulatory penalties; **cannot be undone** (missing history cannot be reconstructed later). |
    | **Detection signal** | Metric alert when `% of completed customer sessions missing a saved trace` rises above 0%, or when trace-storage write errors spike. |
    | **Owning control** | Mandatory, immutable **trace logging** (recording exact prompts, retrieved documents, tool calls, and final outputs) stored in a separate audit database (*Detective / audit control*). |
    | **Residual & who accepted it** | Residual risk: storing full transcripts increases data-privacy exposure if retention/redaction rules fail. Accepted and signed off by the Head of Compliance & Data Privacy. |

5. When you add prompt injection to your risk register, don't waste time listing every clever trick an attacker might use to fool the AI those tricks change every week.

    Instead, look at what tools and permissions the AI has, because that decides the blast radius (the worst-case damage because by knowing the permission set immediately tells you who gets hurt and how bad it can get.) if the AI gets hijacked:
    - If the AI can only read and summarize, a hijacked turn is a small, contained incident.
    - If the AI has a payment or refund tool, a hijacked turn is a full-blown financial breach.
    
    Write the risk register row around the permissions you control, not the attack tricks you don't.
    - The risk (Mechanism) is tied to the permission set—what the agent is allowed to do if a turn gets hijacked, regardless of how the attacker tricked it.
    - The blast radius is what happens if those permissions are abused—the worst-case damage, who gets hurt, and whether it can be undone.

    Example:

    Bad risk register row (focuses on the changing technique):"Risk: An attacker hides white-on-white text in a PDF using a Base64 jailbreak."(Useless next month when attackers switch to a new trick.)

    Good risk register row (focuses on the permission set):

    - Read-only agent: "Risk: Compromised turn on an agent with read_policy and summarize permissions." → Blast radius: Contained incident (at worst, a confusing or rude summary).

    - Refund agent: "Risk: Compromised turn on an agent with issue_refund permission." → Blast radius: Major breach (unauthorized money leaves the bank; hard to undo).

6. When an AI agent runs multiple steps on its own, small mistakes don't stay small they build on each other, cost more money, and cause bigger damage. That is why agents need heavy testing in a safe sandbox first, and should only be used where success is easy to measure and humans can oversee them.

    You cannot control the model's accuracy to perfection. Instead, you control four architectural dials:
    - Scope: What the agent is allowed to access(which tools, accounts, or data). If an error compounds, a narrow scope keeps the damage small and contained.
    - Step count: Maximum number of steps before it must stop. So error accumulation can be stopped.
    - Reversibility: Only letting it take actions that can be undone. If an action cannot be undone (like sending money or deleting records), the agent shouldn't do it autonomously.
    - Checkpoints: Forcing it to pause and ask a human before a big action or when it gets stuck. As agents can pause for feedback at checkpoints or when blocked.

    Here is how the **compounding autonomy error** risk for the bank's refund agent looks as a row in the risk register:

    | Field | Example Entry |
    | --- | --- |
    | **Mechanism** | In a multi-step dispute loop, the agent makes a small early error (such as misreading a receipt or fee) that compounds across subsequent tool calls until it prepares and triggers an incorrect `issue_refund` payout. |
    | **Blast radius** | Incorrect or unauthorized funds leave the bank; bank bears the financial loss and regulatory/customer-trust damage; **cannot be undone** cleanly once the transfer settles. |
    | **Detection signal** | Spike in `agent_max_steps_reached_rate` (agent hitting its step ceiling), jump in `checkpoint_rejection_rate` (staff rejecting the agent's proposed refunds), or rising `token_cost_per_session`. |
    | **Owning control** | Hard **step-count cap** (max 5 tool calls), narrow **scope** (read-only access to the single customer's dispute), restricting autonomous actions to **reversible** ticket drafts, and a mandatory **human checkpoint** before `issue_refund` executes (**Capability control**). |
    | **Residual & who accepted it** | Residual risk: a staff member rubber-stamps a bad refund at the checkpoint without catching the earlier compounding error. Accepted and signed off by the Head of Dispute Operations. |

8. Most teams forget to put three important risks into their AI risk register even though they can quietly break your system:
    1. Over-refusal: Making safety filters too strict causes the AI to block real, legitimate customer requests. You must track both bad requests blocked and good requests accidentally denied.

    2. Over-reliance downstream (Automation bias): Imagine you add a human approval gate before the AI sends a refund. If the AI is right 98% of the time, the human reviewer gets bored, trusts the AI completely, and starts clicking "Approve" without actually checking (rubber-stamping). The UK's data regulator (the ICO) calls this automation bias. When a human approves everything automatically, the human gate stops being a real control—so it needs a row in the risk register, and the gate must be designed to force real checking.

    3. Third-party surface: Any external tool, MCP server, connector, or analytics vendor connected to your AI can see your data or take actions. Your AI provider (like Anthropic) doesn't cover them—so each connector needs its own row in your risk register.

9. Don't sort your risk register by how likely a problem is. Sort it by how bad the damage is (consequence) and how hard it is to spot (detectability). A rare, silent, irreversible failure is much more dangerous than a common, obvious, easy-to-fix one.
    
    Also, never leave "How would we notice?" blank—and never write "a customer will complain." Customer complaints happen after the damage is done and miss everyone who stays quiet. Any row without real monitoring is where your next major incident will come from.

10. if an exam question says "This autonomous agent has 95% accuracy per step, so it is safe to run a 10-step workflow," do not fall for it. Because errors multiply across steps (0.95^10≈60%), a high per-step accuracy number tells you almost nothing about how often the whole workflow will actually succeed.

    An accuracy figure measures one thing. It says nothing about blast radius or the detection signal, so what a wrong action costs and how the team would notice one stay unknown. Register each irreversible action with a control, an owner and a detection signal.

11. A surface-only fix is a wrong fix as it placed after the point of harm. The action has already reached the recipient; the wording neither prevents the error nor detects it, and transfers no consequence away from them. Put a checkpoint before the action and add a detection signal. 
