# Configuration

Arsenal provides a flexible configuration system through the `AgentConfig` class, which allows you to customize how agents interact with AI models.

## AgentConfig

The `AgentConfig` class enables fine-grained control over agent behavior:

```python
from arsenalpy.agents import AgentConfig

config = AgentConfig(
    temperature=0.7,
    max_completion_tokens=1000,
    top_p=0.95,
    frequency_penalty=0.0,
    presence_penalty=0.0,
    response_format={"type": "json_object"},
    reasoning_effort="high",
    store=False,
    tool_choice="auto"
)
```

## Core Parameters

### Temperature

Controls randomness in the AI's responses:

```python
# More deterministic, factual responses
config = AgentConfig(temperature=0.2)

# More creative, varied responses
config = AgentConfig(temperature=0.8)
```

Higher values (0.7-1.0) produce more creative and diverse outputs, while lower values (0.1-0.3) produce more predictable and factual responses.

### Max Completion Tokens

Limits the length of responses:

```python
# Shorter responses
config = AgentConfig(max_completion_tokens=500)

# Longer, more detailed responses
config = AgentConfig(max_completion_tokens=2000)
```

Setting this parameter helps control costs and ensures responses don't exceed desired lengths.

### Top P (Nucleus Sampling)

Controls diversity by considering only the top P probability mass:

```python
config = AgentConfig(top_p=0.9)
```

Lower values make the output more focused on likely tokens, while higher values allow for more diverse outputs.

### Response Format

Specifies the format for responses:

```python
# JSON responses
config = AgentConfig(response_format={"type": "json_object"})

# Free-form text (default)
config = AgentConfig(response_format=None)
```

This is particularly useful when you need structured data from the AI.

### Tool Choice

Controls how agents use skills:

```python
# Let the model decide (default)
config = AgentConfig(tool_choice="auto")

# Never use tools
config = AgentConfig(tool_choice="none")

# Always try to use a tool
config = AgentConfig(tool_choice="required")

# Force the use of a specific tool
config = AgentConfig(tool_choice={
    "function": {"name": "search"}
})
```

This parameter gives you fine-grained control over how and when tools are used.

## Advanced Parameters

### Reasoning Effort

Specifies how much reasoning the model should apply:

```python
# Higher effort for complex questions
config = AgentConfig(reasoning_effort="high")
```

Options include "low", "medium", and "high".

### Frequency and Presence Penalties

Control repetition in responses:

```python
# Reduce repetition of specific tokens
config = AgentConfig(frequency_penalty=0.5)

# Discourage the model from repeating the same topics
config = AgentConfig(presence_penalty=0.5)
```

Positive values reduce repetition, while negative values increase it.

### Store

Enables conversation storage:

```python
# Store conversation history
config = AgentConfig(store=True)
```

## Configuration for Specific Use Cases

### Research Agent

```python
research_config = AgentConfig(
    temperature=0.3,          # More factual
    max_completion_tokens=2000, # Longer responses
    reasoning_effort="high",   # Thorough analysis
    tool_choice="auto"         # Let model decide when to use tools
)
```

### Creative Assistant

```python
creative_config = AgentConfig(
    temperature=0.8,          # More creative
    max_completion_tokens=1500, # Medium-length responses
    frequency_penalty=0.3,    # Reduce repetition
    tool_choice="none"        # No tools needed
)
```

### Data Analyst

```python
analyst_config = AgentConfig(
    temperature=0.1,          # Very factual
    response_format={"type": "json_object"}, # Structured output
    reasoning_effort="high",  # Thorough analysis
    tool_choice="required"    # Always use tools when available
)
```


## Combining Configuration with System Prompts

For optimal results, align your configuration with your system prompt:

```python
# Configuration for a precise coding assistant
config = AgentConfig(
    temperature=0.2,
    reasoning_effort="high",
    tool_choice="auto"
)

# System prompt that aligns with the configuration
system_prompt = """You are an expert coding assistant that:
1. Writes precise, error-free code
2. Explains your reasoning step by step
3. Uses tools when needed to verify solutions
Always prioritize correctness over creativity."""

# Create the agent
agent = Agent(
    name="coding_assistant",
    system_prompt=system_prompt,
    model="o3-mini",
    api_key="your-api-key",
    config=config,
    # ... other parameters
)
```

By thoughtfully combining configuration settings with appropriate system prompts, you can create agents that are precisely tailored to specific tasks.
