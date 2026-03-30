# Agent Instructions for nanobot

> This file provides essential context for AI agents working in this repository.

## Project Overview

**nanobot** is an ultra-lightweight personal AI assistant framework written in Python.
- Package: `nanobot-ai`
- Python: >= 3.11
- License: MIT

## Build / Test / Lint Commands

### Install Dependencies
```bash
# Install with dev dependencies
pip install -e ".[dev]"

# Or using uv (used in CI)
uv sync --all-extras
```

### Run Tests
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/tools/test_filesystem_tools.py

# Run specific test
pytest tests/tools/test_filesystem_tools.py::TestReadFileTool::test_basic_read

# With coverage
pytest --cov=nanobot --cov-report=term-missing
```

### Lint & Format
```bash
# Check code style
ruff check nanobot/

# Auto-fix issues
ruff check --fix nanobot/

# Format code
ruff format nanobot/
```

### Run Application
```bash
# Interactive onboarding
nanobot onboard

# Start CLI chat
nanobot agent

# Start gateway (for chat channels)
nanobot gateway
```

## Code Style Guidelines

### Python Standards
- **Line length**: 100 characters (configured in pyproject.toml)
- **Target**: Python 3.11+
- **Type hints**: Use full type annotations (e.g., `dict[str, Any]`, `str | None`)
- **Imports**: Group stdlib first, then third-party, then local (isort style)

### Naming Conventions
- **Classes**: PascalCase (e.g., `AgentLoop`, `LLMProvider`)
- **Functions/Variables**: snake_case (e.g., `chat_with_retry`, `api_key`)
- **Constants**: UPPER_CASE (e.g., `_CHAT_RETRY_DELAYS`)
- **Private**: Leading underscore (e.g., `_is_transient_error`)
- **Abstract**: Use `@abstractmethod` for interfaces

### Project Patterns

#### Configuration (Pydantic)
```python
from pydantic import BaseModel, Field

class MyConfig(BaseModel):
    """Configuration with validation."""
    enabled: bool = True
    timeout: int = Field(default=30, ge=1)
```

#### Tools (Agent Tools)
```python
from nanobot.agent.tools.base import Tool

class MyTool(Tool):
    """Tool description."""
    
    @property
    def name(self) -> str:
        return "my_tool"
    
    @property
    def description(self) -> str:
        return "What this tool does"
    
    @property
    def parameters(self) -> dict[str, Any]:
        return {
            "type": "object",
            "properties": {"arg": {"type": "string"}},
            "required": ["arg"]
        }
    
    async def execute(self, **kwargs: Any) -> str:
        # Implementation
        return result
```

#### Error Handling
- Use `try/except` with specific exceptions
- Log errors with `logger.error()` or `logger.warning()`
- Never suppress exceptions silently
- For LLM calls, use retry logic with exponential backoff

#### Logging
```python
from loguru import logger

logger.info("Message {}", variable)
logger.warning("Warning: {}", details)
logger.error("Error occurred: {}", exc)
```

### Testing Guidelines
- Use `pytest` with `@pytest.mark.asyncio` for async tests
- Use fixtures for setup (e.g., `tmp_path` for filesystem tests)
- Tests are organized to mirror the `nanobot/` structure
- Async tests need `pytest-asyncio` (configured with `asyncio_mode = "auto"`)

### Branching Strategy
- **`main`**: Stable releases — bug fixes and minor improvements
- **`nightly`**: Experimental features — new features and breaking changes
- Target `nightly` for new features; target `main` for bug fixes

### Key Dependencies
- **Pydantic**: Configuration schemas and validation
- **Loguru**: Structured logging
- **Typer**: CLI commands
- **Asyncio**: All I/O is async
- **Ruff**: Linting and formatting (rules: E, F, I, N, W)

### File Organization
```
nanobot/
├── agent/          # Core agent logic (loop, tools, memory, skills)
├── channels/       # Chat platform integrations
├── providers/      # LLM provider implementations
├── config/         # Configuration schemas and loading
├── cli/            # Command-line interface
├── bus/            # Message routing
├── cron/           # Scheduled tasks
├── session/        # Conversation sessions
├── skills/         # Built-in agent skills
└── templates/      # Template files (MEMORY.md, AGENTS.md, etc.)
```

### Import Style
```python
# Standard library
import asyncio
from pathlib import Path

# Third-party
from loguru import logger
from pydantic import BaseModel

# Local (absolute imports)
from nanobot.agent.tools.base import Tool
from nanobot.config.schema import Config
```

### Async Patterns
- All I/O operations are async
- Use `asyncio.gather()` for parallel operations
- Handle `asyncio.CancelledError` explicitly when needed

### Security Notes
- Tools support `restrict_to_workspace` for sandboxing
- Never log API keys or secrets
- Validate all user inputs
- Use `safe_filename()` for filesystem operations

## Common Tasks

### Adding a New Provider
1. Add `ProviderSpec` to `nanobot/providers/registry.py`
2. Add field to `ProvidersConfig` in `nanobot/config/schema.py`
3. Create provider class inheriting from `LLMProvider`

### Adding a New Tool
1. Create tool class inheriting from `Tool`
2. Register in `nanobot/agent/tools/registry.py`
3. Add to `AgentLoop` tool registry

### Adding a New Channel
1. Create channel class inheriting from `BaseChannel`
2. Register in `nanobot/channels/registry.py`
3. Add config fields to `ChannelsConfig`
