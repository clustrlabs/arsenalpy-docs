# Understanding Skills

## What are Skills?

Skills in Arsenal are specialized functions that extend an agent's capabilities beyond basic conversation. They allow agents to:

- Search the web for information
- Browse and extract content from websites
- Interact with external APIs
- Perform complex calculations
- Access databases
- Execute blockchain transactions
- And more...

## Anatomy of a Skill

A skill consists of three main components:

1. **Input Model**: A Pydantic model defining the parameters
2. **Function Logic**: The actual implementation
3. **Skill Decorator**: Metadata that helps the agent understand the skill

```python
from arsenal.skills import skill
from pydantic import BaseModel

# 1. Input Model
class WeatherInput(BaseModel):
    city: str
    country: str = "US"

# 2. & 3. Function Logic with Decorator
@skill(WeatherInput)
async def get_weather(city: str, country: str) -> str:
    """Get weather information for a specific city."""
    # Implementation here
    return f"Weather in {city}, {country}: ..."
```

## Types of Skills

### Built-in Skills

Arsenal comes with several built-in skills:

1. **Search**: Web search using multiple engines (Brave, Serper, Tavily)
2. **Browse**: Extract content from web pages with support for static and dynamic sites
3. **Embedding**: Generate embeddings from text
4. **Utils**: Common utility functions and decorators


### Community Skills

Community-maintained skills like:

1. **PDF Reader**: Document parsing
2. **Image Generation**: AI image creation
3. **Vector Database**: Embedding storage and retrieval
4. **EVM Wallet**: Blockchain transactions
5. **SQLite Database**: Local database operations

## Core Skills

### Search

The search skill allows agents to search the web using various search engines:

```python
from arsenalpy.skills.search_skill import search, SearchEngine

# Direct usage
results = await search(
    q="Latest developments in quantum computing",
    engine=SearchEngine.BRAVE,
    rerank=True
)

# With an agent
agent = Agent(
    # ... agent configuration ...
    skills=[search],
)

response = await agent.do("What are the latest developments in quantum computing?")
```

The search skill:

- Supports multiple search engines (Brave, Serper, Tavily)
- Can rerank results using Cohere for better relevance
- Optionally rewrites queries using LLM for better search results
- Returns standardized results across different engines

### Browse

The browse skill extracts content from web pages:

```python
from arsenalpy.skills.browse_skill import browse, BrowseMode

# Direct usage
content = await browse(
    url="https://example.com/article",
    mode=BrowseMode.DYNAMIC,
    extract_links=True
)

# With an agent
agent = Agent(
    # ... agent configuration ...
    skills=[browse],
)

response = await agent.do("Browse https://example.com/article and summarize it")
```

The browse skill:

- Supports different browsing modes (static, dynamic, interactive)
- Extracts clean content from messy web pages
- Can capture links for further browsing
- Handles JavaScript-heavy sites with dynamic mode


## Creating Custom Skills

### Basic Structure

```python
from arsenalpy.skills import skill
from pydantic import BaseModel

class SkillInput(BaseModel):
    parameter1: str
    parameter2: int

@skill(SkillInput)
async def my_custom_skill(parameter1: str, parameter2: int) -> str:
    # Implement your logic here
    return "Result"
```


### Example: Web API Skill

```python
from arsenalpy.skills import skill
from pydantic import BaseModel, Field
import httpx
from typing import Dict, Any, Optional

class APIRequestInput(BaseModel):
    endpoint: str = Field(..., description="API endpoint to call")
    method: str = Field("GET", description="HTTP method (GET, POST, PUT, etc.)")
    params: Optional[Dict[str, Any]] = Field(None, description="Query parameters")
    data: Optional[Dict[str, Any]] = Field(None, description="Request body (for POST/PUT)")
    headers: Optional[Dict[str, str]] = Field(None, description="HTTP headers")

@skill(APIRequestInput)
async def call_api(
    endpoint: str,
    method: str = "GET",
    params: Optional[Dict[str, Any]] = None,
    data: Optional[Dict[str, Any]] = None,
    headers: Optional[Dict[str, str]] = None
) -> Dict[str, Any]:
    """
    Make a request to an external API.

    This skill demonstrates how to create a wrapper for external API calls.
    """
    default_headers = {
        "Content-Type": "application/json",
        "User-Agent": "Arsenal/1.0"
    }

    # Merge with custom headers
    all_headers = {**default_headers, **(headers or {})}

    try:
        async with httpx.AsyncClient() as client:
            response = await client.request(
                method=method,
                url=endpoint,
                params=params,
                json=data,
                headers=all_headers,
                timeout=30.0
            )

            response.raise_for_status()

            return {
                "status_code": response.status_code,
                "data": response.json() if response.headers.get("content-type") == "application/json" else response.text,
                "headers": dict(response.headers),
                "success": True
            }
    except Exception as e:
        return {
            "success": False,
            "error": str(e),
            "status_code": getattr(e, "status_code", None)
        }
```
### Best Practices

1. **Input Validation**: Use Pydantic for robust input validation
2. **Async First**: Make skills async for better performance
3. **Error Handling**: Implement proper error handling and fallbacks
4. **Documentation**: Provide clear docstrings with examples
5. **Standardized Returns**: Return structured data with consistent formats
