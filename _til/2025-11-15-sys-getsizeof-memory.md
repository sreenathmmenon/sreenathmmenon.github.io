---
title: "sys.getsizeof() to Track Memory Usage in Python"
date: 2025-11-15
tags: [python, memory, performance]
---

Needed to enforce a memory limit for llmswap's cache. Set it to 100MB, but how do I know how much memory each cached response actually uses?

`sys.getsizeof()` returns the size of an object in bytes:

```python
import sys

# Check size of different objects
text = "Hello World"
sys.getsizeof(text)  # 60 bytes

large_text = "x" * 10000
sys.getsizeof(large_text)  # 10,049 bytes

data = {"prompt": "Explain Python", "response": "Python is..."}
sys.getsizeof(data)  # ~240 bytes (just the dict structure)
```

Used it in the cache to track memory:

```python
class InMemoryCache:
    def set(self, key: str, value: dict) -> bool:
        # Calculate size of entry
        entry_size = sys.getsizeof(key) + sys.getsizeof(value)

        # Check if we'd exceed memory limit
        if self._current_size + entry_size > self._memory_limit:
            self._evict_least_accessed()  # Make room

        # Store and track size
        self._responses[key] = value
        self._current_size += entry_size
```

Now the cache respects the 100MB limit. When it's full, it evicts old entries to make room. No more running out of memory!
