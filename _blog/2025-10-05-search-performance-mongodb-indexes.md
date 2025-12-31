---
title: "Why Our Users Were Complaining About Slow Search (And How 4 Lines Fixed It)"
date: 2025-10-05
excerpt: "How MongoDB indexes transformed our infrastructure search from 15 seconds to under 100ms, fixing customer complaints overnight."
tags: [mongodb, performance, indexing, database, infrastructure, optimization]
keywords: "MongoDB performance, database indexing, query optimization, collection scan, infrastructure search"
---

The complaints started coming in around 2 PM on a Tuesday. "Search is painfully slow." "Why does finding a VM take so long?" "Is the system down?"

Our infrastructure management UI lets customers search across their VMs, volumes, hosts - everything. When you're managing 1000+ VMs, search is critical. And ours was taking 15+ seconds.

## The Scale of the Problem

Our customers weren't just annoyed. They were paying for a premium service and getting slow responses for basic operations:

- Search for VM by UUID: "550e8400-e29b-41d4-a716-446655440000" - 15 seconds
- Find VM by name: "prod-database-01" - 12 seconds
- Look up VM by IP: "10.24.3.156" - 14 seconds
- Find volume by UUID: "7b3f6d2a-9c8e-4f5a-b1d2-3e4f5a6b7c8d" - 16 seconds

Every. Single. Search. Was scanning the entire collection.

With 50,000+ VMs in the database, every query was a collection scan. MongoDB was reading every single document to find matches.

## Finding the Problem

I pulled up the logs and saw the queries we were running:

```javascript
// The search query
db.vms.find({
  $or: [
    { uuid: searchTerm },
    { name: searchTerm },
    { ip: searchTerm },
    { hostname: searchTerm }
  ]
})
```

Looked reasonable. But let me check what MongoDB was actually doing:

```javascript
db.vms.find({
  $or: [
    { uuid: "550e8400-e29b-41d4-a716-446655440000" },
    { name: "550e8400-e29b-41d4-a716-446655440000" },
    { ip: "550e8400-e29b-41d4-a716-446655440000" },
    { hostname: "550e8400-e29b-41d4-a716-446655440000" }
  ]
}).explain("executionStats")
```

The output made me wince:

```javascript
{
  "executionStats": {
    "executionTimeMillis": 14823,  // 14.8 seconds!
    "totalDocsExamined": 54823,    // Scanned EVERY document
    "totalKeysExamined": 0,         // Used ZERO indexes
    "executionStages": {
      "stage": "COLLSCAN",          // Collection scan = disaster
      ...
    }
  }
}
```

There it was. `COLLSCAN`. Collection scan. MongoDB was reading all 54,823 documents for every search query.

## Why No Indexes?

I checked the indexes:

```javascript
db.vms.getIndexes()

[
  { "_id": 1 }  // Just the default _id index
]
```

We had NO indexes on the fields we were searching. Every `$or` branch was doing a full collection scan.

## The Fix

The solution was embarrassingly simple. Create indexes on the fields we search:

```javascript
// Individual indexes for each searchable field
db.vms.createIndex({ "uuid": 1 })
db.vms.createIndex({ "name": 1 })
db.vms.createIndex({ "ip": 1 })
db.vms.createIndex({ "hostname": 1 })

// Same for other collections
db.volumes.createIndex({ "uuid": 1 })
db.volumes.createIndex({ "name": 1 })

db.hosts.createIndex({ "hostname": 1 })
db.hypervisors.createIndex({ "hostname": 1 })
```

Four collections, a few indexes each. Took about 2 minutes to create them all.

## The Results

Ran the same query again:

```javascript
db.vms.find({
  $or: [
    { uuid: "550e8400-e29b-41d4-a716-446655440000" },
    { name: "550e8400-e29b-41d4-a716-446655440000" },
    { ip: "550e8400-e29b-41d4-a716-446655440000" },
    { hostname: "550e8400-e29b-41d4-a716-446655440000" }
  ]
}).explain("executionStats")

{
  "executionStats": {
    "executionTimeMillis": 87,     // 0.087 seconds!
    "totalDocsExamined": 1,        // Only read matching doc
    "totalKeysExamined": 4,        // Used indexes!
    "executionStages": {
      "stage": "IXSCAN",           // Index scan = good!
      ...
    }
  }
}
```

**From 14.8 seconds to 0.087 seconds.**

**170x faster.**

## The Impact

Customer complaints stopped immediately. Search went from "painfully slow" to "instant."

**Before:**
- Average search time: 13.5 seconds
- Customer complaints: 15-20 per day
- Support tickets: "System feels broken"

**After:**
- Average search time: 85ms
- Customer complaints: Zero
- Support tickets: None about search

## What I Learned

### 1. Always Check explain()

Never assume a query is using indexes. Always check:

```javascript
db.collection.find({ field: value }).explain("executionStats")
```

Look for:
- `COLLSCAN` = BAD (collection scan)
- `IXSCAN` = GOOD (index scan)
- `totalDocsExamined` vs `totalKeysExamined`

### 2. $or Queries Need Indexes on ALL Fields

This caught me by surprise. I thought MongoDB would be smart about it. Nope.

```javascript
// This query needs THREE indexes
db.vms.find({
  $or: [
    { uuid: value },    // Needs index on uuid
    { name: value },    // Needs index on name
    { ip: value }       // Needs index on ip
  ]
})

// Missing even ONE index? Collection scan on that branch.
```

Each `$or` branch is evaluated separately. Missing an index on any field = collection scan for that branch.

### 3. Index Creation is Fast

Creating indexes on 50K+ documents took seconds, not hours. MongoDB creates indexes in the background by default (since version 4.2).

```javascript
db.vms.createIndex({ uuid: 1 })
// Done in < 5 seconds on 54,823 documents
```

No reason not to add them.

### 4. Indexes Have Minimal Overhead

Was worried indexes would slow down writes. They don't noticeably.

**Write performance:**
- Before indexes: ~450 inserts/sec
- After indexes: ~425 inserts/sec

5% write slowdown for 170x read speedup? Easy trade-off.

### 5. Monitor Production Queries

Should have caught this earlier. Now I monitor slow queries:

```javascript
// In mongodb.conf
profiling:
  mode: slowOp
  slowOpThresholdMs: 100

// Check slow queries
db.system.profile.find({ millis: { $gt: 100 } }).sort({ ts: -1 })
```

Any query over 100ms gets logged. Helps catch performance issues before customers complain.

## Index Strategy for Infrastructure Search

Here's the pattern I now use for all infrastructure search:

```javascript
// UUIDs - always indexed (most common search)
db.vms.createIndex({ uuid: 1 })
db.volumes.createIndex({ uuid: 1 })
db.snapshots.createIndex({ uuid: 1 })

// Names - indexed (common for human-friendly search)
db.vms.createIndex({ name: 1 })
db.volumes.createIndex({ name: 1 })

// IPs - indexed (network debugging)
db.vms.createIndex({ ip: 1 })

// Hostnames - indexed (infrastructure topology)
db.hosts.createIndex({ hostname: 1 })
db.hypervisors.createIndex({ hostname: 1 })

// Status fields - indexed (filtering active/inactive)
db.vms.createIndex({ status: 1 })
db.volumes.createIndex({ status: 1 })
```

## Compound Indexes for Complex Queries

Found some queries filter by multiple fields together:

```javascript
// Common query: active VMs on a specific host
db.vms.find({ status: "active", host_id: "host-123" })

// Compound index handles this efficiently
db.vms.createIndex({ status: 1, host_id: 1 })
```

Compound indexes work left-to-right. This index helps:
- `{ status: "active" }` ✓
- `{ status: "active", host_id: "host-123" }` ✓

But NOT:
- `{ host_id: "host-123" }` ✗ (needs separate index)

## The Cost of Not Indexing

Let me show you the math:

**Without indexes:**
- 1000 searches/day
- Average 13.5 seconds each
- User frustration: HIGH
- Support burden: 15-20 tickets/day
- Engineer time debugging: 4 hours/day

**With indexes:**
- Same 1000 searches/day
- Average 0.085 seconds each
- User frustration: NONE
- Support burden: 0 tickets
- Engineer time: 10 minutes to add indexes

Four lines of code. Two minutes of work. Transformed the user experience.

## Lessons for Other Databases

This isn't MongoDB-specific. The principles apply everywhere:

**PostgreSQL:**
```sql
CREATE INDEX idx_vms_uuid ON vms(uuid);
CREATE INDEX idx_vms_name ON vms(name);
```

**MySQL:**
```sql
CREATE INDEX idx_vms_uuid ON vms(uuid);
CREATE INDEX idx_vms_name ON vms(name);
```

**Check query plans:**
- PostgreSQL: `EXPLAIN ANALYZE`
- MySQL: `EXPLAIN`
- MongoDB: `.explain("executionStats")`

Look for seq scans (PostgreSQL), table scans (MySQL), collection scans (MongoDB). All bad. All fixed with indexes.

## Current State

Months later, search is still fast:

```javascript
db.currentOp(true).inprog.forEach(op => {
  if (op.secs_running > 1) {
    print(`Slow query: ${op.query} - ${op.secs_running}s`);
  }
});

// No slow queries!
```

Average query time is 82ms. Customers are happy. No more complaints.

The whole incident taught me: **profile before optimizing, but always index your search fields**.

---

**Index checklist for any search feature:**

1. ✓ Identify all searchable fields
2. ✓ Create indexes on each field
3. ✓ Test with `explain()` or equivalent
4. ✓ Monitor slow queries in production
5. ✓ Add compound indexes for common multi-field queries

Four lines of code fixed everything:

```javascript
db.vms.createIndex({ uuid: 1 })
db.vms.createIndex({ name: 1 })
db.vms.createIndex({ ip: 1 })
db.vms.createIndex({ hostname: 1 })
```

Sometimes the best optimizations are the simplest ones.
