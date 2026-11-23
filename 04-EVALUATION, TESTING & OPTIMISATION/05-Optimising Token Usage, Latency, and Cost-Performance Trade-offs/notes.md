1. Optimisation (The Goal) means tweaking your AI system to achieve one of three things:
    - Use fewer tokens (word fragments the AI processes).
    - Reduce latency (how long the user waits for a reply).
    - Lower the cost (your monthly API bill).
    
2. A lever is a specific engineering trick you use to achieve that optimization. Examples of levers include: switching to a smaller AI model, removing unnecessary words from your prompt, or using caching (saving previous answers).

3. The Dominant Term (The Real Culprit) is the massive elephant in the room—the specific thing causing 80% of your pain. 
    eg: If a hotel chatbot takes 10 seconds to reply, you must find the dominant term. Is the AI typing too slowly? Or is the database taking 9 seconds just to find the document? If the database takes 9 seconds, retrieval is the dominant term.

4. The Skill (The Balancing Act) : Amateur engineers just pull levers randomly. A skilled engineer does two things:
    - Rule One: Measure First
    
    They identify the dominant term by stage, token type and call pattern using traces and usage data. 
    
    **Stage**: Your pipeline has three stages: 
    1) Intent Classification (figuring out what the user wants), 
    2) Database Retrieval, and 
    3) Final Answer Generation. 
    
    If the generation stage costs $5 a day, but the classification stage runs on every single keystroke and costs $500 a day, the Classification Stage is your dominant term.

    **Token Type**: You feed the AI a 10,000-word hotel manual (Input Tokens) to answer a user's question with a 5-word reply (Output Tokens). Because you pay for everything the AI reads, Input Tokens are the dominant term driving your bill. If you try to save money by making the 5-word reply even shorter, you are wasting your time, because the massive input is the real problem.

    **Call Pattern**: How often your system talks to the AI. If your code is poorly written and accidentally pings the AI every 5 seconds while the user is typing, the Polling Call Pattern is your dominant term for cost, not the actual conversations.

    Anthropic provides an official "Usage and Cost Admin API." This means instead of a human logging into a dashboard to check the bill, you can write a script that automatically downloads your exact token usage and costs 24/7.

    Pick the exact lever that fixes it. (If the database is slow, you don't swap the AI model; you fix the database).

    - Rule Two: Re-gate With Evals 
    
    They run the cheaper, faster system through the Golden Set (The Gate/ eval test). If the AI is now lightning-fast but starts giving illegal refund advice (dropping below the quality bar), your optimization failed.

    You can NEVER skip the quality bar. If skipped running the evals then it is a regression. 

5. Token and Cost Levers:

    - **Prompt caching**: suits repeated stable prefixes. A stable prefix is unchanged content at the start of each request: long system prompts, policy documents and tool definitions sent on every request. Cache reads are billed at a small fraction of the base input price. Cache writes are billed at a premium over the base input price. The cached content must be a prefix that stays identical byte for byte, so put stable content first. Select caching only when the workload has a repeated prefix.

    - **Context curation** sends only the fields that the task needs and excludes the rest of the file. The cheapest token is one that never enters the request. 

    - **Tight output contracts** cap output tokens, which are the expensive ones.

    - **Effort and reasoning budgets** Advanced models (like Claude 3.7 Sonnet) have the ability to "think" invisibly in a scratchpad before they output their final answer to the user. Because you pay for every token Claude generates, "thinking" costs money and adds latency. The Reasoning Budget is a literal setting in your API code where you tell Claude exactly how many tokens it is allowed to spend thinking. You are setting an "effort limit." Use low effort for simple classification and high-volume work. Use high effort for complex reasoning.

        Tuning effort means adjusting the API settings to tell the AI: "Don't think, just answer" for easy tasks, and "Think step-by-step" for hard tasks. It allows you to optimize cost and speed without having to downgrade your model tier.

    - **Right-sizing per stage** gives each pipeline stage the cheapest model that clears its measured bar and Premium model to hardest cases.

    - **Batch processing** runs work that is not time-sensitive asynchronously, at a discount. Batch processing as suited to large-scale evaluations, content moderation and bulk operations that do not need immediate responses. The cue is that no user is waiting. The words "overnight", "bulk" and "no one is waiting" point to batch. An interactive surface excludes batch. Nobody waits for the nightly review sort, so it can go to batch. 

6. Latency Levers:

    - **Streaming** reduces perceived latency. Users see output while the model generates it, and Anthropic recommends streaming for responsive interactive experiences. Total completion time is unchanged, so streaming is the right lever only where a human is watching.

    - **The smallest model** that reaches the measured bar reduces real latency. It is the first recommendation.

    - **Parallelising independent tool calls** removes serial waits.

    - **Caching** improves time-to-first-token for long stable prefixes, as well as cost.

7. Every lever spends something to save something. Streaming spends nothing but changes only perception. Batch spends turnaround time. Downsizing spends spare capability. Caching spends nothing only when the prefix is genuinely stable. Name the spend and the saving, with eval evidence that the spend is affordable. So always propose a "balanced cascade" using the strict "spend, save, and eval" formula and reject the "tempting wrong answer" like "Let's immediately switch all our traffic to the new Claude Haiku model. It will instantly cut our bill by 80%!" which doesn't has any No named cost, No eval rerun.

8. Example:

    Here is a real-world scenario combining both concepts: rejecting the "tempting wrong answer" and proposing a "balanced cascade" using the strict "spend, save, and eval" formula.

    **The Setup**
    The hotel chain uses the premium Claude 3.5 Sonnet model to process all customer refund emails. The monthly bill is $10,000. The business requires a minimum **Quality Bar of 95% accuracy** on refunds.

    **The Tempting Wrong Answer (The Trap)**
    The Finance Director says: *"Let's instantly route all 10,000 emails to the cheap Claude 3.5 Haiku model. We will save $8,000 this month! Ship it today!"*

    This is the trap. It promises a massive saving, but it has no eval rerun and doesn't name what is being "spent" (the penalty). If you ship this, you might accidentally start illegally denying valid refunds, causing a massive lawsuit.

    **The Balanced Answer (The Escalation Cascade)**
    You step in as the AI engineer and propose a **Right-Sizing Cascade** instead: The cheap model will read every email first. If it is a simple 1-night cancellation, it handles it. If the email contains complex group-rates or international currency, it automatically escalates the ticket to the premium model.

    You present your proposal to the Finance Director using the exact formula: **Name the spend, name the saving, and provide eval evidence.**

    * **The Saving:** "By using the cascade, we will save 75% on token costs, and 80% of our customers will get their refund approved in 2 seconds instead of 10 seconds."
    * **The Spend:** "We are spending *turnaround time* on the hard cases (the 20% of complex emails will take slightly longer because they have to be read by the cheap model, fail, and then get passed to the premium model). We are also spending *spare capability* on the easy cases."
    * **The Eval Evidence (Proving it is affordable):** "I did not just guess. I ran the new Cascade code through our offline Golden Set of 500 historical emails.
    * The old Premium-only system scored **99%**.
    * The new Cascade system scored **96.5%**.
    * Because our business Quality Bar is **95%**, that 2.5% drop in accuracy is an **affordable spend**. We safely kept our quality above the line while achieving massive cost savings."



8. The Cascade is the balanced engineering solution, and use hard mathematical evidence to prove the trade-off was safe to ship.

