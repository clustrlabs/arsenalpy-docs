# Installation Guide

## Requirements

Arsenal requires Python 3.12 or higher and works with various AI providers including OpenAI, X-AI, Anthropic, and OpenRouter.

## Installing Arsenal

You can install Arsenal using pip:

```bash
pip install arsenalpy
```

This installs the core framework with basic functionality. For additional features, use optional dependencies:

```bash
# For all optional features (includes web browsing capabilities via crawl4ai)
pip install "arsenalpy[all]"

# For development (includes formatting, linting, testing tools)
pip install "arsenalpy[dev]"
```

### Optional Dependencies Explained

- **all**: Includes web browsing capabilities via `crawl4ai`
- **dev**: Development tools (black, isort, mypy, ruff) and testing tools (pytest)

## Environment Setup

Arsenal requires certain environment variables to be set for connecting to AI providers:

```bash
# OpenAI
export OPENAI_API_KEY=your_openai_key

# X-AI
export XAI_API_KEY=your_xai_key

# OpenRouter
export OPENROUTER_API_KEY=your_openrouter_key
```

## Search and Browse Skills

To use the search and browse skills, you'll need API keys for the corresponding services:

```bash
# Search engines
export BRAVE_API_KEY=your_brave_key
export SERPER_API_KEY=your_serper_key
export TAVILY_API_KEY=your_tavily_key

# For search result reranking
export COHERE_API_KEY=your_cohere_key
```

## Web Browsing Setup

For web browsing capabilities, install Arsenal with the browse extra:

```bash
pip install "arsenalpy[all]"
```

This installs the Crawl4AI package which Arsenal uses for advanced browsing.

## Development Installation

For development, install Arsenal with development dependencies:

```bash
git clone https://github.com/clustrlabs/arsenalpy.git
cd arsenalpy
pip install -e ".[dev]"
```

This installs all development tools including code formatting, linting, and testing tools.

## Verifying Installation

You can verify your installation by running a simple example:

```python
import asyncio
from arsenalpy.agents import Agent

async def main():
    agent = Agent(
        name="test_agent",
        system_prompt="You are a helpful assistant.",
        model="o3-mini",
        api_key="your_openai_api_key",
        provider="openai"
    )

    response = await agent.do("Hello, how are you?")
    print(response)

if __name__ == "__main__":
    asyncio.run(main())
```

For more detailed examples, check the [Quick Start Guide](quickstart.md).
