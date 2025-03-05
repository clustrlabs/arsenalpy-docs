# Arsenal Examples

This section contains real-world examples of Arsenal in action, demonstrating various capabilities and use cases.

## Minimal Agent

A simple example showing how to set up a basic agent with Arsenal:

```python
import os
from dotenv import load_dotenv
from arsenalpy.agents.agent import Agent, AgentConfig
import asyncio

load_dotenv()

agent = Agent(
    name="basic_agent",
    provider="openrouter",
    system_prompt="You are a helpful assistant that can answer questions and help with tasks.",
    model="x-ai/grok-2-1212",
    api_key=os.getenv("OPENROUTER_API_KEY"),
    config=AgentConfig(),
)

async def main():
    # stream the response
    async for chunk in agent.do_stream(
        prompt="tell me what is the meaning of life?",
    ):
        print(chunk, end="", flush=True)

if __name__ == "__main__":
    asyncio.run(main())
```

This example demonstrates:

- Basic agent setup with OpenRouter provider
- Streaming response method
- Using environment variables for API keys

## Minimal Vision Agent

Shows how to use Arsenal with vision capabilities:

```python
import os
from dotenv import load_dotenv
from arsenalpy.agents.agent import Agent, AgentConfig
import asyncio

load_dotenv()

agent = Agent(
    name="basic_agent",
    provider="openrouter",
    system_prompt="You are a helpful assistant that can answer questions and help with tasks.",
    model="openai/o3-mini",
    api_key=os.getenv("OPENROUTER_API_KEY"),
    config=AgentConfig(response_format={"type": "json_object"}),
)

async def main():
    # stream the response
    async for chunk in agent.do_stream(
        prompt="tell me what you see in the image and describe it in detail and in json format",
        image_url="https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
    ):
        print(chunk, end="", flush=True)

if __name__ == "__main__":
    asyncio.run(main())
```

This example demonstrates:

- Using vision capabilities with image input
- Streaming responses with image analysis
- JSON formatting for structured outputs

## Using Skills as Functions

Shows how to use Arsenal skills directly as standalone functions:

```python
# Use a skill as a function like a normal function tavily as an example
from arsenal.skills.tavily_search import web_search
import asyncio
from dotenv import load_dotenv

load_dotenv()

async def main():
    search_results = await web_search(query="latest news on the stock market")
    print(search_results)

if __name__ == "__main__":
    asyncio.run(main())
```

This example demonstrates:

- Using skills as standalone functions
- The search skill functionality
- Processing and displaying search results

## FastAPI Integration

Example of integrating Arsenal with FastAPI to create an API endpoint:

```python
# api agent
import os
from dotenv import load_dotenv
from arsenalpy.agents.agent import Agent, AgentConfig
from arsenalpy.skills.search_skill import search
from fastapi import FastAPI
from pydantic import BaseModel

import uvicorn

load_dotenv()

agent = Agent(
    name="fastapi_agent",
    provider="openrouter",
    model="x-ai/grok-2-1212",
    api_key=os.getenv("OPENROUTER_API_KEY"),
    config=AgentConfig(),
    system_prompt="You are a helpful API agent.",
    skills=[search],
)

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "I am a fastapi agent"}

class QueryRequest(BaseModel):
    query: str

@app.post("/query")
async def query_agent(req: QueryRequest):
    result = await agent.do(prompt=req.query)
    return {"result": result}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=3000)
```

This example demonstrates:

- Integrating Arsenal with FastAPI
- Creating API endpoints for AI assistant functionality
- Handling requests and responses in an API context

## Deep Research Agent

Example of a sophisticated agent that combines search and web browsing for in-depth research:

```python
# deep research agent - combines search and browse capabilities for crypto token analysis
import os
from dotenv import load_dotenv
from arsenalpy.agents.agent import Agent, AgentConfig
from arsenalpy.skills.search_skill import search
from arsenalpy.skills.browse_skill import browse, BrowseMode
import asyncio
from datetime import datetime, timedelta

load_dotenv()

async def main():
    # Get current date for time-relevant searches
    current_date = datetime.now()
    last_week = current_date - timedelta(days=7)
    date_str = current_date.strftime("%B %Y")
    week_str = f"{last_week.strftime('%d %B')} to {current_date.strftime('%d %B %Y')}"

    print(f"=== Deep Research: Solana (SOL) Weekly Token Trends ({week_str}) ===")
    # Demonstrate the deep research process

    print("\n> Step 1: Searching for current Solana token trends...")

    # First search for current Solana price and trends
    search_query1 = (
        f"Solana SOL current price analysis trend prediction {date_str} week"
    )
    print(f"\nSearch query 1: '{search_query1}'")
    search_results1 = await search(q=search_query1, rerank=True, engine="brave")

    # Second search for Solana ecosystem tokens and projects
    search_query2 = (
        f"top performing Solana ecosystem tokens projects {date_str} weekly analysis"
    )
    print(f"\nSearch query 2: '{search_query2}'")
    search_results2 = await search(q=search_query2, rerank=True, engine="brave")

    # Third search for technical indicators and on-chain data
    search_query3 = (
        f"Solana SOL technical analysis on-chain data trading volume {week_str}"
    )
    print(f"\nSearch query 3: '{search_query3}'")
    search_results3 = await search(q=search_query3, rerank=True, engine="brave")

    # Combine results from all searches
    all_results = (
        search_results1["results"]
        + search_results2["results"]
        + search_results3["results"]
    )

    # Remove duplicates based on URL
    unique_results = []
    seen_urls = set()
    for result in sorted(
        all_results, key=lambda x: x.get("relevance_score", 0), reverse=True
    ):
        if result["link"] not in seen_urls:
            unique_results.append(result)
            seen_urls.add(result["link"])

    # Step 2: Browse top results to get full content
    print("\n> Step 2: Browsing top results to extract full content...")

    browse_results = []
    # Browse top results, focusing on crypto analysis sites
    for i, result in enumerate(unique_results[:10], 1):
        url = result["link"]
        print(f"\nBrowsing result {i}: {result['title']}")

        try:
            browse_result = await browse(
                url=url,
                mode=BrowseMode.DYNAMIC,  # Use dynamic mode for JS-heavy crypto sites
                extract_links=True,
                content_threshold=0.4,     # Lower threshold to keep more content
                visible_browser=True,      # Show the browser window during browsing
            )

            if browse_result["success"]:
                browse_results.append(browse_result)
                # Output details about the browsed content
            else:
                print(f"✗ Failed to browse {url}: {browse_result['error']}")
        except Exception as e:
            print(f"✗ Error browsing {url}: {str(e)}")

    # Step 3: Analyze all gathered information with an expert agent
    print("\n> Step 3: Analyzing all gathered information...")

    # Create a comprehensive prompt with all gathered information
    analysis_prompt = f"""I've collected the latest information about Solana (SOL) token trends for the week of {week_str}..."""

    # Create content analysis agent
    analysis_agent = Agent(
        name="crypto_analyst",
        provider="openrouter",
        system_prompt="""You are an expert cryptocurrency analyst specializing in token trends, price analysis, and trading strategies.""",
        model="anthropic/claude-3.7-sonnet",
        api_key=os.getenv("OPENROUTER_API_KEY"),
        config=AgentConfig(temperature=0.2, max_tokens=4000),
    )

    print(f"\n=== Solana (SOL) Weekly Token Analysis: {week_str} ===\n")
    async for chunk in analysis_agent.do_stream(prompt=analysis_prompt):
        print(chunk, end="", flush=True)

if __name__ == "__main__":
    asyncio.run(main())
```

This example demonstrates:

- Combining multiple search queries for comprehensive information gathering
- Using the browse skill to extract detailed content from webpages
- Processing and analyzing information with an expert agent
- Working with dynamic web content and JS-heavy sites
- Creating sophisticated research workflows
