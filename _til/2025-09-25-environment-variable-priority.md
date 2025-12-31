---
title: "Environment Variable Priority Pattern I Use Everywhere"
date: 2025-09-25
tags: [python, configuration, best-practices]
---

Needed a clean way to configure API keys in llmswap. Users should be able to pass them via CLI argument, environment variable, or config file. But what takes priority?

Settled on this pattern:

```python
class AnthropicProvider:
    def __init__(self, api_key=None, model=None):
        # Priority: argument > env var > config file
        self.api_key = (
            api_key or                           # 1. Explicit argument
            os.getenv("ANTHROPIC_API_KEY") or    # 2. Environment variable
            self._load_from_config("api_key")    # 3. Config file
        )

        if not self.api_key:
            raise ConfigurationError("ANTHROPIC_API_KEY not found")
```

This gives users flexibility:

```bash
# Production - env vars (secure, no code changes)
export ANTHROPIC_API_KEY="sk-..."
python app.py

# Development - config file (convenient)
# config.yaml has the key
python app.py

# Override - CLI argument (testing different keys)
python app.py --api-key "sk-test-..."
```

Clear priority order: most explicit wins. Used this same pattern for models, timeouts, all configuration.
