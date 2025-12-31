---
title: "What I Learned Building a CLI That People Actually Use"
date: 2025-09-15
excerpt: "Lessons from building llmswap's command-line interface - from argparse chaos to a tool with 6000+ downloads."
tags: [python, cli, argparse, ux, developer-tools]
keywords: "CLI design, argparse, Python CLI, developer tools, user experience, command-line interface"
---

My first version of llmswap CLI was terrible.

```bash
llmswap --ask "What is Docker?" --provider anthropic --model claude-3-opus-20240229 --temperature 0.7 --max-tokens 4000

# Nobody wants to type this.
```

After building, testing, and iterating for months, the CLI now has 6000+ downloads on PyPI. Here's what I learned about making command-line tools people actually want to use.

## Lesson 1: Start With the User Experience, Not the Code

**Wrong approach:** Build based on your code structure

```bash
# My first attempt - mirrored my Python classes
llmswap --client-type llm --operation query --provider openai --input "hello"
```

**Right approach:** Think about what users type

```bash
# What users actually want
llmswap ask "hello"
llmswap chat
llmswap review app.py
```

I rewrote the entire argument parsing three times before getting this right. The final version has clean subcommands:

```python
import argparse

parser = argparse.ArgumentParser(description='llmswap - Universal AI CLI')
subparsers = parser.add_subparsers(dest='command')

# Clean subcommands
ask_parser = subparsers.add_parser('ask', help='Ask a question')
chat_parser = subparsers.add_parser('chat', help='Start conversation')
review_parser = subparsers.add_parser('review', help='Review code')
```

Think git-style: `git commit`, `git push`, `git log`. Users know this pattern.

## Lesson 2: Smart Defaults Beat Configuration

**Wrong:** Make users specify everything

```bash
llmswap ask "question" --provider anthropic --model claude-3-opus-20240229 --max-tokens 4000 --temperature 0.7
```

**Right:** Sane defaults, easy overrides

```bash
# Just works
llmswap ask "question"

# Override only what you need
llmswap ask "question" --provider openai
```

Implementation:

```python
class LLMClient:
    def __init__(
        self,
        provider=None,  # Auto-detects from env vars
        model=None,  # Uses provider's default
        temperature=None,  # Sensible default per provider
        max_tokens=None  # Provider default
    ):
        self.provider = provider or self._detect_provider()
        self.model = model or self._get_default_model(self.provider)
        # ...
```

90% of users never change defaults. Make the defaults excellent.

## Lesson 3: Progressive Disclosure

Show complexity only when needed.

**Basic usage - hide all complexity:**

```bash
llmswap ask "What is Docker?"
# Clean, simple, works
```

**Power users - full control available:**

```bash
llmswap ask "What is Docker?" \
  --provider anthropic \
  --model claude-3-opus-20240229 \
  --temperature 0.9 \
  --max-tokens 2000 \
  --mentor guru \
  --age 25 \
  --format markdown
```

Everyone gets what they need at their level.

## Lesson 4: Error Messages Should Help, Not Scold

**Bad error message:**

```
Error: Invalid provider
```

What do I do with this?

**Good error message:**

```
Error: Provider 'opeanai' not found

Did you mean 'openai'?

Available providers:
  - anthropic (Claude)
  - openai (GPT-4)
  - gemini (Google)
  - groq (Fast inference)

Set provider with: --provider openai
Or set env var: export OPENAI_API_KEY="sk-..."
```

Implementation:

```python
def validate_provider(provider):
    valid_providers = ['anthropic', 'openai', 'gemini', 'groq', ...]

    if provider not in valid_providers:
        # Fuzzy match for typos
        from difflib import get_close_matches
        suggestions = get_close_matches(provider, valid_providers, n=1)

        error = f"Error: Provider '{provider}' not found\n"
        if suggestions:
            error += f"\nDid you mean '{suggestions[0]}'?\n"

        error += "\nAvailable providers:\n"
        for p in valid_providers:
            error += f"  - {p}\n"

        error += f"\nSet provider with: --provider {suggestions[0] if suggestions else 'anthropic'}"

        raise ConfigurationError(error)
```

Help users fix the problem immediately.

## Lesson 5: Interactive Mode is Powerful

Some tasks don't fit one-off commands. Add interactive modes:

```bash
llmswap chat

Starting chat with Claude (anthropic)...
Type /help for commands, /quit to exit

You: What is Docker?
Claude: Docker is a containerization platform...

You: How do I install it?
Claude: Here are the installation steps...

You: /switch openai
Switched to OpenAI (gpt-4)

You: Same question
GPT-4: Docker installation varies by OS...

You: /quit
Goodbye!
```

The interactive loop:

```python
def interactive_chat():
    print("Starting chat... Type /help for commands, /quit to exit\n")

    client = LLMClient()
    history = []

    while True:
        try:
            user_input = input("You: ").strip()

            if not user_input:
                continue

            # Handle commands
            if user_input.startswith('/'):
                handle_command(user_input, client)
                continue

            # Regular chat
            history.append({"role": "user", "content": user_input})
            response = client.chat(history)

            print(f"{client.provider.title()}: {response.content}\n")
            history.append({"role": "assistant", "content": response.content})

        except KeyboardInterrupt:
            print("\nGoodbye!")
            break
        except Exception as e:
            print(f"Error: {e}\n")
```

Interactive mode increased usage by 40%. People love it for exploratory tasks.

## Lesson 6: Color and Formatting Matter

**Without formatting:**

```
Response from claude: Docker is a platform for developing shipping and running applications Docker provides the ability to package and run an application in a loosely isolated environment called a container
```

Hard to read. Walls of text.

**With formatting:**

```python
from rich.console import Console
from rich.markdown import Markdown

console = Console()

# Pretty markdown rendering
console.print(Markdown(response.content))

# Colored status messages
console.print("[green]✓[/green] Response cached (saved $0.03)")
console.print("[yellow]![/yellow] Using fallback provider")
console.print("[red]✗[/red] API key not found")
```

Users noticed and appreciated the polish.

## Lesson 7: Help Should Be Discoverable

**Bad help:**

```bash
llmswap --help  # 200 lines of text, overwhelming
```

**Good help:**

```bash
# Top-level help - concise
llmswap --help
Usage: llmswap <command> [options]

Commands:
  ask       Ask a quick question
  chat      Start interactive chat
  generate  Generate code/commands
  review    Review code with AI
  debug     Debug errors

Run 'llmswap <command> --help' for command-specific help.

# Command-specific help
llmswap ask --help
Usage: llmswap ask <question> [options]

Ask a quick question and get an immediate answer.

Options:
  --provider    AI provider (anthropic, openai, gemini, groq)
  --mentor      Teaching style (guru, coach, friend)
  --age         Explanation level (10, 15, 25, expert)

Examples:
  llmswap ask "What is Docker?"
  llmswap ask "Explain decorators" --mentor guru
  llmswap ask "How does TCP work?" --age 15
```

Organized help by what users need, when they need it.

## Lesson 8: Examples Are the Best Documentation

Every command has real examples:

```bash
llmswap review --help

Examples:
  # Basic code review
  llmswap review app.py

  # Focus on specific concerns
  llmswap review app.py --focus security
  llmswap review app.py --focus performance
  llmswap review app.py --focus bugs

  # Review from stdin
  cat app.py | llmswap review

  # Different language
  llmswap review script.sh --language bash
```

Examples got copied more than documentation was read.

## Lesson 9: Fail Fast with Clear Guidance

**Bad startup:**

```bash
llmswap ask "hello"
# Hangs for 30 seconds
Error: API request failed
```

**Good startup:**

```bash
llmswap ask "hello"

Checking configuration...
✗ No API keys found

To use llmswap, set at least one provider's API key:

  export ANTHROPIC_API_KEY="sk-..."
  export OPENAI_API_KEY="sk-..."
  export GEMINI_API_KEY="..."

Or create ~/.llmswap/config.yaml:

  provider:
    default: anthropic
    api_keys:
      anthropic: "sk-..."

Get API keys:
  - Anthropic: https://console.anthropic.com
  - OpenAI: https://platform.openai.com
  - Gemini: https://makersuite.google.com

Run 'llmswap providers' to see all supported providers.
```

Check requirements at startup. Guide users to success.

## Lesson 10: Version and Cache Management

Users need visibility and control:

```bash
# Show version clearly
llmswap --version
llmswap version 5.0.2

# Cache management
llmswap cache stats
Hit rate: 82.3%
Entries: 1,247
Memory used: 47.2 MB / 100 MB

llmswap cache clear
Cache cleared. Freed 47.2 MB.

# Provider status
llmswap providers
✓ anthropic (Claude) - configured
✓ openai (GPT-4) - configured
✗ gemini (Gemini) - missing API key
✓ groq (Fast LLaMA) - configured
```

Give users insight into what's happening.

## The Numbers

After implementing these lessons:

**Before:**
- Daily active users: ~50
- Average session length: 2 minutes
- Commands per session: 1.3
- Error rate: 23%
- GitHub issues: "Too confusing", "How do I...?"

**After:**
- Daily active users: ~400
- Average session length: 8 minutes
- Commands per session: 6.7
- Error rate: 4%
- GitHub issues: Feature requests, not confusion

## Key Principles

**1. Think like a user, not a programmer**
- What would I type if I didn't know how it worked?
- What's the minimal information needed?

**2. Optimize for the common case**
- 90% of users do simple things - make those trivial
- Power features available but not required

**3. Guide, don't block**
- Error messages should teach
- Examples should inspire
- Help should clarify

**4. Polish matters**
- Color improves comprehension
- Formatting aids readability
- Responsiveness feels professional

**5. Test with real users**
- Watch someone use it (don't help!)
- Fix what confuses them
- Simplify what takes too long

## The CLI Checklist

For any new CLI tool, I now ensure:

- ✓ Subcommands for different operations
- ✓ Smart defaults for 90% use case
- ✓ Help text with real examples
- ✓ Descriptive error messages
- ✓ Fuzzy matching for typos
- ✓ Interactive mode for complex workflows
- ✓ Version and status commands
- ✓ Colored output with rich/colorama
- ✓ Fail fast with clear guidance
- ✓ Progressive disclosure of complexity

The best CLIs feel invisible. Users get their work done and forget the tool was even there. That's success.

---

**Code:** llmswap CLI is open source - [github.com/sreenathmmenon/llmswap](https://github.com/sreenathmmenon/llmswap)

**Try it:**
```bash
pip install llmswap
llmswap ask "What is the best way to design a CLI?"
```

The CLI will tell you itself. And it won't be overwhelming.
