---
title: "MongoDB Compound Indexes vs Multiple Single Indexes"
date: 2025-11-22
tags: [mongodb, database, indexing]
---

Was building a query filter UI where users search VMs by `status` AND `hostname` together. Wasn't sure if I needed one compound index or two separate indexes.

**Compound index** (single index on multiple fields):

```javascript
// For queries that filter on BOTH fields together
db.vms.createIndex({ status: 1, hostname: 1 })

// Efficiently handles:
db.vms.find({ status: "active", hostname: "prod-db-01" })
db.vms.find({ status: "active" })  // Also works (uses prefix)

// But NOT efficient for:
db.vms.find({ hostname: "prod-db-01" })  // Doesn't use index
```

**Separate indexes** (two individual indexes):

```javascript
// For queries that filter on fields independently
db.vms.createIndex({ status: 1 })
db.vms.createIndex({ hostname: 1 })

// Efficiently handles:
db.vms.find({ status: "active" })
db.vms.find({ hostname: "prod-db-01" })

// For combined queries, MongoDB can use index intersection (slower than compound)
db.vms.find({ status: "active", hostname: "prod-db-01" })
```

**Rule I use:**
- **AND queries** on same fields repeatedly → Compound index
- **OR queries** or independent filters → Separate indexes

For our UI, users always filter status AND hostname together, so compound index was the right choice.
