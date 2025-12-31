---
title: "time.time() vs time.monotonic() for Duration Tracking"
date: 2025-12-01
tags: [python, performance, timing]
---

Was tracking API response times in llmswap and noticed some latencies were showing as negative numbers. Turned out my server's NTP had adjusted the clock mid-request.

`time.time()` can go backwards when system clock adjusts. Bad for measuring duration.

```python
# Don't do this - can give negative or wrong durations
import time

start = time.time()
# NTP adjusts clock backwards here...
result = api_call()
duration = time.time() - start  # Negative? Wrong? Who knows!
```

`time.monotonic()` never goes backwards:

```python
import time

class AnthropicProvider:
    def query(self, prompt: str):
        start_time = time.monotonic()  # Monotonically increasing

        response = self.client.messages.create(...)

        latency = time.monotonic() - start_time  # Always accurate

        return LLMResponse(content=..., latency=latency)
```

`time.monotonic()` is for measuring elapsed time. `time.time()` is for wall clock/timestamps.

Changed all duration tracking in llmswap to use `monotonic()`. No more weird negative latencies in the logs.
