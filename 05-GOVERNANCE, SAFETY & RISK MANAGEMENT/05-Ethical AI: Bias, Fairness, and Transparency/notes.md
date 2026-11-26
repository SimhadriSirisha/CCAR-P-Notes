1. Key Terms

    - Bias: A gap in outcomes between different groups of people.
    - Protected attribute: A field recording a legally protected characteristic that you are not allowed to discriminate on (such as sex, race, or age).
    - Proxy: A different variable that accidentally carries the same information as a protected attribute (e.g., ZIP code acting as a stand-in for race or income).
    - Cohort: A specific group of people whose outcomes are measured together.
    - Fairness criterion: The rule that defines which outcome gap between cohorts counts as unfair.

    **EXAMPLE**

    A bank builds an AI to approve or deny personal loans:
    - Protected attribute: The applicant's sex or race (which the bank removes from the AI's inputs).
    - Proxy: The applicant's ZIP code (which is still in the inputs and strongly correlates with race in that city).
    - Cohorts: Group A (applicants from Neighborhood 1) and Group B (applicants from Neighborhood 2), measured separately.
    - Bias: Group A gets approved 80% of the time, while equally qualified applicants in Group B get approved 55% of the time—a 25% outcome gap.
    - Fairness criterion: The bank's rule stating that equally qualified applicants across cohorts must have equal approval rates—which officially flags that 25% gap as unfair.


2. Bias Is Measured in Outcomes
    - Deleting protected fields doesn't make an AI fair: Ordinary variables (working hours, occupation, application channel, address history, language style) act as proxies that carry the same information.
    - ML amplifies this: Modern models combine multiple ordinary fields (redundant encodings) to unintentionally reproduce discrimination.
    - The proxies are ordinary variables: working hours, occupation, application channel, address history and language register (the style and level of formality of a person's writing).
    - Even if your dataset has zero proxies and zero protected attributes, bias can still enter through the way humans collect data, design the goal, and build the product around the model.
        - how data is labelled: 
        
            A hospital builds an AI to predict which patients need extra medical care, using past healthcare spending as the label for "how sick a patient is." Because lower-income patients historically spent less money on doctors (not because they were healthier, but because they couldn't afford visits), the AI learns that they are "less sick" and denies them care.
        
        - builders' cultural assumptions:
        
            The engineers and product managers designing the system unconsciously assume their own culture, norms, or language style is the "standard" or "correct" one—and bake that assumption into prompts, rubrics, or training data.
        
            example: A global bank trains a customer-service AI to flag "high-risk" or "evasive" chats based on a rubric written by US-based designers who equate direct, blunt answers with honesty. Customers from cultures where polite, indirect communication is the norm get flagged as "evasive" at much higher rates even when their answers are completely truthful.
        
        - the goal the model optimises for: 
        
            Choosing a "best outcome" that silently favours one group over another.
        
            example: A telecoms company sets its support AI's objective to "minimise average handling time per chat." To hit that target fast, the AI rushes through or prematurely closes chats with elderly customers or non-native speakers who take longer to explain their billing issue, optimising the average metric while systematically failing the people who need help most.
        
        - inaccessible interfaces:
        
            The AI model itself might be 100% fair, but if the app, website, or form in front of it blocks certain people from using it, the harm happens before the model even runs.

            Example: A government agency launches an AI assistant to help citizens apply for housing benefits, but makes it voice-only (shutting out deaf or speech-impaired applicants) or smartphone-app-only with tiny low-contrast buttons (shutting out visually impaired or low-income applicants without modern phones). Those groups never even reach the model to get a fair outcome.

    - The rule: Never declare a system fair just because you hid a protected attribute. You must measure the actual outcomes across groups.

    **EXAMPLE**

    A telecoms company builds an AI to decide which customers qualify for a fast-track payment plan, and deletes the sex and nationality fields from the inputs.

    - Proxy bias (redundant encodings): Even without sex or nationality, the model sees part-time work history, paper forms submitted in-store, and second-language writing—and quietly denies payment plans to women and non-native speakers at twice the rate of other customers.
    - Interface bias (before the model acts): The payment-plan form only works on a smartphone app with small, low-contrast text and no screen-reader support—blocking visually impaired and elderly customers before the AI ever evaluates them.
    - The trap: If the team points to the deleted sex column and writes "Fairness verified" on the risk register, they have measured nothing. They must test the actual approval outcomes across cohorts.

3. To prove an AI is fair, follow three steps:

    - Measure at the cohort level, never just the aggregate: High overall accuracy (even with a tight confidence interval) is dominated by the majority and hides failure on smaller groups.
    - Pick cohorts from a harm hypothesis, not convenience: Ask "Who could this harm, and why?" Then track an observable variable (behavioural or channel) that represents that group.
    - Name and document your fairness criterion: Because you generally cannot satisfy all three at once, you must choose the one that best fits the harm and document your choice:
        - Equal error rates: The AI makes mistakes equally often for every group.
        - Equal selection rates: The AI approves/selects the same percentage of each group.
        - Calibration within groups: A given score (e.g., 80%) means the exact same real-world probability of the outcome in every group.

    **EXAMPLE**

    A telecoms company uses an AI to score job applicants and decide who gets an interview. Overall accuracy is 92% with a very tight confidence interval—but 90% of applicants are full-time workers, so that 92% mostly just measures full-time applicants.

    - Harm hypothesis: The team suspects the AI screens out part-time applicants (a behavioural variable representing a group disproportionately made up of women). They measure the part-time cohort separately instead of just slicing by whatever demographic data they happened to collect.

    - Choosing the fairness criterion (and why they differ):
        - Equal selection rates: Interview 30% of full-time applicants and 30% of part-time applicants.
        - Equal error rates: Make sure qualified part-time applicants aren't wrongly rejected (false negatives) any more often than qualified full-time applicants.
        - Calibration within groups: Make sure an AI score of "80/100" means an 80% chance of passing the final interview for both full-time and part-time applicants.

    Because all three mathematically conflict in most real-world datasets, the team picks equal error rates (to prevent wrongly screening out qualified part-time candidates) and documents that choice in their risk register.

4. Fix bias using the Remediation Ladder (cheapest and most durable first):

    - Fix the data: Rebalance group representation, check how labels were created, and verify which population the ground truth came from (Who were the actual people), If that ground truth only came from one narrow group of people, the model will perform badly or unfairly on everyone outside that group. because models trained on biased past decisions just repeat them.

    - Fix the framing: Change what the model is asked to decide, or question whether AI should be used for that decision at all (cheapest to answer before building). 

    - Constrain the decision: Narrow the AI's remit let it score and rank, while a human makes the final call on contested or affected cases.

    - Monitor continuously: Assign a live fairness metric, a threshold, and an owner because populations shift and AI decisions create feedback loops that entrench unfairness if left unwatched.

    Two traps that are NOT fixes:
    - Hiding the score: Removes the visible evidence, while the underlying harm stays intact.
    - Blaming the foundation model's training data: A weak excuse because framing, constraining the remit, and continuous monitoring are always the application team's responsibility.

    **EXAMPLE**

    A telecoms company finds its AI hiring tool is rejecting qualified part-time applicants:
    - Fix the data: Audit past hiring records and discover that 90% of historical "hired" labels came from full-time candidates; rebalance the training set and clean the labels.
    - Fix the framing: Stop asking the AI "Should we reject this person?" (or ask whether AI should screen resumes for this role at all).
    - Constrain the decision: Narrow the AI's remit—the assistant only ranks applications and highlights relevant skills, while a human recruiter decides who gets interviewed.
    - Monitor continuously: Track the interview rate gap between part-time and full-time applicants with a 3% threshold and an HR Compliance owner—so past hiring patterns don't keep feeding back into next year's model.
    What fails (Non-fixes): Hiding the AI's numerical score from recruiters while still letting it auto-sort the pile, or shrugging and saying "The base LLM is biased, so our engineering team can't do anything."

5. Transparency splits into three separate obligations, each aimed at a different audience:
    - Disclosure (to the user, at the point of use):

        Tell external users they are interacting with AI—at the start of the chat or in the interface—along with its limits and how to reach a human.

        For any High-risk AI Recommendation (like a loan decision or legal advice), explicitly tell the person that AI was used to produce it.

    - Explainability (to the affected person):
        
        Process-based explanation: Explains the governance and checks you followed overall i.e. Explains "How did you make sure this system can be trusted?" by explaining things like:

        - Design & data choices: What data the system was trained or grounded on, and how you checked it for bias.
        - Testing & validation: How you tested the AI's accuracy and fairness across cohorts before launch.
        - Oversight & accountability: Who signed off on the system, how human reviewers are kept in the loop, and how often it is audited.
        - Safety & compliance controls: How you follow laws (like GDPR or HIPAA) and monitor the system for drift.
        
            The rules, checks, testing, and human oversight your organisation followed to design, build, and run the AI system responsibly.

        Outcome-based explanation: Explains this specific decision in plain language, says whether a human was meaningfully involved, and gives a timely way to contest it. (Never substitute architecture diagrams or "the model decided.")

    - Traceability (to the organisation):
        
        Log the exact model version, prompt, and retrieved context behind every output—and retain those records long enough to investigate a dispute months later.

    **EXAMPLE**

    A telecoms company uses an AI assistant to help screen job applicants, and one candidate is rejected:
    - Disclosure: When the candidate opens the application portal, a banner at the top states they are interacting with an AI assistant, explains it can only screen basic eligibility (its limits), and provides an email/button to reach a human recruiter. Because employment is a High-risk Area, the rejection notice explicitly states AI was used in the process.
    - Explainability (Outcome-based): When the rejected candidate asks why, the company does not send a diagram of its RAG pipeline or say "the model scored you low." Instead, it sends a plain-language note explaining which specific job requirement was missing, confirming that a human recruiter reviewed the file, and giving a link to appeal within 14 days.
    - Traceability: Four months later, the candidate files a formal complaint. Because the company logged the exact model version, system prompt, and retrieved job-rubric text used on the day of the decision, the compliance team can reproduce exactly what the AI saw and answer the challenge.

6. Accountability stays with the organisation/deployer using the model and not the vendor.

    A real person owns the output: Every AI system needs a named human or role inside the organisation is accountable for its decisions. Automating a process does not transfer legal or ethical blame to the software.

    Vendor vs. Deployer split: 

    - Vendor's responsibility: Transparency and documentation for the foundation model it builds.
    - Your organisation's responsibility: Everything in your application—deciding whether outputs are appropriate before using or sharing them, and deciding where human review belongs. You cannot blame vendor or say "the model said so" to a regulator.

    ICO expectations:

    - Senior management must formally approve the AI system's intended use against the organisation's risk appetite (how much risk the company is willing to accept).
    - Accountability(who owns what) must be assigned from the outset (before launch, not after something goes wrong).
    - Whenever a system affects a person's rights, money, health, safety, or opportunities, the design must keep a human accountable and give the affected person a way to reach a human.

    **EXAMPLE**

    A telecoms company builds a resume-screening assistant using Anthropic's API:
    - Before launch (ICO rule): The company's senior management approves the screening tool against the company's risk appetite and formally assigns the Head of HR as the accountable owner from day one.
    - During operation (Commercial terms): The telecoms company—not Anthropic—is responsible for deciding whether the AI's candidate rankings are appropriate to use and where human recruiters must review them.
    - If a candidate challenges a rejection: The telecoms company cannot say "the model rejected you" or point to Anthropic's model card. The Head of HR owns those screening outputs, and the rejected applicant has a direct way to reach a human recruiter.