# Quick Start Guide

This guide will help you get started with Arsenal quickly.

## Installation

First, install Arsenal using pip:

```bash
pip install arsenalpy
```

For additional features like web browsing:

```bash
pip install "arsenalpy[all]"
```

## Environment Setup

Set up your API keys as environment variables:

```bash
# Choose your preferred provider
export OPENAI_API_KEY=your_openai_key
# or
export OPENROUTER_API_KEY=your_openrouter_key
# or
export XAI_API_KEY=your_xai_key

# For search capabilities
export BRAVE_API_KEY=your_brave_key
export TAVILY_API_KEY=your_tavily_key
export COHERE_API_KEY=your_cohere_key
```

## Basic Usage

Here's a simple example of how to create and use an agent:

```python
# basic agent

import os
from dotenv import load_dotenv
from arsenalpy.agents.agent import Agent, AgentConfig
import asyncio

load_dotenv()

print("api key: ", os.getenv("OPENROUTER_API_KEY"))

agent = Agent(
    name="basic_agent",
    provider="openrouter",
    system_prompt="You are a helpful assistant that can answer questions and help with tasks.",
    model="x-ai/grok-2-1212",
    api_key=os.getenv("OPENROUTER_API_KEY"),
    config=AgentConfig(
        # temperature=0.5,
        # max_tokens=100000,
        # frequency_penalty=0.0,
        # presence_penalty=0.0,
        # response_format="json_object",
    ),
)


async def main():
    # non-streaming response
    # response = await agent.do(
    #     prompt="tell me what is the meaning of life?",
    # )
    # print(response)

    # stream the response
    async for chunk in agent.do_stream(
        prompt="tell me what is the meaning of life?",
    ):
        print(chunk, end="", flush=True)


if __name__ == "__main__":
    asyncio.run(main())
```

## Multi-Agent/Provider Support

Arsenal supports multiple agents with multiple LLM providers:

```python
# OpenAI
agent = Agent(
    name="gpt_assistant",
    system_prompt="You are a helpful assistant",
    model="o3-mini",
    api_key=os.getenv("OPENAI_API_KEY"),
    provider="openai",
    config=config
)

# X-AI (Grok)
agent = Agent(
    name="grok_assistant",
    system_prompt="You are a helpful assistant",
    model="grok-2-1212",
    api_key=os.getenv("XAI_API_KEY"),
    provider="x-ai",
    config=config
)

# OpenRouter (access to many models including Claude)
agent = Agent(
    name="claude_assistant",
    system_prompt="You are a helpful assistant",
    model="anthropic/claude-3-opus",
    api_key=os.getenv("OPENROUTER_API_KEY"),
    provider="openrouter",
    config=config
)
```

## Using Search Skills

Arsenal includes built-in search capabilities:

```python
import asyncio
import os
from arsenalpy.agents import Agent, AgentConfig
from arsenalpy.skills.search_skill import search, SearchEngine

async def main():
    # Direct skill usage
    results = await search(
        q="What are the latest developments in AI?",
        engine=SearchEngine.BRAVE,
        rerank=True
    )

    print(f"Found {len(results['results'])} results")

    # Or create a search-capable agent
    agent = Agent(
        name="research_assistant",
        system_prompt="You are a research assistant with web search capabilities.",
        model="o3-mini",
        api_key=os.getenv("OPENAI_API_KEY"),
        provider="openai",
        skills=[search],
        config=AgentConfig(temperature=0.3)
    )

    # Ask a research question
    response = await agent.do("What are the latest advancements in quantum computing?")
    print(response)

if __name__ == "__main__":
    asyncio.run(main())
```

## Web Browsing

Extract content from web pages with the browse skill:

```python
import asyncio
import os
from arsenalpy.agents import Agent, AgentConfig
from arsenalpy.skills.browse_skill import browse, BrowseMode

async def main():
    # Direct skill usage
    content = await browse(
        url="https://example.com/article",
        mode=BrowseMode.DYNAMIC,
        extract_links=True
    )

    print(f"Page title: {content['title']}")
    print(f"Content sample: {content['content'][:200]}...")

    # Or create a browsing-capable agent
    agent = Agent(
        name="web_assistant",
        system_prompt="You are an assistant that can browse and summarize web pages.",
        model="o3-mini",
        api_key=os.getenv("OPENAI_API_KEY"),
        provider="openai",
        skills=[browse],
        config=AgentConfig(temperature=0.3)
    )

    # Ask the agent to browse and summarize
    response = await agent.do("Browse and summarize the content at https://example.com")
    print(response)

if __name__ == "__main__":
    asyncio.run(main())
```

## Creating Custom Skills

Arsenal supports custom skills that can be attached to agents:

```python
from arsenalpy.skills import skill
from pydantic import BaseModel, Field
from typing import List

class SummarizeInput(BaseModel):
    text: str = Field(..., description="Text to summarize")
    max_length: int = Field(100, description="Maximum length of the summary")
    key_points: bool = Field(True, description="Whether to include key points")

@skill(SummarizeInput)
async def summarize(text: str, max_length: int = 100, key_points: bool = True) -> dict:
    """Summarize the given text."""
    # Simple implementation (in a real skill, you'd use an API or algorithm)
    summary = text[:max_length] + "..." if len(text) > max_length else text

    result = {"summary": summary}

    if key_points:
        # Simple example key points extraction
        result["key_points"] = [text[i:i+10] for i in range(0, min(len(text), 50), 10)]

    return result

# Use with an agent
agent = Agent(
    name="skill_assistant",
    system_prompt="You are an assistant with text summarization capabilities.",
    model="o3-mini",
    api_key=os.getenv("OPENAI_API_KEY"),
    provider="openai",
    skills=[summarize],
    config=AgentConfig(temperature=0.3)
)

# The agent can now use the summarize skill
```

## Combining Multiple Skills

Create a powerful agent by combining multiple skills:

```python
import os
import asyncio
from arsenalpy.agents import Agent, AgentConfig
from arsenalpy.skills.search_skill import search
from arsenalpy.skills.browse_skill import browse

# Create a research agent combining search and browse skills
agent = Agent(
    name="research_assistant",
    system_prompt="You are a research assistant. First search for information, then browse relevant pages for details.",
    model="o3-mini",
    api_key=os.getenv("OPENAI_API_KEY"),
    provider="openai",
    skills=[search, browse],
    config=AgentConfig(temperature=0.3)
)

# ⚠️ Note: When providing multiple skills to an agent (like search and browse together),
# the agent may not reliably use both skills in sequence. For more reliable sequential
# skill usage, see the advanced patterns in the core concepts documentation.

async def main():
    # Ask a complex research question
    response = await agent.do("Research the latest developments in renewable energy")
    print(response)

if __name__ == "__main__":
    asyncio.run(main())
```

## Advanced Configuration

Control how agents behave with detailed configuration:

```python
from arsenalpy.agents import AgentConfig

# Research configuration
research_config = AgentConfig(
    temperature=0.3,  # More factual
    max_completion_tokens=2000,
    reasoning_effort="high",
    tool_choice="auto"  # Let the model decide when to use skills
)

# Creative configuration
creative_config = AgentConfig(
    temperature=0.8,  # More creative
    max_completion_tokens=1000,
    frequency_penalty=0.5,  # Reduce repetition
    tool_choice="none"  # Don't use skills
)

# JSON output configuration
json_config = AgentConfig(
    temperature=0.2,
    response_format={"type": "json_object"},
    tool_choice="required"  # Always try to use skills
)
```

## Working with Images

Arsenal supports multimodal inputs:

```python
# Image analysis
response = await agent.do(
    prompt="Describe this image in detail",
    image_url="https://example.com/image.jpg"
)
```

## Next Steps

Now that you're familiar with the basics, explore:

- [Detailed Examples](../examples/index.md) - Real-world use cases
- [Core Concepts](../core-concepts/agents.md) - Learn about agents, skills, and configuration
- [API Reference](../api/agents.md) - Complete API documentation
