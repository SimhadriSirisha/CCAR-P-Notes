1. Launch-time evals prove readiness("readiness" simply means the AI system has passed the quality bar and is safe, accurate, and reliable enough to be unleashed on real customers) once. 

    Here, only once Because before you ship, Running your offline Golden Set tests proves the AI is ready to be launched right now. But it only proves it "once"

    Monitoring proves that readiness stays true in production over time.

    In short: Evals give you the green light to launch. Monitoring proves you don't need to pull the plug later.

2.  A green uptime dashboard proves that the service is up. It does not prove that the service is right. A system can be fully up, fast on average and confidently incorrect. So quality monitoring is a separate discipline. It has its own scored signals. So, Uptime does not measure wrongness.

3. Drift is a change after launch that moves quality. This is the silent degradation of your AI system that happens after you launch it. Unlike traditional software bugs that happen when you push bad code on a release schedule, AI drift happens unpredictably because the real world changes.
    The 4 Causes of Drift:
    - Users change what they ask (input distribution shift).

        eg: Your hotel chatbot normally receives short questions like "What time is checkout?" or "Is the pool open?" Suddenly, a hurricane hits the city. Overnight, 95% of your traffic shifts to panicked, complex questions about force-majeure refunds, emergency evacuations, and flight cancellations. The AI starts failing because the "distribution" (the mix of topics) shifted to something it was never tested on.

    - The underlying database gets updated (corpora refresh).
    - The AI model provider updates their systems (model migration).
    - Engineers add too many conflicting rules over time (prompts accumulate edits).

4. How to catch "drift" ? 

    When you launch, your offline tests (evals) prove the system is at 95% quality. But two months later, users start asking new questions, or the database gets messy. Your quality silently "drifts" down to 80%, but because you only run formal evals when you release new code, you have no idea your live system is currently failing.

    To fix this, we need **Continuous Evaluation**, instead of testing only at launch, you automatically grade a random sample of live user chats every single day. Quality becomes a live heart-monitor graph (a trend line) instead of a one-time physical exam.

    How to build this live grading system:

    1. Programmatic Checks (For Black-and-White Rules) 
        
        You write standard code (Python/Regex) to check deterministic, hard rules. It is cheap and instant.

    2. The Calibrated Judge (For Complex Reasoning)

        You cannot use a Python script to check if the AI's tone was polite, or if its refund math was logical. For fuzzy concepts, you use an LLM-as-a-judge. You use a powerful model (like Claude 3.5 Sonnet) to read the cheap model's live chats and grade them 1 to 5. (It is "calibrated" because you previously proved this AI judge gives the exact same grades a human manager would).

    3. Prioritize Volume through Automation
        
        Anthropic's guidance is clear: do not use humans to read daily logs. By automating the grading (using programmatic checks and the LLM Judge), you can afford to grade 10,000 live chats a day. Volume is what reveals the truth.

    4. Security and Cost Monitoring
        
        Continuous evaluation isn't just for quality. You use it for two other things:
        - Cost Tracking: Using the API to ensure the new AI isn't suddenly burning through your budget.
        - Security (Guardrails): Having the LLM Judge constantly scan live outputs to see if a malicious user successfully "jailbroke" or injected bad instructions into your bot.

5. Leading Indicators vs. User Complaints
    - User Complaints (The Lagging Indicator): Waiting for users to complain is the most expensive way to monitor an AI. A complaint means the failure has already reached the customer and done damage. It is a signal that arrives too late.
    - Leading Indicators (The Check Engine Light): These are internal metrics that spike the exact second the system starts drifting, long before a human user notices or complains.

6. Every leading indicator maps directly to a specific stage in your pipeline. When an alert fires, you don't have to guess where the bug is, the metric tells you exactly where to look so you can start your diagnostic loop.

    - Token spend per request: inputs grew, or a prompt edit changed behaviour
    - Refusal rate: traffic shifted, or a policy prompt interacts badly with new inputs
    - Tool-failure rate: an upstream API changed
    - Input-length distribution: a new client integrated
    - Retrieval-quality signals: a corpus refreshed

7. Alerts fire on guard metrics at tail percentiles. Averages hide the problems, so alerts on averages fire late or never. Alerts on everything send the team so many alarms that it ignores the real alert. To keep the real alerts visible, the team deliberately chooses what not to alert on. The firm sets tail thresholds from its baselines. It also names the signals that do not alert anyone.

8. When an alert fires, the incident loop runs:

    - Alert
    - Traces
    - Diagnose
    - Fix at the failing layer
    - Create regression cases
    - Re-baseline, if the baseline legitimately moved

9. It turns quality into a trend line for the traffic arriving now, not the launch distribution. Sampled automatic scoring shows what declined and when. Drift indicators show what moved. The baseline comparison shows where the change is, for a diagnosis. 

10. **EXAMPLE**
    
    **The Setup:**
    
    The hotel chatbot has been running smoothly for months. On a random Wednesday, the marketing team silently uploads a massive new "Summer VIP Spa Catalog" to the chatbot's internal database (a corpus refresh). They do not tell the engineering team.

    Here is how the 6-step incident loop catches and fixes the drift before users even notice.

    **1. Alert**
    
    At 2:00 PM, an automated alert fires on a tail percentile. The p99 "Token spend per request" leading indicator spikes. The system is suddenly spending $0.50 per chat instead of the baseline $0.05. No users have complained yet, but the alarm is ringing.

    **2. Traces**
    
    The on-call AI engineer does not guess what is wrong. They immediately pull the API traces for the last 10 minutes of live traffic.

    **3. Diagnose**
    
    The engineer follows the strict Evidence -> Stage -> Failure Class -> Test -> Fix framework.

    * **Evidence:** The trace shows the AI is processing 15,000 input tokens per turn instead of the normal 500.
    * **Stage:** The engineer looks at the pipeline and sees the Generation stage (the prompt) hasn't changed. The anomaly is in the **Retrieval stage**.
    * **Failure Class:** Degraded retrieval-quality signal.
    * **Decisive Test:** The engineer reads the raw text the database is sending to the AI. They see that for a simple question like *"What time is checkout?"*, the database is frantically pulling all 50 pages of the new "Summer VIP Spa Catalog" and feeding it to the AI.

    **4. Fix at the failing layer**
    
    The engineer does *not* fix this by changing the AI's prompt to say *"Ignore the spa catalog."* That is fixing the wrong layer. They go upstream to the database layer and edit the retrieval algorithm, capping the search so it only grabs the top 3 most relevant paragraphs instead of the whole book.

    **5. Create regression cases**
    
    To make sure this never happens again, the engineer takes 5 of the failed live chats (e.g., *"What time is checkout?"*) and permanently adds them to the offline Golden Set (the eval). Now, if a future engineer breaks the retrieval algorithm again, the offline test will fail, and the bad code will be blocked from shipping.

    **6. Re-baseline**
    
    The fix is shipped, and token spend drops back down. However, the marketing team *does* want the bot to use the new Spa Catalog when users actually ask for it. This means the system's baseline token spend is legitimately going to be slightly higher moving forward. The engineer updates the p99 alert threshold from 1,000 tokens to 1,200 tokens. If they skipped this step, the alarm would falsely fire every single day, causing alert fatigue.