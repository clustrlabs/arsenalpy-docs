# Developing Core Components

This guide is for developers who want to contribute to Arsenal's core components.

## Setting Up Your Development Environment

1. Clone the repository:

```bash
git clone https://github.com/clustrlabs/arsenalpy.git
cd arsenalpy
```

2. Install development dependencies:

```bash
uv install -e ".[dev]"
```

This installs Arsenal in development mode along with all tools needed for development (black, isort, mypy, ruff, pytest, etc.).

## Project Structure

The core Arsenal project is structured as follows:

```
core/
├── src/
│   └── arsenalpy/
│       ├── agents/            # Agent implementation
│       ├── skills/            # Built-in skills
│       └── utils/             # Utility functions
├── tests/                     # Test suite
└── pyproject.toml            # Project configuration
```

## Coding Standards

Arsenal follows these coding standards:

- **Code Formatting**: Uses Black and isort for consistent formatting
- **Linting**: Uses Ruff for code quality checks
- **Type Checking**: Uses mypy for static type checking

We use pre-commit hooks to automatically run these checks before each commit. To set up pre-commit:

```bash
# Install pre-commit
uv install pre-commit

# Set up the hooks
pre-commit install
```

Once set up, the hooks will run automatically on each commit. You can also run them manually:

```bash
# Run all pre-commit hooks
pre-commit run --all-files

# Run a specific hook
pre-commit run mypy --all-files
```

The pre-commit configuration is defined in `.pre-commit-config.yaml` and includes all necessary tools (black, isort, ruff, mypy) with appropriate settings.

## Testing

Arsenal uses pytest for testing. The test suite is organized to mirror the project structure.

To run the tests:

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=arsenalpy

# Run specific tests
pytest tests/agents/
```

### Writing Tests

When writing tests:

- Place test files in the `tests/` directory following the same structure as the source code
- Use descriptive test names that explain what is being tested
- Mock external dependencies to avoid actual API calls during unit tests
- For integration tests, use small, representative test cases

Example test:

```python
import pytest
from unittest.mock import patch, MagicMock

from arsenalpy.agents import Agent

def test_agent_initialization():
    # Arrange
    agent = Agent(
        name="test_agent",
        system_prompt="Test prompt",
        model="test-model",
        api_key="fake-key",
        provider="openai"
    )

    # Assert
    assert agent.name == "test_agent"
    assert agent.system_prompt == "Test prompt"

@patch("arsenalpy.agents.agent.LLMClient")
async def test_agent_do_method(mock_client):
    # Arrange
    mock_instance = MagicMock()
    mock_instance.generate.return_value = "Test response"
    mock_client.return_value = mock_instance

    agent = Agent(
        name="test_agent",
        system_prompt="Test prompt",
        model="test-model",
        api_key="fake-key",
        provider="openai"
    )

    # Act
    response = await agent.do("Test query")

    # Assert
    assert response == "Test response"
    mock_instance.generate.assert_called_once()
```

## Adding a New Feature to Core

When adding a new feature to the core library:

1. Identify where it belongs in the project structure
2. Create necessary files and implement the feature
3. Add comprehensive tests
4. Update documentation
5. Submit a pull request
