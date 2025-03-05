# Skills API Reference

## Skill Decorator

The `@skill` decorator is used to create callable skills that can be attached to agents.

```python
from arsenalpy.skills import skill
from pydantic import BaseModel

class SkillInput(BaseModel):
    parameter: str
    value: int

@skill(SkillInput)
async def my_skill(parameter: str, value: int) -> str:
    return f"Processed {parameter} with value {value}"
```

### Parameters

- Input Model (BaseModel): A Pydantic model defining the skill's input parameters

### Returns

- Callable: The decorated skill function that can be attached to an agent
