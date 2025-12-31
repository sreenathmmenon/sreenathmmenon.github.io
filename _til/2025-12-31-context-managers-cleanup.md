---
title: "Context Managers Clean Up API Clients Automatically"
date: 2025-12-31
tags: [python, best-practices, api]
---

Was debugging why some API connections weren't closing properly in llmswap. Realized I was forgetting to cleanup client resources in error cases.

Python's context managers (`with` statement) handle this automatically:

```python
class LLMClient:
    def __enter__(self):
        # Setup - called when entering 'with' block
        self._setup_connections()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        # Cleanup - ALWAYS called, even on errors
        self._close_connections()
        self._flush_cache()
        return False  # Don't suppress exceptions

# Usage
with LLMClient() as client:
    response = client.query("Hello")
    # Even if this raises an error...
# ...cleanup still happens here!
```

Before, I had try/finally blocks everywhere:

```python
# Old way - easy to forget
client = LLMClient()
try:
    response = client.query("Hello")
finally:
    client.close()  # Easy to forget this!
```

Now cleanup is automatic. The `__exit__` method runs no matter what - normal completion, exception, even `KeyboardInterrupt`.
