---
title: "threading.Lock() Saved Me From Race Conditions"
date: 2025-10-18
tags: [python, threading, concurrency]
---

Building the caching system for llmswap, I hit a nasty bug. When multiple requests came in simultaneously, cache stats were getting corrupted. Turns out two threads were modifying `self._hits` at the exact same time.

```python
# Before - race condition disaster
def get(self, key: str):
    if key not in self._responses:
        self._misses += 1  # Thread 1 reads: 5
        return None        # Thread 2 reads: 5
                          # Both write: 6 (should be 7!)
```

Python's `threading.Lock()` fixed it:

```python
import threading

class InMemoryCache:
    def __init__(self):
        self._lock = threading.Lock()
        self._responses = {}
        self._hits = 0

    def get(self, key: str):
        with self._lock:  # Only ONE thread at a time
            if key not in self._responses:
                self._misses += 1
                return None

            self._hits += 1
            return self._responses[key]
```

The `with self._lock:` ensures only one thread can execute that code block at once. Others wait their turn.

Cache stats are now accurate even with 100 concurrent requests. Simple fix, huge impact.
