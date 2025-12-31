---
title: "hashlib.sha256() for Consistent Cache Keys"
date: 2025-11-01
tags: [python, caching, hashing]
---

Building llmswap's cache, I needed a way to generate consistent keys from prompts. Can't use the prompt itself as the dictionary key - some prompts are 1000+ characters long.

Hash functions create fixed-size keys from any input:

```python
import hashlib

def create_cache_key(prompt: str, context: dict = None) -> str:
    if context:
        # Combine prompt + context
        key_data = f"{prompt}:{json.dumps(context, sort_keys=True)}"
    else:
        key_data = prompt

    # Generate 64-character hex string
    return hashlib.sha256(key_data.encode()).hexdigest()

# Example
key1 = create_cache_key("Explain Python")
# '7f3d...' (64 chars, always the same for this prompt)

key2 = create_cache_key("Explain Python", {"user": "123"})
# 'a8c2...' (different - includes user context)
```

Why SHA256?
- **Deterministic**: Same input always produces same hash
- **Fixed size**: Any prompt → 64 character key
- **Fast**: Millions of hashes per second
- **Collision-resistant**: Two different prompts won't hash to same key

This lets me use prompts of any length as cache keys. A 5000-character prompt becomes a neat 64-character hash.
