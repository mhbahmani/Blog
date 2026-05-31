# MohammadHosein's Treasury

[![ci](https://github.com/mhbahmani/Blog/actions/workflows/deploy.yml/badge.svg)](https://github.com/mhbahmani/Blog/actions/workflows/deploy.yml)

A personal blog built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/), deployed to [blog.mhbahmani.ir](https://blog.mhbahmani.ir).

## Local Development

```bash
pip install -r requirements.txt
mkdocs serve
```

## Prompts
This prompt is being used to generate some documentation out of a chat with LLM:


> Act as an expert Technical Writer and Senior Systems Engineer. I want you to review our entire conversation above and generate a comprehensive, professional technical documentation based on the issue we just fixed. 
> 
> The target audience is other engineers who need to understand exactly what happened, how we fixed it, and the concepts behind it. **Crucially, do not truncate or summarize configurations or logs. I need the full context included.**
> 
> Please format the documentation using Markdown and include the following sections exactly as described:
> 
> 1. **Context & Initial State:** Explain the background of the system. What were we trying to achieve? 
>     * **Initial Configuration:** You MUST output the **FULL original (broken) configuration** that I provided at the start of the chat. Do not just summarize it; provide the complete code block.
> 2. **The Problem & Symptoms:** 
>     * Explain the exact problem and error we were experiencing.
>     * **Error Logs & Output:** Explicitly include the exact error logs, terminal outputs, and debugging commands I shared that demonstrate the 'broken state'. 
> 3. **Troubleshooting Investigation:** Detail the investigation process. What debugging tools or commands were used? Explain what the *expected* result was versus the *actual* result shown in the logs above.
> 4. **The Solution & Applied Changes:** Step-by-step instructions on what we changed to fix the issue and *why* these specific changes solved the problem.
>     * **Final Working Configuration:** You MUST output the **FULL, corrected configuration**. Do not just show the diff or the changed lines; provide the complete, working configuration block so a reader has the full picture.
> 5. **Deep Dive into the Architecture/Concepts:** Identify any major or complex configuration concepts related to our setup or this fix. Even if we didn't explicitly discuss them in our chat, provide a brief, educational explanation of how they work under the hood. 
> 6. **Verification of the Fix:** How can a reader verify the system is working? 
>     * **Success Logs:** Mention the specific logs, command outputs, or signs that represent the 'fixed state'.
> 
> Please ensure the tone is educational, objective, and highly detailed. Remember: Full configurations and exact log outputs are required
