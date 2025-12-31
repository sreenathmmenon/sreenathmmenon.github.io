---
title: "MongoDB $or Queries Need Indexes on EACH Field"
date: 2025-10-08
tags: [mongodb, indexing, performance]
---

Our search was letting users find VMs by UUID, name, or IP address. Used `$or` for this. Had an index on `uuid`, but query was still slow.

Assumed one index would be enough for the whole `$or`. Wrong.

```javascript
// This query was slow even with index on uuid
db.vms.find({
  $or: [
    { uuid: searchTerm },
    { name: searchTerm },
    { ip: searchTerm }
  ]
})
```

MongoDB needs an index on **each field** in the `$or`:

```javascript
// Create indexes on ALL fields used in $or
db.vms.createIndex({ uuid: 1 })
db.vms.createIndex({ name: 1 })
db.vms.createIndex({ ip: 1 })

// Now the query is fast!
db.vms.find({
  $or: [
    { uuid: searchTerm },
    { name: searchTerm },
    { ip: searchTerm }
  ]
})
```

MongoDB evaluates each `$or` branch separately. If even one branch lacks an index, you get a collection scan.

Check with `.explain()`:
```javascript
// Look for COLLSCAN in any branch = missing index
db.vms.find({ $or: [...] }).explain("executionStats")
```

After adding all three indexes, search went from 8 seconds to <100ms.
