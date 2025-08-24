# Sreenath M Menon - Personal Website

A simple Jekyll-based portfolio website showcasing open source projects, technical learnings, and professional journey.

## Local Development

```bash
# Install dependencies
bundle install

# Run locally
bundle exec jekyll serve

# View at http://localhost:4000
```

## Structure

- `/` - Homepage with brief intro and highlights
- `/open-source/` - Open source projects and contributions
- `/til/` - Today I Learned posts (quick technical insights)
- `/bookshelf/` - Books and reading recommendations
- `/bookmarks/` - Useful tools and resources
- `/about/` - Personal background and journey

## Adding TIL Posts

Create new files in `_til/` directory with format: `YYYY-MM-DD-title.md`

```markdown
---
title: "Quick Learning Title"
date: 2024-12-20
tags: [python, api, tools]
---

Brief description of what you learned...
```

## Deployment

This site is designed to work with GitHub Pages. Just push to your GitHub repository and enable Pages in settings.