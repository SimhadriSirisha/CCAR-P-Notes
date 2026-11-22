1. An offline eval tests a candidate change(new version of the AI) on a fixed dataset(Golden Set) before users see it. An online A/B test splits live traffic between the current baseline and the candidate.

2. A two-step process you must follow before releasing a new AI model to your actual users. You must pass "The Gate" and then survive "The Validation."
    1. The Gate (Offline Evaluations):
        - What it is: This is your private "lab test." You run the new AI against your Golden Set (historical data, past tickets, simulated questions).
        - Why it is a "Gate": It is a hard checkpoint. If the AI fails this test, the gate stays closed. The model is rejected and sent back to the developers.
        - "Necessary but not sufficient": Passing this test is required, but it does not mean the AI is ready for everyone. It just means the AI is good at taking practice tests. It has not yet survived contact with unpredictable, messy real-world humans.
    2. The Validation (Online Experiments / A/B Testing)
        - What it is: This is your real-world "road test." Once the AI passes the Gate, you release it to a small percentage of live users (e.g., 5% of traffic).
        - How it works: You run an A/B test. Group A gets the old, live baseline AI. Group B gets the new candidate AI. You compare their real-time metrics (Are users happier? Are response times faster?).
        - Why it is necessary: A test set only approximates reality. Online validation proves if the AI actually works on live, unexpected user behavior.

3. Anthropic's model-selection guidance sets the same order for model changes:
    - Build benchmark tests specific to your use case.(Private testcases specific to the company - called offline gate)
    - Test with your actual prompts and data.
    - Once tested, look at the price vs. performance. (Is this new AI 2% smarter but 10x more expensive? If so, don't launch it).

4. A rollback plan is an emergency "undo" button to revert to the old AI if the new one breaks in production. Having an undo button does not excuse you from testing. You cannot skip the Offline Gate or the Online A/B Test just because you can quickly revert a disaster. You must still earn the right to launch by passing both stages.

5. Only an isolated change can be credited with a result. This means you can only know why a metric improved or dropped if you only changed one thing at a time. 

    If the prompt is rewritten and the model is upgraded together, 
    - a win cannot be credited:
        If you upgrade the AI model and rewrite the prompt on the same day, and the system improves, you have no idea which change actually caused the win. Did the new model fix a bad prompt, or did the new prompt fix a bad model? Because the true cause is unknown, a future developer might later delete the exact line of code that caused the improvement without realizing it.
    - the causes of a loss cannot be separated:
        If you bundle three changes together and the system's accuracy drops, you are flying blind. Your team will waste hours guessing which of the three updates broke the AI, instead of instantly knowing the single culprit.

    So release the first change, establish the new baseline, then test the second change against that baseline. This feels slower and costs less. Wins with no known cause get reverted by accident, and losses with no known cause get debugged without evidence.

6. Non-deterministic outputs make measurements noisy. So any result needs more evidence before you declare it. 
    e.g:

    If you ask a standard software script to calculate 5 + 5, it outputs "10" every single time (deterministic). There is zero noise.
    If you ask an AI, "Write a polite greeting," it is non-deterministic (it changes its mind).
    Run 1: "Hello, how can I help you today?" (Judge score: 10/10)
    Run 2: "Greetings." (Judge score: 5/10)
    Run 3: "Hey there!" (Judge score: 7/10)
    That random bouncing of scores (10, 5, 7) on the exact same question is the "noise." Because the data is noisy, you cannot run the test just once and declare, "The AI is a 10/10!"

    So, Anthropic's statistical approach to evals:
    1. Report the standard error of the mean and confidence intervals, not bare scores
    2. When two variants answer the same questions, use a paired-difference test.

        Instead of just comparing the final average score of your Old AI (80%) against your New AI (85%), a paired test compares them head-to-head on the exact same questions.
Imagine Question #42 is an incredibly confusing, high-difficulty legal prompt. If you just look at the overall average, you hide the details. A paired-difference test looks at Question #42 and says: "The Old AI scored a 2/10 here, but the New AI scored an 8/10 on this exact same brutal question."
Because question difficulty affects both AIs equally, comparing them side-by-side (pairing them) removes the "difficulty" factor from the math. It proves the new AI is actually smarter, not just lucky on the easy questions.
    3. Instead of just comparing the final average score of your Old AI (80%) against your New AI (85%), a paired test compares them head-to-head on the exact same questions.
Imagine Question #42 is an incredibly confusing, high-difficulty legal prompt. If you just look at the overall average, you hide the details. A paired-difference test looks at Question #42 and says: "The Old AI scored a 2/10 here, but the New AI scored an 8/10 on this exact same brutal question."
Because question difficulty affects both AIs equally, comparing them side-by-side (pairing them) removes the "difficulty" factor from the math. It proves the new AI is actually smarter, not just lucky on the easy questions.

    4. Run a power analysis first

7. Standard deployment practice exposes risky changes in stages. Model-version migrations need this most, because behaviour can change broadly and silently.
    So, Here are the four stages of safely launching a new AI:

    1. Shadow Run (The "Ghost" Test)

        Upgrading to a new AI model is highly dangerous because the AI might start making new, weird mistakes you didn't predict. To test it safely, you run the new AI in the "shadows." When a real customer sends a message, the Old AI replies to the customer normally. But secretly, the New AI is also given the message, and its answer is saved to a private file.
        
        The Benefit: Developers can compare the New AI's answers to the Old AI's answers using actual, live customer questions, with absolutely zero risk of the customer seeing a bad answer.

    2. Canary

        Once the new AI passes the shadow test, you do a "canary".
        You turn the New AI on for a tiny slice of real traffic, for example, just 1% of users. You watch your dashboards (guard metrics) closely. If the 1% of users start getting angry or the AI breaks, the blast radius is very small.

        guard metrics like:
        - Human Escalation Rate: The Old AI has a 10% rate of users clicking "Talk to an Agent." whereas new AI spiking to a 45% escalation rate, it means the new AI is frustrating users or giving useless answers.
        - P95 Latency: The Old AI takes 1.5 seconds to reply whereas new AI is taking 8 seconds, the new model is too slow and users are abandoning the chat.
        - Refusal Rate: The Old AI refuses 1% of prompts whereas new AI's refusal rate spikes to 20%, the safety filter is over-tuned and it is mistakenly blocking normal customer questions.

    3. Clean Rollback (The "Undo Button")

        To run these tests, you must have the ability to instantly switch the system back to the old version with one click. An experiment that cannot be reversed cleanly is a deployment. This means if your code is so messy that you cannot easily hit "undo", you aren't actually running a safe test. you are just blindly launching a permanent change and crossing your fingers.

        - Shadow Run Rollback: Protects infrastructure/budget (User experience is already protected by the shadow architecture).

        - Canary Run Rollback: Protects infrastructure/budget + User experience (Because both are actively exposed to the new code).

    4. Setting a New Baseline

        Once the canary test succeeds and you give the New AI to 100% of your users, the migration is complete. The New AI is now your standard. If you decide to make another update next month, you must compare that future update against this New AI.

8. The Standard Loop (What teams usually do):
    - Measure & Hypothesise: Look at current AI performance and guess how to improve it.
    - Change one thing: Isolate the variable (as we discussed earlier).
    - Eval offline: Run it through "The Gate" (your Golden Set).
    - A/B test online: Run it in a Canary release, watching success metrics (e.g., higher resolution rate) and guard metrics (e.g., cost, server crashes).
    - Ship or Revert: If it wins, launch it. If it loses, roll it back.

9. Make the losing variant's failures into regression cases.

10. Never trust your eyes on a small sample. Trust the math on a large sample. Steps to follow :
    - Power Analysis: Calculates the sizing you need.
    - Standard Error: Grades whether your sizing was big enough.
    - Paired-Difference: Reduces the sizing you need.
    - Resampling: Multiplies the sizing per question.
