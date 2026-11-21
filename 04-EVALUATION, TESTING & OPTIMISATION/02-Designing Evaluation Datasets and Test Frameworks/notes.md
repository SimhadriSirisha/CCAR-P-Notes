1. An evaluation dataset is the set of cases that a system is tested against. What a dataset contains decides which risks it covers. Each population(a distinct group or category of test cases) protects against a different failure.

2. There are 4 cases of populations:
    - **Representative cases** are the "normal" cases, but they are much better than basic "happy-path" cases. These cases are pulled from real-world reality. They represent the actual, everyday, slightly messy inputs that make up 90% of real user traffic.
    - **Edge cases** are the inputs at the boundaries where behaviour changes. They include irrelevant data, overly long inputs, poor user input, and ambiguous.
    - **Adversarial cases** are the concrete tests for the safety and security metrics. They include injection attempts, leading questions(A leading question embeds a false premise, fake fact, or harmful assumption directly into the prompt to see if the AI takes the bait) and malformed input.
    - **Regression cases** are the tests created from past mistakes. Imagine your AI makes a stupid mistake in production: a customer asks for a "receipt," and the AI accidentally deletes their account. You realize there was a bug, and you write new code to fix it. To make sure you (or another developer) don't accidentally bring that bug back six months later, you take that exact "receipt" request and put it in your permanent test vault. Now, every time you update the AI, it must pass that specific test. It prevents the system from "regressing" (going backwards) and repeating old failures.

3. A golden set is a dataset that is trusted for comparing scores over time. These 3 properties make a golder set trustworthy:
    - **Versioned**: Any change to the dataset, we should mark as new version without overwriting the old file. This ensures you always have a historical record of exactly which questions were in the test at any given time. A score is comparable to last month's only if the set is same. If the set change then recorded.
    - **Held-out**: This dataset is held out of the prompt, training data and tuning data.
    - **Refreshed**: A test set must be updated when a major business change happens (like launching a mobile app, because you need to start testing mobile-related questions). However, even if your business doesn't change, user behavior and language naturally drift over time. Therefore, an engineering team cannot just wait for a new product launch; they must make a deliberate schedule (a "design decision") to pull in fresh real-world data every few months so the AI isn't being tested on outdated traffic.

4. LLM outputs are non-deterministic. The same report can give a different ticket on each run, so small samples mislead. To trust your AI's score, you must use strict statistics. Here are the 4 rules:
    - Report standard error and confidence intervals (Stop using bare scores)
        - The Concept: Never just say, "The AI scored 85%." Because of AI randomness, that number isn't perfectly solid.
        - The Fix: You must report the margin of error. Say instead: "The AI scored 85%, and we are 95% confident the true score is somewhere between 82% and 88%."
        - What does it tells about size: They prove whether your current test size is actually large enough to overcome the AI's variance; if your confidence interval is too wide, it means your sample size is too small to trust and must be increased.
    - Use clustered standard errors (Don't cheat your data)
        - The Concept: If your test set has 100 questions, but 10 of them are from the exact same frustrated customer talking about the exact same issue, you don't really have 100 independent tests.
        - The Fix: You have to group ("cluster") those related questions together in your math. If you treat them as completely separate, your math will falsely tell you that you are more certain about your AI's performance than you actually are.
        - What does it tells about size: They prevent you from falsely inflating your test size with highly repetitive questions, forcing you to increase the size of your test with truly independent cases before you can claim the AI is safe.
    - Resample and average per question (Smooth out the randomness)
        - The Concept: Because the AI might give a great answer on Monday and a bad answer on Tuesday to the exact same prompt, one test isn't enough.
        - The Fix: Run the exact same test question through the AI multiple times (e.g., 5 times), and average the score for that specific question. This proves the AI is consistently good, not just lucky once.
        - What does it tells about size: It directly increases the size of your evaluation by forcing you to run the exact same question multiple times to smooth out the AI's natural variance (randomness) before trusting the score.
    - Run a power analysis (Do the math before you test)
        - The Concept: How many test cases do you need to prove your new AI prompt is 5% better than your old one? You can't just guess.
        - The Fix: A "power analysis" is a statistical formula you run before testing. It tells you exactly how many test cases you need to mathematically prove a difference.
        - What does it tells about size: It is the mathematical formula you use before testing to calculate the exact sample size needed, ensuring that high-risk or highly variable tasks are given a mathematically large enough test set to catch dangerous failures.

5. Mixed Methodologies: The Pairing Rule - It establishes a strict hierarchy for how you should grade your AI, using the right tool for the right job to balance cost and accuracy.
    - Code first (Deterministic): If a simple software script can check the answer (e.g., "Is the output exactly 10 digits?"), use code. It is instant and free. They are cheap, fast and objective. Always automate when possible, and prioritise volume(It means testing a massive number of cases is more important for finding bugs than having the absolute most perfect, detailed grading method on a tiny number of cases). More questions with slightly less informative automated grading are better than fewer hand-graded questions.
    - LLM-as-judge: scores open-ended quality, such as tone, completeness and faithfulness, at scale. If the answer is subjective (e.g., "Is this summary well-written?"), code can't read it. Use a second AI to grade it. . Always evaluate the output with a different model from the model that generated the output. The judge's rubric has explicit scoring anchors (a written description for each score), and the rubric is versioned like code. 
    - Humans review (The Calibrator/Auditor): It is the reference standard for grading. Humans are slow and expensive, so you only use them to grade a small sample to ensure the AI Judge is scoring things correctly(calibrating), and to make the final call on really confusing or dangerous edge cases (High-stakes). Use human review to calibrate the judge, to audit samples, and to adjudicate high-stakes or ambiguous cases.

6. The judge is itself a model with failure modes. It scores a system that it resembles. Standard practice is to validate the judge against human labels on a sample before you trust its scores. Until then, the judge's scores are not evidence, however precise they look.

7. More cases from the same population confirm the demo more strongly. The recurring incidents happen in edge, adversarial, and regression cases, which the set never contains. Add those populations rather than more clean variants.