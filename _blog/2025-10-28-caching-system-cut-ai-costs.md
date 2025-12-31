---
title: "Building a Smart Caching System That Cut AI API Costs by 85%"
date: 2025-10-28
excerpt: "How I built an in-memory cache for llmswap that reduced repeat API calls from 3 seconds to 0.66 seconds using thread-safe LRU eviction."
tags: [python, caching, performance, llm, api, threading, memory-management, optimization]
keywords: "Python caching, LRU cache, thread safety, API cost reduction, memory management, TTL cache, performance optimization"
---

Every repeat query to Claude or GPT-4 was costing money. Same question, same answer, different bill.

I was testing llmswap with the same prompts repeatedly during development. "Explain Python decorators" - API call. Test again - another API call. $0.03 each time.

That's when I realized: I need caching.

## The Problem

```python
# Without caching - every query hits the API
client = LLMClient()
response1 = client.query("What is Docker?")  # API call: 2.96s, $0.03
response2 = client.query("What is Docker?")  # API call: 2.91s, $0.03  <- Same question!
```

Multiply this by hundreds of test runs, documentation queries, and repeated user questions in production. The costs add up fast.

## Design Requirements

Building a cache for AI responses isn't just "store the result." I needed:

1. **Thread safety** - Multiple requests hitting cache simultaneously
2. **Memory limits** - Can't let cache grow forever
3. **TTL (Time To Live)** - Stale data should expire
4. **Fast lookups** - Caching shouldn't add latency
5. **Context-aware keys** - Same prompt, different context = different cache entry

## Implementation

### 1. Hash-Based Keys

Can't use prompts as dictionary keys - some are 1000+ characters. Used SHA256 hashing:

```python
import hashlib
import json

def create_cache_key(prompt: str, context: dict = None) -> str:
    if context:
        # Include user context in key
        key_data = f"{prompt}:{json.dumps(context, sort_keys=True)}"
    else:
        key_data = prompt

    return hashlib.sha256(key_data.encode()).hexdigest()
```

Why `sort_keys=True`? Without it, `{"user": "123", "session": "abc"}` and `{"session": "abc", "user": "123"}` hash differently even though they're the same data. That bug killed my cache hit rate until I found it.

### 2. Thread-Safe Operations

The cache gets hit from multiple API requests simultaneously. Without thread safety, I was getting corrupted statistics and occasional crashes.

```python
import threading

class InMemoryCache:
    def __init__(self):
        self._responses = {}
        self._lock = threading.Lock()
        self._hits = 0
        self._misses = 0

    def get(self, key: str):
        with self._lock:  # Only one thread at a time
            if key not in self._responses:
                self._misses += 1
                return None

            self._hits += 1
            return self._responses[key]
```

The `with self._lock:` ensures only one thread modifies cache at a time. Others wait. Simple, effective.

### 3. TTL Expiration

Cached responses shouldn't live forever. AI models improve, data changes, answers get stale.

```python
import time

class InMemoryCache:
    def __init__(self, default_ttl=3600):  # 1 hour default
        self._responses = {}
        self._expiry_map = {}
        self._default_ttl = default_ttl

    def set(self, key: str, value: dict, ttl: int = None):
        ttl = ttl if ttl is not None else self._default_ttl

        with self._lock:
            self._responses[key] = value
            self._expiry_map[key] = time.time() + ttl  # Expiry timestamp

    def get(self, key: str):
        with self._lock:
            if key not in self._responses:
                return None

            # Check if expired
            if time.time() > self._expiry_map[key]:
                self._remove_entry(key)
                return None

            return self._responses[key]
```

FAQ bot responses? Cache for 1 week. Real-time data queries? Maybe 5 minutes. Configurable per use case.

### 4. Memory Management with LRU Eviction

Setting a memory limit was crucial - don't want the cache eating all available RAM.

```python
import sys

class InMemoryCache:
    def __init__(self, max_memory_mb=100):
        self._memory_limit = max_memory_mb * 1024 * 1024  # Convert to bytes
        self._current_size = 0
        self._access_count = {}

    def set(self, key: str, value: dict):
        # Calculate entry size
        entry_size = sys.getsizeof(key) + sys.getsizeof(value)

        with self._lock:
            # Would we exceed limit?
            if self._current_size + entry_size > self._memory_limit:
                self._evict_least_accessed()  # Make room

            self._responses[key] = value
            self._current_size += entry_size
            self._access_count[key] = 0

    def _evict_least_accessed(self):
        # Sort by access count
        sorted_keys = sorted(
            self._access_count.keys(),
            key=lambda k: self._access_count[k]
        )

        # Remove bottom 20%
        remove_count = max(1, len(sorted_keys) // 5)
        for key in sorted_keys[:remove_count]:
            self._remove_entry(key)
```

When cache fills up, evict the least accessed 20%. Popular entries stay, rarely-used ones go. LRU (Least Recently Used) strategy.

## The Results

Ran tests with 100 queries - mix of unique and repeated:

```python
# Test Results
client = LLMClient(cache_enabled=True)

# First query - cache miss
response = client.query("Explain Python decorators")
# Time: 2.96s, Cost: $0.03

# Second query - cache hit!
response = client.query("Explain Python decorators")
# Time: 0.66s, Cost: $0.00

# Speedup: 4.48x
# Cost savings: 100%
```

Real-world stats after one week in production:
- Cache hit rate: 82%
- Average response time: 1.2s (was 2.8s)
- API cost reduction: 85%
- Memory usage: 47MB (under 100MB limit)

## Usage in llmswap

```python
from llmswap import LLMClient

# Enable caching
client = LLMClient(
    cache_enabled=True,
    cache_ttl=3600,  # 1 hour
    cache_max_size_mb=100
)

# First call hits API
r1 = client.query("What are Python generators?")

# Second call uses cache (instant + free!)
r2 = client.query("What are Python generators?")

# Check stats
stats = client.get_cache_stats()
print(f"Hit rate: {stats['hit_rate']}%")
print(f"Memory used: {stats['memory_used_mb']}MB")
```

## Lessons Learned

**1. Thread safety isn't optional**

Learned this the hard way when cache statistics were corrupted. `threading.Lock()` is simple and works.

**2. `json.dumps(sort_keys=True)` matters**

Without key sorting, identical context objects hashed differently. Cache hit rate was 45%. After adding `sort_keys=True`, jumped to 82%.

**3. Use `time.monotonic()` for duration tracking**

Initially used `time.time()` to measure response times. Got negative durations when NTP adjusted the server clock. `time.monotonic()` never goes backwards.

**4. Memory limits prevent production disasters**

In testing, cache grew to 850MB before I added limits. Would have killed production. Now capped at 100MB with LRU eviction.

**5. SHA256 is fast enough**

Worried hashing would add latency. It doesn't. SHA256 hashes millions of times per second. The 0.01ms overhead is invisible compared to 2-3 second API calls.

## When NOT to Cache

Caching isn't always the answer:

❌ **Real-time data queries** - "What's the current stock price?"
❌ **User-specific sensitive data** - Privacy concerns
❌ **Creative writing requests** - Want variety, not same response

✅ **Documentation queries** - "How do I use Docker?"
✅ **FAQ systems** - Same questions repeatedly
✅ **Code explanations** - "What does this function do?"
✅ **Tutorial content** - Doesn't change often

## The Complete Cache

Full implementation is in `llmswap/cache.py` (195 lines). Features:

- Thread-safe operations (`threading.Lock`)
- SHA256-based key generation
- TTL expiration tracking
- LRU eviction on memory limit
- Cache statistics (hits, misses, hit rate)
- Memory usage monitoring

## Impact

For a typical llmswap user running a documentation bot:

**Before caching:**
- 1000 queries/day
- Average 2.5s per query
- 90% repeat questions
- Cost: $30/day ($900/month)

**After caching:**
- Same 1000 queries/day
- Average 0.8s per query (70% faster)
- 82% cache hit rate
- Cost: $5.40/day ($162/month)

**Savings: $738/month, 68% faster responses**

The cache paid for itself on day one. Been running in production for two months now - zero issues, consistent 80%+ hit rate.

Building this taught me that good caching isn't complicated. Thread safety, expiration, and memory limits. Get those right, and you've got a production-ready cache.

---

**Code:** [llmswap/cache.py on GitHub](https://github.com/sreenathmmenon/llmswap)

**Try it:**
```bash
pip install llmswap
```
