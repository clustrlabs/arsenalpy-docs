# Search Skill

The search skill allows Arsenal agents to search the web for information.

## Overview

This skill supports multiple search engines and provides a unified interface for web search. It includes options for query rewriting and result reranking to improve search quality.

## Usage

### Direct Usage

You can use the search skill directly in your code:

```python
import asyncio
from arsenalpy.skills.search_skill import search, SearchEngine

async def main():
    results = await search(
        q="Python programming best practices",
        engine=SearchEngine.BRAVE,
        llm_rewrite=True,  # Use LLM to optimize the query
        rerank=True        # Rerank results for better relevance
    )

    print(f"Search results for: {results['query']}")
    for result in results['standardized_results']:
        print(f"- {result['title']}: {result['link']}")
        print(f"  {result['snippet'][:100]}...\n")

if __name__ == "__main__":
    asyncio.run(main())
```

### With Agents

To give an agent search capabilities:

```python
import os
from arsenalpy.agents import Agent, AgentConfig
from arsenalpy.skills.search_skill import search

agent = Agent(
    name="research_assistant",
    system_prompt="You are a research assistant that can search for information.",
    model="gpt-4",
    api_key=os.getenv("OPENAI_API_KEY"),
    provider="openai",
    skills=[search],
    config=AgentConfig(temperature=0.3)
)

# The agent can now use search
response = await agent.do("Find information about quantum computing")
```

## API Reference

```python
async def search(
    q: str,
    engine: SearchEngine = SearchEngine.BRAVE,
    llm_rewrite: bool = True,
    rerank: bool = True
) -> Dict[str, Any]
```

### Parameters

- `q` (str): The search query
- `engine` (SearchEngine): The search engine to use (default: BRAVE)
  - Options: `SearchEngine.BRAVE`, `SearchEngine.SERPER`, `SearchEngine.TAVILY`
- `llm_rewrite` (bool): Whether to use LLM to optimize the query (default: True)
- `rerank` (bool): Whether to rerank results using Cohere's API (default: True)

### Returns

- Dictionary containing:
  - `query`: The original search query
  - `rewritten_query`: The optimized query (if llm_rewrite=True)
  - `raw_results`: The raw search results from the engine
  - `standardized_results`: Results in a unified format with these fields:
    - `title`: The title of the search result
    - `link`: The URL of the result
    - `snippet`: A text snippet from the result
    - `position`: The position in the search results
    - `relevance_score`: A relevance score (if rerank=True)

## Required Environment Variables

The search skill requires different API keys depending on the engine used:

```bash
# For Brave Search
export BRAVE_API_KEY=your_brave_key

# For Serper
export SERPER_API_KEY=your_serper_key

# For Tavily
export TAVILY_API_KEY=your_tavily_key

# For result reranking (if rerank=True)
export COHERE_API_KEY=your_cohere_key

# For query rewriting (if llm_rewrite=True)
export OPENAI_API_KEY=your_openai_key

# For Arsenal
export ARSENAL_API_KEY=your_arsenal_key
```
