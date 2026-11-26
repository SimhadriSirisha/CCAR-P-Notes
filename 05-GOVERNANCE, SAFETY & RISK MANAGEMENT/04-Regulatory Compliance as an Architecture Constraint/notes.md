1. Obligation(a legal or contractual rule) turns to Architecture constraint when that directly changes how you design and build your system.

    Regulated data: Sensitive data restricted by law or contract (like personal data or medical records).
    
    Hop: Any system, API, or service that regulated data touches—whether it is just sent there briefly or stored there.
    
    Trust boundary: The invisible fence separating the systems you legally trust to handle regulated data from everything else outside.

    **EXAMPLE** (Bank Customer Assistant)
    
    * Regulated data: A customer's name, account number, and loan balance.
    
    * Obligation: Banking law says this data cannot be shared with unapproved third-party vendors.
    
    * Trust boundary: The bank's internal servers plus its approved AI provider (which signed a strict data-protection contract).
    
    * Hops: The data travels from the Bank App → AI Model → Third-party analytics tool.
    
    * Architecture constraint: Because that third hop (the analytics tool) sits outside the trust boundary, the obligation forces the architect to change the design—either by stripping/redacting all customer details before logging, or removing the analytics tool from that path entirely.

2. When teams do a privacy or compliance review, their biggest mistake is drawing a diagram of **the app** (user → backend → LLM → user) and stopping there.

    Regulated data does not stay inside the app—it gets copied into half a dozen side systems along the way. If you only review the app boundary, you completely miss those copies, which means you have unassessed compliance leaks.

    The Core Idea: Follow Every Copy of the Data

    Instead of mapping your application code, you have to trace **every single hop** where sensitive data gets sent or stored:

    * **The model provider:** The API call carrying the user's prompt.
    * **The retrieval index & embeddings:** Vector databases storing chunks of sensitive documents (and their embeddings).
    * **Prompt and response logs:** Standard server logs capturing raw inputs and outputs.
    * **Observability traces:** Debugging traces captured to monitor latency, errors, and tool calls.
    * **Evaluation datasets:** Real production examples saved to test and grade the AI's accuracy.
    * **Analytics/monitoring vendors (and their subprocessors):** Third-party tools that ingest logs or traces, plus the outside companies *those* vendors use.

    The Big Exam Trap: Traces and Eval Sets

    **Observability traces** and **evaluation datasets** are the two copies teams forget most often.

    * Why? Because engineering teams build them deliberately for monitoring and quality testing, not as part of the customer-facing app. When compliance asks *"Where does customer data live?"*, teams point to the main database and completely forget that raw customer data was copied into an eval spreadsheet or a third-party tracing dashboard.

    What the UK ICO Requires

    To prevent these blind spots, the ICO sets three strict rules for personal data:

    1. **Record every movement and storage location** (every hop, not just the main app).
    2. **Keep clear audit trails** of where the data went.
    3. **Delete intermediate files** once they are no longer needed (don't leave old logs, temp files, or eval copies sitting around forever).

    **EXAMPLE**

    A customer types into an insurer's AI chat: *"Does my policy cover my recent Type 2 diabetes diagnosis?"*

    * **What the team's *App Diagram* shows:** Customer → Insurer App → AI Provider → Answer. (They assume the medical data only touches two places.)
    * **What the *Whole Data Path* actually shows:** That single diabetes diagnosis just got copied to **five different places**:
    1. Sent to the **AI provider** in the prompt.
    2. Written to the insurer's **prompt/response logs**.
    3. Captured inside a **third-party observability vendor's trace** (so engineers can debug slow responses).
    4. Exported by a data scientist into an **evaluation dataset** to test next month's model upgrade.
    5. Passed to the observability vendor's **cloud subprocessor** for storage.

    If the insurer only assessed the main app, that customer's medical diagnosis is now sitting 
    
    unprotected in a trace vendor and an eval dataset—violating data protection rules without anyone realizing it.

3. Send the minimum: Only send the exact fields the AI needs to do the job.

    Mask before crossing: Redact, tokenise, or pseudonymise regulated data before it leaves your trust boundary, and keep the re-identification key inside. Once raw data crosses, you can't take it back because the recipient holds it.
    
    De-identify before extracting for evals/training: Whenever you pull live customer records to build a training or evaluation dataset (even for internal teams), strip out identifying details before exporting if the risk to individuals is high.

    **EXAMPLE**

    At the live boundary: A customer asks the insurer's AI assistant "What is the status of my claim?" The backend only sends claim_id: 9921 and status: approved across the trust boundary to the LLM. The customer's medical diagnosis stays inside the insurer's database because the AI doesn't need it to answer the status question.
    
    For the evaluation set: Later, the QA team wants 500 real customer chats to test a new prompt. Before exporting those chats from the production database, an automated script replaces all customer names, policy numbers, and phone numbers with tokens ([USER_1], [POLICY_1])—so the evaluation set still tests the AI's accuracy accurately without leaking anyone's identity.

4. Never send regulated data to a vendor until a contract restricting its use is already signed.

    Under HIPAA (the US health-data privacy law), any vendor handling health data for an insurer or hospital is called a Business Associate. Their contract must require four things:
    - Permitted uses: Exactly what they are allowed to do with the data.
    - Safeguards & breach reporting: How they protect it, and the duty to report breaches.
    - Flow-down: Any subcontractor they use must obey the exact same rules.
    - Return or destroy at termination: When the contract ends, return or delete every copy—and if any copy can't be deleted, keep protecting it under the contract's rules indefinitely.

    Most AI teams fail #3 (forgetting their vendor's subprocessors, vendor brings in their own subcontractors/subprocessors like an analytics tool sitting behind your trace vendor and if you didn't map the whole data path, those downstream vendors end up holding regulated data without being bound by your contract.) and #4 (forgetting hidden copies in evaluation datasets).

    **EXAMPLE**

    A health insurer wants to use a third-party trace vendor to monitor its AI claims assistant:

    Timing: The trace vendor must sign a HIPAA Business Associate contract before the very first trace is sent.
    
    Flow-down failure: The trace vendor quietly sends log data to an outside analytics subprocessor that never signed a matching contract breaking the chain of protection.
    
    Return-or-destroy failure: A year later, the insurer terminates the vendor. The vendor deletes the main trace database, but forgets 1,000 real patient chats that were exported into an evaluation dataset. Even if the vendor claims it's "not feasible" to scrub those chats from archived backups, the contract's protection rules still bind them to keep securing that data.

5. Four compliance rules you have to design into your architecture before launch and why copies of data make them much harder than teams expect.

    1. Purpose and Retention (Decide Why and How Long Upfront)
    - Purpose: Exactly why you are collecting and keeping a piece of data.
    - Retention: Exactly how long you will keep it before deleting it.
    - The rule: Don't figure this out after launch. Decide both before launch not just for your main database, but for every single copy (logs, traces, vector indexes, eval sets).

    2. Residency (Where Data Lives and Runs)
    - Residency means the physical geographic region (e.g., EU, US, India) where inference runs (where the AI processes the prompt) and where data is stored.
    - State the requirement in terms that will not go out of date i.e. Don't write a compliance requirement like "Use AWS us-east-1" or "Use Claude's current EU beta endpoint", vendor regions and options change all the time. Instead, state the durable rule (e.g., "All EU customer personal data must be processed and stored exclusively within the EU"), and then check which regional endpoints the provider currently supports to meet that rule.

    3. Data-Subject Rights (Engineering, Not Just Policy)
    
        Under privacy laws (like GDPR), people have data-subject rights over their own personal data:
        - Access (show me what you have on me)
        - Correction (fix my wrong details)
        - Deletion (erase my data)
        - Portability (give me a copy to take elsewhere)

        To actually honor a deletion or access request, your engineers must be able to find every copy of that person's data. That requires two things built into the system:
        - An inventory: A live list of every place data is stored.
        - Provenance: Tracking where each record came from so you can trace a specific person's data across systems.

        **The big trap**: When a user asks to be deleted, teams usually delete their row in the main SQL database and completely miss the copies sitting inside vector indexes (embeddings), observability trace stores, and evaluation datasets.

    4. The Provider Trap

        Even if Anthropic (or another AI provider) signs a strict contract promising "Zero data retention" and "EU-only residency," that only covers their servers. It does nothing to protect or delete the copies your own system saved in your logs, vector database, trace tool, or eval spreadsheets.

    **EXAMPLE**

    A customer at an insurer submits a deletion request to erase their personal data, including their home address.

    What happens in a naive system: The insurer deletes the customer's profile from the main customer database and tells them "Done!" even though their home address is still sitting inside:
    - A vector index embedding built from their past support tickets.
    - An observability trace from last month's AI chat.
    - An evaluation fixture (test set) used by the QA team.

    Why the provider contract doesn't save them: The insurer points to their AI provider's "Zero Data Retention" agreement, but that only proves the AI provider didn't keep the address. The insurer's own copies still violate the customer's deletion right because they never built an inventory and provenance to track and wipe every copy.

6. How US federal cloud compliance (FedRAMP) directly affects which AI tools and vendors you can pick during system design.

    The Key Terms

    * FedRAMP: The US government program that reviews and certifies cloud services so federal agencies can use them safely.
    * Authorising Official (AO): The specific government agency official who has the power to approve a system to go live (Authorisation to Operate / ATO).
    * Authorisation Boundary: The fence around every component of your system that the AO has to review and approve. Crucially, it excludes external systems that are already separately authorised.
    * Impact Level: A rating (e.g., Low, Moderate, High) of how much damage a data breach would cause.

    Two Big Exam Distinctions

    * FedRAMP Authorisation $\neq$ Authorisation to Operate (ATO):
    Just because a cloud vendor or AI provider is "FedRAMP authorised" does not mean you can automatically launch it. FedRAMP only proves the vendor passed the FedRAMP review process. Your agency's Authorising Official still has to grant an ATO for your specific environment.
    * What stays stable vs. what can change:
    The definition of an authorisation boundary comes from foundational US standards (OMB Circular A-130 and NIST SP 800-37), not from FedRAMP itself. That means FedRAMP's own rules might evolve over time, but the agency's boundary rule is unlikely to change.

    Why Vendor Choice Is a *Design* Decision

    Every time your system sends federal data to a service (an LLM, a vector database, a trace vendor), you have to ask: Is this service already authorised at our required impact level?

    * **If YES:** It is a "separately authorised system." It stays **outside** your authorisation boundary, so you don't have to assess it from scratch.
    * **If NO:** That vendor gets pulled **inside** your authorisation boundary—meaning *you* now inherit the massive, expensive burden of getting that vendor assessed and approved by the agency's Authorising Official.

    That is why picking authorised vendors belongs at the **design stage**, not right before launch.

    **EXAMPLE**

    Suppose a health insurer builds an AI claims assistant for a US federal agency at a Moderate impact level. It connects to two vendors:

    * Vendor A (AI Model Provider): Already FedRAMP authorised at the Moderate impact level.
    * Result: It sits outside the insurer's authorisation boundary. (The agency's AO still has to grant the final ATO for the overall claims app, but they don't have to re-assess Vendor A from scratch.)


    * Vendor B (Third-Party Trace/Analytics Tool): **Not** FedRAMP authorised at Moderate.
    * Result: Because it carries federal claim data and isn't separately authorised, Vendor B gets dragged inside the insurer's authorisation boundary—stalling the launch while the agency's AO is forced to assess Vendor B's entire security posture.

