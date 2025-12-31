---
title: "ABC Module Makes Abstract Classes Actually Work"
date: 2025-09-10
tags: [python, design-patterns, oop]
---

Was building llmswap and needed to support multiple AI providers (OpenAI, Anthropic, Gemini, etc.). Started with just `class BaseProvider:` but subclasses kept forgetting to implement required methods.

Discovered Python's ABC module forces you to implement abstract methods:

```python
from abc import ABC, abstractmethod

class BaseProvider(ABC):
    @abstractmethod
    def query(self, prompt: str):
        """Every provider MUST implement this"""
        pass

    @abstractmethod
    def is_available(self) -> bool:
        """And this too"""
        pass

class AnthropicProvider(BaseProvider):
    def query(self, prompt: str):
        # Implementation here
        return response

    # Forgot is_available()? Can't instantiate!
```

Try to create `AnthropicProvider()` without implementing both methods? Python raises `TypeError: Can't instantiate abstract class`.

This caught so many bugs during development. Now all 10+ providers in llmswap implement the same interface - guaranteed at runtime.
