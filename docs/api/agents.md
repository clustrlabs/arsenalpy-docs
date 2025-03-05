# Agents API Reference

## Agent Class

The `Agent` class is the core component of Arsenal, providing a unified interface for interacting with AI models.

### Constructor

```python
def __init__(
    name: str,
    system_prompt: str,
    model: str,
    api_key: str,
    provider: Literal["openai", "x-ai", "openrouter"] = "openrouter",
    config: AgentConfig = None,
    skills: Optional[List] = None
) -> None
```

#### Parameters

- `name` (str): The name of the agent
- `system_prompt` (str): The system prompt that defines the agent's behavior
- `model` (str): The AI model to use (e.g., "gpt-4", "claude-3", "grok-2-1212")
- `api_key` (str): API key for the provider
- `provider` (str): The AI provider to use ("openai", "x-ai", or "openrouter")
- `config` (AgentConfig): Configuration settings for the agent
- `skills` (Optional[List]): List of skills available to the agent. Skills can be:
  - `Callable` functions with a `skill_definition` attribute
  - `ConfiguredSkillCall` instances
  - `dict` objects representing skill schemas
  - `Skill` model instances

  **Note**: When providing multiple skills to an agent (e.g., `skills=[search, browse]`), there's no guarantee the agent will use both skills in sequence even with explicit instructions. For reliable sequential skill usage, consider:
  1. Using separate agents for each skill
  2. Calling skill functions directly and passing results between steps
  3. Using the `tool_choice` parameter to force specific tool usage

  See the [core concepts documentation](../core-concepts/agents.md#important-notes-on-using-multiple-skills) for detailed patterns and examples.

### Methods

#### do

```python
async def do(
    prompt: str,
    image_url: Optional[str] = None
) -> str
```

Executes a single interaction with the AI model. This method handles both direct responses and tool-based interactions. It will automatically execute any requested tools and provide their results back to the AI for final response generation.

#### Parameters

- `prompt` (str): The user's input prompt
- `image_url` (Optional[str]): URL of an image to include in the prompt

#### Returns

- `str`: The AI's response

#### Raises

- `Exception`: If the API response is invalid or contains errors
- `ValueError`: If function arguments are invalid

#### do_stream

```python
async def do_stream(
    prompt: str,
    image_url: Optional[str] = None
) -> AsyncGenerator[str, None]
```

Stream the AI's response token by token. This method provides the same functionality as `do()` but streams the response as it's generated, which can provide a better user experience for longer responses.

#### Parameters

- `prompt` (str): The user's input prompt
- `image_url` (Optional[str]): URL of an image to include in the prompt

#### Returns

- `AsyncGenerator[str, None]`: Generator yielding response tokens

#### Raises

- `Exception`: If the API response is invalid or contains errors
- `ValueError`: If function arguments are invalid

## AgentConfig Class

Configuration settings for customizing agent behavior.

```python
class AgentConfig(BaseModel):
    temperature: Optional[float] = 0.5
    max_completion_tokens: Optional[int] = None
    top_p: Optional[float] = 1.0
    frequency_penalty: Optional[float] = 0.0
    presence_penalty: Optional[float] = 0.0
    response_format: Optional[Dict[str, Any]] = None
    reasoning_effort: Optional[Literal["medium", "high", "low"]] = None
    store: Optional[bool] = False
    tool_choice: Optional[Union[Literal["none", "auto", "required"], Dict[str, Any]]] = "auto"
```

### Parameters

- `temperature` (float): Controls randomness in the output (0.0 to 1.0). Higher values like 0.8 will make the output more random, while lower values like 0.2 will make it more focused and deterministic.
- `max_completion_tokens` (int): Maximum number of tokens in the completion. Useful for controlling response length.
- `top_p` (float): Nucleus sampling parameter. An alternative to temperature, used to determine how the model selects tokens for output.
- `frequency_penalty` (float): Penalty for frequent token use. Positive values discourage the model from repeating the same tokens.
- `presence_penalty` (float): Penalty for repeated topics. Positive values encourage the model to talk about new topics.
- `response_format` (Dict[str, Any]): Format for the response, such as `{"type": "json_object"}` for JSON responses.
- `reasoning_effort` (str): Level of reasoning effort ("low", "medium", or "high").
- `store` (bool): Whether to store the conversation.
- `tool_choice` (Union[str, Dict]): Controls how tools are selected. Can be "auto", "none", "required", or a specific tool configuration dict.

### Methods

#### to_api_params

```python
def to_api_params(self) -> dict
```

Converts the configuration to API parameters for use with LLM providers.

#### Returns

- `dict`: A dictionary of parameters formatted for API requests

## Handling Images

The Agent class can process images by providing an image URL:

```python
# Image analysis
response = await agent.do(
    prompt="Describe what you see in this image",
    image_url="https://example.com/path/to/image.jpg"
)
```

## Provider Selection

The Agent supports multiple LLM providers:

- `openai`: OpenAI's models like GPT-4
- `x-ai`: X.AI's models like Grok
- `openrouter`: OpenRouter for access to various models

```python
# Using OpenRouter with Anthropic's Claude
agent = Agent(
    name="claude_assistant",
    system_prompt="You are a helpful assistant powered by Claude.",
    model="anthropic/claude-3-opus",
    api_key=os.getenv("OPENROUTER_API_KEY"),
    provider="openrouter",
    config=config
)
```
