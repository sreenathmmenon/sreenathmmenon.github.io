---
title: "argparse Subcommands Make CLIs Feel Professional"
date: 2025-10-12
tags: [python, cli, argparse]
---

My first version of llmswap CLI was a mess of flags: `llmswap --ask "question"` or `llmswap --chat` or `llmswap --review file.py`. Felt clunky.

Wanted it to work like git: `git commit`, `git push`, `git log` - clean subcommands.

argparse's `add_subparsers()` does exactly this:

```python
import argparse

parser = argparse.ArgumentParser(description='llmswap CLI')

# Create subcommands
subparsers = parser.add_subparsers(dest='command', help='Available commands')

# 'ask' subcommand
ask_parser = subparsers.add_parser('ask', help='Ask a quick question')
ask_parser.add_argument('question', help='Question to ask')
ask_parser.add_argument('--provider', help='AI provider to use')

# 'chat' subcommand
chat_parser = subparsers.add_parser('chat', help='Start interactive chat')
chat_parser.add_argument('--mentor', help='Teaching style')

# 'review' subcommand
review_parser = subparsers.add_parser('review', help='Review code')
review_parser.add_argument('file', help='File to review')

args = parser.parse_args()
```

Now users can run:

```bash
llmswap ask "What is Docker?"
llmswap chat --mentor guru
llmswap review app.py --focus security
```

Much cleaner than flags everywhere. Each subcommand gets its own help too: `llmswap ask --help`.
