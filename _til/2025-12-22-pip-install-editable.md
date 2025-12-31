---
title: "pip install -e . for Local Package Development"
date: 2025-12-22
tags: [python, development, pip]
---

While developing llmswap, I was constantly running `pip uninstall llmswap` then `pip install .` after every code change to test. Incredibly annoying.

Learned about editable installs: `pip install -e .`

The `-e` flag (editable mode) creates a link to your source code instead of copying it:

```bash
cd /path/to/llmswap

# Regular install - copies code to site-packages
pip install .
# Change code -> need to reinstall

# Editable install - links to your source
pip install -e .
# Change code -> changes reflected immediately!
```

Now my workflow is:

```bash
# One-time setup
cd ~/Code/llmswap
pip install -e .

# Make changes to llmswap/client.py
vim llmswap/client.py

# Test immediately - no reinstall needed!
python test_script.py

# Changes are live!
```

Perfect for development. When I edit `llmswap/providers.py`, the changes work right away in my test scripts. No more install/uninstall cycle.

For production, use regular `pip install llmswap` without `-e`.
