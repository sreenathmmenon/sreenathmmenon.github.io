---
title: "MongoDB explain() Shows Why Your Query is Slow"
date: 2025-11-29
tags: [mongodb, performance, database]
---

Query was taking 12 seconds on our VM search. I knew indexes existed, but why wasn't MongoDB using them?

`explain("executionStats")` shows exactly what MongoDB is doing:

```javascript
// Your slow query
db.vms.find({ name: "prod-db" }).explain("executionStats")

// Output reveals the problem:
{
  "executionStats": {
    "executionTimeMillis": 12450,
    "totalDocsExamined": 54823,      // Scanned ALL documents!
    "totalKeysExamined": 0,           // Used ZERO indexes!
    "executionStages": {
      "stage": "COLLSCAN",            // COLLSCAN = bad!
      ...
    }
  }
}
```

**What to look for:**

**Bad - Collection scan:**
```javascript
"stage": "COLLSCAN"  // Scanning entire collection
"totalDocsExamined": 54823  // Read every document
"totalKeysExamined": 0  // No index used
```

**Good - Index scan:**
```javascript
"stage": "IXSCAN"  // Using index!
"totalDocsExamined": 1  // Only read matching docs
"totalKeysExamined": 1  // Used index efficiently
```

After adding the missing index:

```javascript
db.vms.createIndex({ name: 1 })

db.vms.find({ name: "prod-db" }).explain("executionStats")
// Now: IXSCAN, 85ms instead of 12s!
```

Run `explain()` on any slow query - it tells you exactly what's wrong.
