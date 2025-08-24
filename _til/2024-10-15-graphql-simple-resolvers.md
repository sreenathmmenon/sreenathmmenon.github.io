---
title: "GraphQL Resolvers Don't Have to be Complex"
date: 2024-10-15
tags: [graphql, javascript, api]
---

Used to think GraphQL resolvers needed to be these complex, optimized functions. Turns out simple is often better.

```javascript
// This works fine
const resolvers = {
  Query: {
    user: (parent, { id }) => users.find(u => u.id === id),
    posts: () => posts
  }
}
```

Start simple, optimize later when you actually need it.