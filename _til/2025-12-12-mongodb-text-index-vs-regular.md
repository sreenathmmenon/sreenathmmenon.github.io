---
title: "MongoDB Text Index vs Regular Index for Search"
date: 2025-12-12
tags: [mongodb, search, indexing]
---

Needed search functionality but wasn't sure when to use text indexes vs regular indexes. They're for different use cases.

**Regular index** - For exact or prefix matches:

```javascript
db.articles.createIndex({ title: 1 })

// Fast for exact match
db.articles.find({ title: "Python Tutorial" })

// Fast for prefix (starts with)
db.articles.find({ title: /^Python/ })

// NOT good for word search
db.articles.find({ title: /tutorial/ })  // Still slow
```

**Text index** - For full-text search:

```javascript
db.articles.createIndex({ title: "text", content: "text" })

// Search for words anywhere in text
db.articles.find({ $text: { $search: "python mongodb" } })

// Searches across multiple fields
// Handles word stems (search, searching, searched)
// Ignores common words (the, and, or)
```

**When to use which:**

| Use Case | Index Type | Example |
|----------|------------|---------|
| Search by exact UUID | Regular | `{ uuid: "abc-123" }` |
| Search by VM name prefix | Regular | `{ name: /^prod/ }` |
| Search documentation | Text | `$text: "how to deploy"` |
| Autocomplete | Regular | `{ name: /^user_input/ }` |
| Article/content search | Text | `$text: "kubernetes docker"` |

For our infrastructure UI, we use regular indexes (exact UUID, IP lookups). For the docs section, we'd use text indexes.
