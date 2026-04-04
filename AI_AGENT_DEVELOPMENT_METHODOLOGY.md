# AI Agent Development Methodology - Sensible Analytics

**Production-grade architecture for building maintainable, enterprise-ready AI agents.**

---

## Core Philosophy

> **No ad-hoc slop code.** Every AI agent must follow the same architectural standards as production microservices.

---

## 1. Architecture Patterns

### Coordinator-Worker Pattern (Recommended)

```
User Input
    ↓
Coordinator Agent (Planning & Delegation)
    ↓
├── Subagent A: Research/Analysis
├── Subagent B: Execution  
└── Subagent C: Validation
    ↓
Coordinator (Synthesis & Response)
```

### Supervisor Pattern (For Simpler Cases)

```
Supervisor Agent
    ├── Tool: Subagent A (as callable tool)
    ├── Tool: Subagent B (as callable tool)
    └── Tool: Subagent C (as callable tool)
```

### Handoff Pattern (For Escalation)

```
Agent A ──transfer_with_context──→ Agent B ──transfer_with_context──→ Agent C
```

---

## 2. Required Project Structure

```
agent-project/
├── src/
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── base.py           # BaseAgent with common functionality
│   │   ├── coordinator.py    # Main orchestration agent
│   │   └── subagents/        # Specialized subagents
│   │       ├── __init__.py
│   │       ├── research.py
│   │       ├── analysis.py
│   │       └── execution.py
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── search.py         # Each tool = one file
│   │   ├── database.py
│   │   └── api.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── input.py          # Pydantic input models
│   │   └── output.py         # Pydantic output models
│   ├── middleware/
│   │   ├── __init__.py
│   │   ├── retry.py          # Retry with backoff
│   │   ├── fallback.py       # Model fallback
│   │   └── observability.py  # LangSmith tracing
│   └── memory/
│       ├── __init__.py
│       └── checkpointer.py   # State persistence
├── tests/
│   ├── unit/
│   ├── integration/
│   └── agent/
│       └── test_agent_flow.py
├── pyproject.toml
└── README.md
```

---

## 3. Mandatory Quality Standards

### Dependency Injection (Required)

```python
from dataclasses import dataclass

@dataclass
class AgentDependencies:
    db: DatabaseConn
    api_client: APIClient
    user_id: int

agent = Agent(
    model="openai:gpt-4.1",
    deps_type=AgentDependencies,
    output_type=AgentResponse,
)
```

### Input/Output Validation (Required)

```python
from pydantic import BaseModel

class QueryInput(BaseModel):
    query: str
    max_results: int = 10

class QueryOutput(BaseModel):
    results: list[dict]
    confidence: float
```

### Error Handling Middleware (Required)

```python
from pydantic_ai import Agent

agent = Agent(
    model="openai:gpt-4.1",
    retries=3,  # Built-in retry
)
```

---

## 4. Testing Requirements

### Test with MockModel

```python
from pydantic_ai.models.test import TestModel

async def test_agent():
    with agent.override(model=TestModel()):
        result = await agent.run("test query")
        assert result.output == expected
```

### Test Dependencies

```python
class TestDeps(RealDependencies):
    async def get_db(self):
        return MockDatabase()

async def test_with_fake_deps():
    with agent.override(deps=TestDeps()):
        result = await agent.run("query")
```

### LLM-as-Judge for Quality

```python
# Evaluate agent responses for quality
evaluator = create_trajectory_match_evaluator()
evaluation = evaluator(
    outputs=agent_result,
    reference_outputs=expected_trajectory,
)
```

---

## 5. Observability (Required)

### Always Enable Tracing

```python
import os
os.environ["LANGSMITH_TRACING"] = "true"
os.environ["LANGSMITH_API_KEY"] = os.getenv("LANGSMITH_API_KEY")
```

### Log Structured Data

```python
import logging
logger = logging.getLogger(__name__)

logger.info("agent_called", extra={
    "agent": "coordinator",
    "input_tokens": usage.prompt_tokens,
    "output_tokens": usage.completion_tokens,
})
```

---

## 6. Security Standards

### No Hardcoded Keys

```python
# ❌ WRONG
api_key = "sk-xxx"

# ✅ CORRECT
from config import get_api_key
api_key = get_api_key("openai")
```

### Prompt Injection Guardrails

```python
# Sanitize user input before adding to prompts
def sanitize_input(user_input: str) -> str:
    # Remove potential injection patterns
    return user_input.replace("Ignore previous", "")
```

---

## 7. Human-in-the-Loop

For critical actions (DELETE, PAY, SEND_EMAIL):

```python
middleware=[
    HumanInTheLoopMiddleware(
        interrupt_on={"send_email": {}, "delete_data": {}},
    )
]
```

---

## 8. Agent Communication Protocol

### Context Handoff

```python
@tool
def transfer_to_agent(new_agent: str, context: dict) -> Command:
    return Command(
        goto=new_agent,
        update={"messages": [...context...]}
    )
```

### Tool as Agent

```python
research_agent = create_agent(model, tools=[...])

main_agent = create_agent(
    model,
    tools=[research_agent.as_tool()]  # Subagent as tool
)
```

---

## Anti-Patterns (Forbidden)

| ❌ Never Do This | ✅ Instead |
|-----------------|------------|
| Hardcode API keys | Use environment variables |
| No error handling | Add retry/fallback middleware |
| Untyped inputs/outputs | Use Pydantic models |
| No state persistence | Use checkpointer |
| Skip observability | Enable LangSmith |
| Direct LLM in tools | Use typed tools with schemas |
| Single monolithic agent | Decompose into coordinator + subagents |
| Skip testing | Use TestModel + dependency override |

---

## Quick Reference for AI Agents

When building ANY agent at Sensible Analytics:

```bash
# 1. Create project structure
mkdir -p src/agents src/tools src/schemas src/middleware tests/agent

# 2. Use Pydantic for I/O
@dataclass
class MyDeps:
    db: Database
    
agent = Agent(model="...", deps_type=MyDeps)

# 3. Add retry middleware
agent = Agent(model="...", retries=3)

# 4. Enable tracing
os.environ["LANGSMITH_TRACING"] = "true"

# 5. Test with TestModel
with agent.override(model=TestModel()):
    result = await agent.run("test")
```

---

**Remember:** Production AI agents are just as critical as production microservices. Apply the same rigor.