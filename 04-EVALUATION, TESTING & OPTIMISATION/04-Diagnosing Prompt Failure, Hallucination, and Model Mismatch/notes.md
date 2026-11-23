1. Diagnosis is a fixed method that you follow, not a search for a ready answer to findout why a Claude system fails before you change anything.

2. The Diagnostic Order:
    1. Trace and Reproduce: A trace records the inputs to the request (prompt version, retrieved context, tool results) and the output. To confirm a pattern across cases (prompt failure, hallucination & model mismatch), we have to run several times, as we can't find from single run because the system is non-deterministic. So one bad output proves nothing.
    2. Isolate stage: Finding out the failure stage in a pipeline using the evidence got from step 1. E.g.: The chain's errors began with the rate load, and the model and latency did not change. This evidence points to retrieval and indexing before anyone says the word "hallucination".
    3. Classify against the signatures (the typical symptoms of each class): prompt failure, hallucination (grounding or generation), model mismatch. Each has a recognisable signature
    4. Run the cheapest decisive test to confirm thw hypothesis
    5. Always fix at the failing layer rather than directly jumping to the conclusion of easiest fix without trying step 2-4. The reflex fixes, "upgrade the model" and "rewrite the prompt harder", are conclusions without a diagnosis.
    6. Create regression cases from the failing inputs, so the diagnosis remains after the incident ends.

    So the cheetsheet is : `evidence before conclusions, stage before failure class, cheapest decisive test before any fix.`

3. There are 3 different failure clases. Each has its own signature. The 3 failures are :
    1. A Prompt failure, the instructions are the problem.
    2. A hallucination is confident fabrication.
    3. A model mismatch, the task exceeds the model tier.\

|Failure case| Signature| When failures cluster | The decisive test | Anthropic's consistency guidance methods | Cheetsheet of List of technique's| example | Wrong fixes
|-|-|-|-|-|-|-|-
|Prompt Failure| format drift, violated output contracts (outputs that break the specified format), rules ignored or applied inconsistently, and behaviour varying with phrasing on identical inputs.|When the instructions do not fully specify the behaviour.| a clarified, structured prompt (explicit rules, delimited data, an exact-format example) fixes the failing cases while everything else stays constant.|1. Specify the output format precisely. 2. Constrain with examples. 3. Use structured outputs where schema conformance must be guaranteed. | clarity, then examples, then reasoning | User A asks: "What is the pet fee?" -> AI replies: "$50." User B asks: "How much for a dog?" -> AI replies: "I am sorry, I do not have pricing information." The core question is exactly the same, but the slight change in vocabulary completely breaks the AI's ability to answer.| A model upgrade is the wrong fix. A larger model also follows ambiguous instructions, at a higher cost.|
|Hallucination: Grounding|the context was wrong, stale or missing, and the model answered faithfully from it.| Wrong answers correlate with events in the corpus, the document collection that retrieval searches (a refresh, a migration, the chain's rate load), or with degraded retrieval-quality signals. | re-index (rebuild the search index), re-embed (compute the document embeddings again), repair retrieval. | | The pipeline fed the AI bad data. Fix the database (re-index, re-embed). | User: "Does the Downtown Hotel have a swimming pool?", AI: "Yes, we have a pool! It is located right next to the bar, and it costs $5 per hour to play." The AI accurately read the document it was given. The root cause is a degraded retrieval-quality signal: your search engine's embeddings are so weak that it cannot tell the difference between "a swimming pool" and "a billiards pool table."| Fixing the prompt imstruction. No instruction fixes the context.
|Hallucination: Grounding | The trace shows the right passages, and the answer contradicts or exceeds them. | | The fix is a set of grounding techniques i.e. Prompt-level grounding and measure faithfulness -> "We tested 500 cases. Our faithfulness metric mathematically improved from 92.4% to 98.1%."| 1. Explicitly allow the answer "I don't know". This option drastically reduces false information. 2. Extract  word-for-word quotes first (the right documents to scratchpad first) and then generate answer, well suited for long documents. 3. Instruct AI to add citation to any finding and if found finding without citation then retract (delete) 4. Restrict answers to the provided text. 5. Use chain-of-thought verification (Claude explains its reasoning step by step before the final answer, The act of "thinking" logically guides the AI to the correct conclusion.) or best-of-N verification (run the same prompt several times and compare the outputs).| The AI lied about good data. Fix the AI (use grounding prompt techniques). | On an older rate, the retrieved passage says non-refundable, yet the assistant says that the rate is refundable. | "Every time the AI hallucinates, I'm going to rewrite the prompt," they will be wrong 50% of the time, because half the time the prompt is perfectly fine and the database is the thing that actually needs fixing.
| Model Mismatch | uniformly mediocre quality on the hard cases across prompt variants. | Failures persist with a clean prompt and verified-correct context. Quality changes with task difficulty, not with phrasing.| Run the failing set on a stronger tier. If the stronger tier meets the quality bar, the gap is capability. | 1. Benchmark with your actual prompts and data, to get a baseline. 2. Compare accuracy and edge-case handling. 3. Weigh cost. | Mismatch happens in both directions. Consider a premium tier that is far above a simple task. The same diagnosis and the opposite fix | User: "I am booking 3 rooms for 5 nights. Two rooms are on the standard $100/night rate. The third room is standard for the first 3 nights, but I want to upgrade it to a $250/night suite for the last 2 nights. I have a 10% loyalty discount, but the rules say discounts do not apply to suite upgrades. What is my exact total?"; AI: "$15" which is mathematically wrong. ||

4. Explanation with example (All together):

    User: "I am booking 3 rooms for 5 nights. Two rooms are on the standard $100/night rate. The third room is standard for the first 3 nights, but I want to upgrade it to a $250/night suite for the last 2 nights. I have a 10% loyalty discount, but the rules say discounts do not apply to suite upgrades. What is my exact total?"

    AI: "$1100"

    Diagnosis steps:
    
    1.  Check the Context (Grounding): You look at the trace. The database perfectly retrieved the $100 rate, the $250 rate, and the exact rule about loyalty discounts. It is not a grounding failure.
    2. Check the Prompt (Generation): Your prompt is highly structured. You even added a "Chain of Thought" command telling the AI to calculate the standard rooms first, then the suite, then apply the discount. It is not a prompt failure.
    3. You take the exact same prompt and the exact same database rules, but you run it through a premium, heavy-duty AI model tier. The premium model instantly calculates the correct total ($1,600).

5. The two specific AI behavioral problems: Inconsistency (the AI changes its mind) and Bias (the AI treats certain users unfairly).

    **Inconsistency**: Because AI is non-deterministic, it can be flaky. This can be of 2 types:

    1. Divergent Cases: You ask the exact same question 5 times. 3 times it says "Yes," 2 times it says "No." To fix this, you don't guess, you use the exact same diagnostic method we learned earlier. You pull the logs (trace) for the 5 runs, group them together (cluster), and look for the hidden variable that caused the 2 failures.

    2. Self-Contradiction: This is worse than a divergent case. This is when the AI argues with itself in a single reply. (e.g., "Our hotel is 100% pet-friendly. Therefore, you cannot bring your dog."). The text explicitly states that if you catch the AI doing this, it is such a massive red flag that you must immediately turn that specific chat into a permanent test (a regression case) for your Golden Set.

    **Bias**: AI can easily hide inside high average scores. This appears as a detection task: disaggregate (split) eval results across user groups and content types.
    
    eg: Imagine your hotel chatbot scores a 90% overall accuracy rate on customer service tests. That looks like a massive success. You split that final score into different buckets based on user groups (e.g., separating the scores of English speakers vs. Spanish speakers, or corporate accounts vs. standard guests).
        
    - When you split the data, you discover the AI is 100% accurate for English speakers, but only 40% accurate for Spanish speakers.
    - Because English speakers make up the vast majority of the traffic, they dragged the average score up to 90%.

    You must split the data to prove it is fair to everyone.