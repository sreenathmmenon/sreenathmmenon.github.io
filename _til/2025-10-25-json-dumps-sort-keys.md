---
title: "json.dumps(sort_keys=True) for Reliable Hashing"
date: 2025-10-25
tags: [python, json, hashing]
---

Hit a weird caching bug in llmswap. Same query with same context was creating different cache keys randomly. Caching wasn't working.

Problem: JSON dictionaries don't have guaranteed order in Python < 3.7, and even in 3.7+ the insertion order might vary.

```python
# These are the same data, but hash differently!
context1 = {"user": "123", "session": "abc"}
context2 = {"session": "abc", "user": "123"}

key1 = json.dumps(context1)  # '{"user":"123","session":"abc"}'
key2 = json.dumps(context2)  # '{"session":"abc","user":"123"}'

# Different strings = different hashes = cache miss!
```

Solution: `sort_keys=True` ensures consistent JSON output:

```python
import json

# Always produce same string regardless of dict order
key_data = json.dumps(context, sort_keys=True)

# Now these produce identical strings
json.dumps({"user": "123", "session": "abc"}, sort_keys=True)
json.dumps({"session": "abc", "user": "123"}, sort_keys=True)
# Both: '{"session":"abc","user":"123"}'  (alphabetical)
```

Used this in llmswap's cache key generation:

```python
key_data = f"{prompt}:{json.dumps(context, sort_keys=True)}"
cache_key = hashlib.sha256(key_data.encode()).hexdigest()
```

Cache hit rate went from 45% to 82%. Same context now reliably produces same key.
