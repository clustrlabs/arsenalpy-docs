# Understanding Agents

## What is an Agent?

In Arsenal, an agent is an AI-powered assistant that can understand natural language, perform tasks, and use specialized skills. Each agent is configured with:

- A name and system prompt that defines its personality and behavior
- A specific AI model (like GPT-4, Claude, or Grok)
- Optional skills that extend its capabilities
- Configuration settings that control its behavior

## Agent Components

### System Prompt

The system prompt is crucial in defining how your agent behaves. It sets the context and role for the AI:

```python
agent = Agent(
    name="research_assistant",
    system_prompt="You are a research assistant specialized in scientific papers.
                  Always provide citations and be thorough in your analysis.",
    # ... other configuration
)
```

### Skills

Skills are specialized functions that agents can use to perform specific tasks:

```python
from arsenalpy.skills.search_skill import search
from arsenalpy.skills.browse_skill import browse

agent = Agent(
    name="researcher",
    system_prompt="You are a research assistant",
    skills=[search, browse],  # Add web capabilities
    # ... other configuration
)
```

Arsenal supports different ways to provide skills to agents:

1. **Direct Skill Functions**: Add the skill function directly
2. **Configured Skill Calls**: Pre-configured skills with specific parameters
3. **Custom Callable Skills**: Your own functions with `skill_definition` attributes
4. **Skill Dictionaries**: Skill definitions as raw dictionaries

### Important Notes on Using Multiple Skills

While you can provide multiple skills to an agent as shown above, there are some important limitations to be aware of:

1. **Sequential Tool Usage**: When an agent has multiple skills (e.g., search and browse), there's no guarantee it will use both skills in sequence, even when instructed to do so in the system prompt.

2. **Model Limitations**: Most AI models tend to favor using one skill per interaction, even when sequential use would be beneficial.

3. **Recommended Patterns**: For reliable sequential use of multiple skills (e.g., search then browse), there are three recommended approaches:

   a. **Separate Agents**: Use different agents for each skill and chain them together
   ```python
   search_agent = Agent(name="searcher", skills=[search], ...)
   browse_agent = Agent(name="browser", skills=[browse], ...)

   # Chain them in your application logic
   search_results = await search_agent.do("Search for renewable energy news")
   url = extract_url_from_results(search_results)
   browse_results = await browse_agent.do(f"Browse {url}")
   ```

   b. **Direct Function Calls**: Call skill functions directly rather than relying on the agent to choose them
   ```python
   # Create an agent with search only for the initial search
   search_agent = Agent(name="searcher", skills=[search], ...)
   search_results = await search_agent.do("Search for renewable energy news")

   # Extract URLs and call browse directly
   url = extract_url_from_results(search_results)
   browse_results = await browse(url=url, mode=BrowseMode.STATIC)

   # Use a synthesis agent to combine the results
   synthesis_agent = Agent(name="synthesizer", ...)
   final_result = await synthesis_agent.do(f"Combine search and browse results: {search_results}\n{browse_results}")
   ```

   c. **Forced Tool Choice**: Use the `tool_choice` parameter to force specific tool usage (not always reliable)
   ```python
   # Force search first
   search_config = AgentConfig(tool_choice={"type": "function", "function": {"name": "search"}})
   agent = Agent(skills=[search, browse], config=search_config, ...)
   search_results = await agent.do("Find information about renewable energy")

   # Then create a new agent to force browse (or update configuration)
   browse_config = AgentConfig(tool_choice={"type": "function", "function": {"name": "browse"}})
   browse_agent = Agent(skills=[browse], config=browse_config, ...)
   browse_results = await browse_agent.do(f"Browse URL from search results: {extract_url(search_results)}")
   ```


### Provider Support

Arsenal supports multiple LLM providers through a unified interface:

```python
# OpenAI
agent = Agent(
    name="gpt_assistant",
    provider="openai",
    model="gpt-4",
    api_key=os.getenv("OPENAI_API_KEY"),
    # ... other configuration
)

# X-AI (Grok)
agent = Agent(
    name="grok_assistant",
    provider="x-ai",
    model="grok-2-1212",
    api_key=os.getenv("XAI_API_KEY"),
    # ... other configuration
)

# OpenRouter (access to many models)
agent = Agent(
    name="claude_assistant",
    provider="openrouter",
    model="anthropic/claude-3-opus",
    api_key=os.getenv("OPENROUTER_API_KEY"),
    # ... other configuration
)
```

### Configuration

AgentConfig allows fine-tuning of the agent's behavior:

```python
config = AgentConfig(
    temperature=0.7,  # More creative responses
    max_completion_tokens=2000,  # Longer responses
    reasoning_effort="high",  # More thorough analysis
    tool_choice="auto"  # Let the model decide when to use tools
)
```

The `tool_choice` parameter is particularly important for controlling how an agent uses skills:

- `"auto"`: Let the model decide when to use skills (default)
- `"none"`: Never use skills, even if available
- `"required"`: Always try to use a skill
- `{"function": {"name": "skill_name"}}`: Force the use of a specific skill

## Interaction Methods

Agents provide two primary methods for interaction:

### Direct Responses (do)

```python
response = await agent.do("What is the capital of France?")
```

The `do` method:

- Takes a prompt and optional image URL
- Handles skill execution automatically
- Returns the final response as a string

### Streaming Responses (do_stream)

```python
async for chunk in agent.do_stream("Tell me about quantum physics"):
    print(chunk, end="", flush=True)
```

The `do_stream` method:

- Provides the same functionality as `do` but streams the response
- Yields chunks of the response as they're generated
- Improves user experience for longer responses

## Multimodal Capabilities

Agents can process text and images together:

```python
response = await agent.do(
    prompt="What can you see in this image?",
    image_url="https://example.com/image.jpg"
)
```

## Easy Debugging

Because Arsenal uses OpenAI's client, you can export the environment variable `OPENAI_LOG` to get more information about the API calls made by Arsenal. For example:

```bash
export OPENAI_LOG=debug
```

## Best Practices

1. **Clear System Prompts**: Write clear, specific system prompts that define the agent's role and constraints.

2. **Appropriate Temperature**: Use lower temperature (0.1-0.3) for factual tasks, higher (0.6-0.8) for creative ones.

3. **Relevant Skills**: Only attach skills that the agent needs for its specific role.

4. **Error Handling**: Always handle potential API errors and token limits gracefully.

5. **Tool Control**: Use the `tool_choice` parameter to control when and how skills are used.
