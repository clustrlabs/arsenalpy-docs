# Developing Community Skills

Arsenal is designed to be extended with community skills. These are extensions that enhance Arsenal's capabilities without being part of the core library.

## Creating a Standalone Skill Package

A community skill package follows this structure:

```
arsenalpy_skillname/
├── src/
│   └── arsenalpy_skillname/
│       ├── __init__.py        # Exports the skill functions
│       └── skill_functions.py # Implementation
├── tests/                     # Tests for the skill
├── pyproject.toml            # Package configuration
└── README.md                 # Documentation
```

## Skill Package Configuration

Here's an example `pyproject.toml` for a community skill package:

```toml
[project]
name = "arsenalpy-skillname"
version = "0.1.0"
description = "SkillName extension for Arsenal"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "arsenalpy>=0.1.0",
    # Other dependencies
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

## Implementing a Community Skill

A skill is essentially a function decorated with the `@skill` decorator from Arsenal:

```python
from arsenalpy.skills import skill
from pydantic import BaseModel, Field
from typing import Dict, Any

class MySkillInput(BaseModel):
    param1: str = Field(..., description="First parameter")
    param2: int = Field(10, description="Second parameter with default")

@skill(MySkillInput)
async def my_skill(param1: str, param2: int = 10) -> Dict[str, Any]:
    """
    Description of what my skill does.

    Args:
        param1: Description of param1
        param2: Description of param2, default 10

    Returns:
        Dictionary with results
    """
    # Skill implementation
    return {
        "result": f"Processed {param1} with parameter {param2}"
    }
```

## Testing Community Skills

Test your community skills using the same principles as core testing:

```python
import pytest
from unittest.mock import patch, MagicMock

from arsenalpy_skillname import my_skill

@pytest.mark.asyncio
async def test_my_skill():
    # Simple test
    result = await my_skill("test", 20)
    assert result["result"] == "Processed test with parameter 20"

@pytest.mark.asyncio
@patch("arsenalpy_skillname.skill_functions.external_api")
async def test_my_skill_with_mock(mock_api):
    # Mock external dependencies
    mock_api.return_value = {"data": "mocked_data"}

    result = await my_skill("test")

    assert "mocked_data" in result["result"]
    mock_api.assert_called_once_with("test")
```

## Publishing Community Skills

1. Ensure your package has comprehensive tests and documentation
2. Package your skill:
   ```bash
   uv build
   ```
3. Publish to PyPI:
   ```bash
   uv publish
   ```

## Integrating Community Skills with Arsenal

Users can install and use your skill as follows:

```python
# Install the skill
uv install arsenalpy-skillname

# Use in code
from arsenalpy_skillname import my_skill
from arsenalpy.agents import Agent

agent = Agent(
    name="agent_with_custom_skill",
    system_prompt="You are a helpful assistant with access to custom skills.",
    model="o3-mini",
    api_key="your-api-key",
    provider="openai",
    skills=[my_skill]
)
```
