---
title: "From Hackathon to Open Source: Why I Extracted My LLM Switching Logic"
date: 2025-08-30
excerpt: "How a hackathon project that didn't win became a PyPI package with 6,000+ downloads in one month."
tags: [hackathon, open-source, ai, llm, python, openai, anthropic, gemini, watsonx, ollama, pypi, developer-tools, infrastructure, rag, mcp, cli-tools, machine-learning, artificial-intelligence, software-engineering, devops, cloud-computing, openstack, programming, coding, tech-story, startup, innovation, python-package, api, chatbot, automation]
keywords: "LLM switching, AI providers, hackathon to open source, PyPI package, developer tools, infrastructure AI, OpenAI alternatives, multi-provider LLM, AI CLI tools, machine learning tools, developer productivity, open source Python, hackathon project, artificial intelligence, software engineering story"
---

I was working on a hackathon project - an AI assistant that lets you chat with your infrastructure using RAG + MCP. Think of it as a live conversation with your entire cloud setup.

Started simple with Gemini, then added Watsonx and Ollama. Everything was hardcoded per provider.

Then I came across [this VentureBeat article](https://venturebeat.com/ai/ibm-sees-enterprise-customers-are-using-everything-when-it-comes-to-ai-the-challenge-is-matching-the-llm-to-the-right-use-case/) where IBM's AI VP discussed how enterprise customers use multiple AI providers - "the challenge is matching the LLM to the right use case." That hit me.

So I created a config system. Users could specify different providers and models, pass API keys through config files or environment variables, and set a default provider. Clean and flexible.

*Ref: [https://venturebeat.com/ai/ibm-sees-enterprise-customers-are-using-everything-when-it-comes-to-ai-the-challenge-is-matching-the-llm-to-the-right-use-case/](https://venturebeat.com/ai/ibm-sees-enterprise-customers-are-using-everything-when-it-comes-to-ai-the-challenge-is-matching-the-llm-to-the-right-use-case/)*

By the hackathon submission deadline, we supported Anthropic, Gemini, Watsonx, and Ollama. Same app, different brains.

The hackathon didn't go our way and was disappointing.

What I had was this solid provider-switching logic sitting in my codebase. It was too useful to let it die with the project. So I extracted it, cleaned it up, and open-sourced it as `llmswap`.

```python
# Instead of this mess in every project
if provider == "openai":
    from openai import OpenAI
    client = OpenAI(api_key=key)
elif provider == "anthropic":
    from anthropic import Anthropic
    client = Anthropic(api_key=key)
# ... repeat for 7 providers

# Just this
from llmswap import LLMSwap
llm = LLMSwap()  # Reads from config or env vars
response = llm.ask("Your question")
```

But that's just the SDK. llmswap also comes with CLI tools that became my daily workflow:

```bash
# Quick infrastructure questions
llmswap ask "Which logs should I check to debug Nova VM creation failure?"

# Interactive troubleshooting
llmswap chat

# Debug OpenStack errors
llmswap debug --error "QuotaPoolLimit:"

# Review infrastructure code
llmswap review heat_template.yaml --focus security
```

These CLI tools alone save me 10+ ChatGPT tab switches daily.

6,000+ downloads on PyPI in the first month.

Sometimes your best open source contributions come from hackathon leftovers. The project that didn't win can still help thousands of developers.

---

*PyPI: [https://pypi.org/project/llmswap/](https://pypi.org/project/llmswap/)*

*This is the first in a series about llmswap - upcoming posts will cover different use cases and implementation patterns.*