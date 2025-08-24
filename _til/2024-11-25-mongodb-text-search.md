---
title: "MongoDB Text Search is Pretty Good Actually"
date: 2024-11-25
tags: [mongodb, search, database]
---

Elasticsearch is the standard for search, but for our search feature, we needed to find options and libraries within our tech stack itself. Since we use MongoDB Community Edition, discovered their built-in text search works well.

```javascript
// Create text index
db.articles.createIndex({ title: "text", content: "text" })

// Search
db.articles.find({ $text: { $search: "python mongodb" } })
```

Works well for basic full-text search. Not as powerful as Elasticsearch, but way simpler to set up.