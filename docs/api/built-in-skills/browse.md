# Browse Skill

The browse skill allows Arsenal agents to extract content from web pages.

## Overview

This skill uses Crawl4AI to browse and extract content from web pages. It supports different browsing modes for static and dynamic websites and can handle JavaScript-heavy sites.

## Usage

### Direct Usage

You can use the browse skill directly in your code:

```python
import asyncio
from arsenalpy.skills.browse_skill import browse, BrowseMode

async def main():
    result = await browse(
        url="https://example.com/article",
        mode=BrowseMode.DYNAMIC,  # For JavaScript-heavy sites
        extract_links=True         # Also extract links from the page
    )

    print(f"Title: {result['title']}")
    print(f"Content sample: {result['content'][:200]}...")

    if result['links']:
        print("\nLinks found:")
        for link in result['links'][:5]:
            print(f"- {link['text']}: {link['url']}")

if __name__ == "__main__":
    asyncio.run(main())
```

### With Agents

To give an agent browsing capabilities:

```python
import os
from arsenalpy.agents import Agent, AgentConfig
from arsenalpy.skills.browse_skill import browse

agent = Agent(
    name="web_assistant",
    system_prompt="You are an assistant that can browse and summarize web pages.",
    model="gpt-4",
    api_key=os.getenv("OPENAI_API_KEY"),
    provider="openai",
    skills=[browse],
    config=AgentConfig(temperature=0.3)
)

# The agent can now browse web pages
response = await agent.do("Browse and summarize the content at https://example.com")
```

## API Reference

```python
async def browse(
    url: str,
    mode: BrowseMode = BrowseMode.STATIC,
    javascript: Optional[List[str]] = None,
    wait_for_selector: Optional[str] = None,
    extract_links: bool = False,
    content_threshold: float = 0.5,
    visible_browser: bool = False
) -> Dict[str, Any]
```

### Parameters

- `url` (str): The URL to browse
- `mode` (BrowseMode): The browsing mode (default: STATIC)
  - Options: `BrowseMode.STATIC`, `BrowseMode.DYNAMIC`, `BrowseMode.INTERACTIVE`
- `javascript` (List[str], optional): Custom JavaScript to execute on the page
- `wait_for_selector` (str, optional): CSS selector to wait for before extracting content
- `extract_links` (bool): Whether to extract links from the page (default: False)
- `content_threshold` (float): Threshold for content filtering (default: 0.5)
- `visible_browser` (bool): Whether to show the browser window (default: False)

### Returns

- Dictionary containing:
  - `url`: The URL that was browsed
  - `title`: The page title
  - `content`: The extracted page content
  - `links`: List of links if extract_links=True (each with 'text' and 'url' fields)

## Installation Requirements

The browse skill requires the Crawl4AI package:

```bash
pip install "arsenalpy[all]"
```

## Browsing Modes

The skill supports three browsing modes:

- **STATIC**: For simple HTML pages without JavaScript dependencies (fastest)
- **DYNAMIC**: For JavaScript-heavy pages that require rendering (recommended for most sites)
- **INTERACTIVE**: For pages that require user interaction or complex JavaScript execution
