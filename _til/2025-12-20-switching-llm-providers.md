---
title: "Switching LLM Providers Without Rewriting Code"
date: 2025-12-20
tags: [python, llm, api]
---

Kept switching between OpenAI and Anthropic APIs for different projects. Got annoying having to change imports and API calls each time.

```python
# Instead of this mess
if provider == "openai":
    from openai import OpenAI
    client = OpenAI(api_key=key)
elif provider == "anthropic":
    from anthropic import Anthropic
    client = Anthropic(api_key=key)

# Just this
from llmswap import LLMSwap
llm = LLMSwap()  # Auto-detects from env vars
```

Built [llmswap](https://pypi.org/project/llmswap/) to solve this. Now I can switch providers just by changing environment variables.