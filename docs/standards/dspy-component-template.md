# AI Component Specification Template (DSPy-Style)

## Overview

| Field | Value |
|-------|-------|
| **Component Name** | `[ComponentName]` |
| **Owner** | `[Team/Owner]` |
| **Version** | `[Semantic Version]` |
| **Status** | `draft | active | deprecated` |
| **Created** | `[YYYY-MM-DD]` |

---

## 1. Signatures (Explicit Contracts)

### Input Signature

```python
class ComponentInput(Signature):
    """Input contract for this component."""
    query: str = Field(description="User query or request")
    context: Optional[dict] = Field(default=None, description="Additional context")
```

### Output Signature

```python
class ComponentOutput(Signature):
    """Output contract for this component."""
    result: str = Field(description="Primary result")
    confidence: float = Field(description="Confidence score 0-1")
    reasoning: Optional[str] = Field(description="Chain-of-thought reasoning")
```

---

## 2. Modules (Reusable Components)

### Primary Module

```python
class ComponentModule(Module):
    def __init__(self):
        # Initialize modules with signatures
        self.analyzer = ChainOfThought("query, context -> analysis")
        self.generator = Predict("analysis -> result, confidence")
    
    def forward(self, query: str, context: Optional[dict] = None) -> ComponentOutput:
        analysis = self.analyzer(query=query, context=context)
        return self.generator(analysis=analysis.reasoning)
```

### Module Configuration

| Module | Type | Signature | Config |
|--------|------|-----------|--------|
| analyzer | ChainOfThought | query, context → analysis | max_tokens: 512 |
| generator | Predict | analysis → result | temperature: 0.7 |

---

## 3. Pipeline Composition

```python
class ComponentPipeline(Pipeline):
    def __init__(self):
        self.component = ComponentModule()
        self.validator = ValidateOutput()
        self.fallback = FallbackHandler()
    
    def forward(self, input_data: ComponentInput) -> ComponentOutput:
        try:
            result = self.component.forward(input_data.query, input_data.context)
            validated = self.validator.validate(result)
            return validated
        except Exception as e:
            return self.fallback.handle(e)
```

---

## 4. Optimizer Configuration

### Compilation Target

```python
optimizer = DSPyOptimizer(
    metric=accuracy_metric,
    max_iterations=20,
    strategy="bootstrap",  # or "random", "fewshot"
    trainset=training_examples
)

compiled_pipeline = optimizer.compile(
    program=self.pipeline,
    trainset=training_examples
)
```

### Metrics

| Metric | Definition | Threshold |
|--------|------------|------------|
| accuracy | Exact match on test set | >= 0.85 |
| latency | Response time (ms) | < 2000 |
| confidence_calibration | P(confidence == correct) | >= 0.90 |

---

## 5. Testing Strategy

### Unit Tests (Component Level)

```python
def test_component_signature():
    """Test that signature enforces contract."""
    input_data = ComponentInput(query="test", context=None)
    assert input_data.query is not None
    
def test_module_isolated():
    """Test module in isolation with mocked LM."""
    with mock_lm(output="expected") as lm:
        module = ComponentModule()
        result = module.forward("test query")
        assert result.result == "expected"
```

### Contract Tests

```python
def test_contract_compatibility():
    """Verify contract matches consumer expectations."""
    contract = load_contract("component-v1")
    assert contract.input_fields == ComponentInput.__annotations__
    assert contract.output_fields == ComponentOutput.__annotations__
```

### Integration Tests

```python
def test_pipeline_end_to_end():
    """Test full pipeline with real LM."""
    pipeline = ComponentPipeline()
    result = pipeline.forward(ComponentInput(query="test"))
    assert result.confidence >= 0.7
```

---

## 6. Observability

### Required Logging

```python
# Every call must log:
- trace_id: str          # Unique identifier for tracing
- input_signature: dict # Masked input (no PII)
- output_signature: dict # Output result
- latency_ms: int       # End-to-end latency
- token_usage: dict     # Input/output tokens
- confidence: float     # Model confidence score
```

### Metrics

| Metric | Type | Alert Threshold |
|--------|------|-----------------|
| error_rate | Counter | > 5% |
| latency_p99 | Histogram | > 5000ms |
| confidence_avg | Gauge | < 0.6 |
| contract_breaks | Counter | > 0 |

---

## 7. Safety & Guardrails

### Input Validation

```python
class InputGuardrail:
    def validate(self, input_data: ComponentInput) -> ValidationResult:
        # Check: length limits, PII detection, disallowed content
        # Return: (is_valid, reason_if_invalid)
```

### Output Filtering

```python
class OutputGuardrail:
    def filter(self, output: ComponentOutput) -> ComponentOutput:
        # Check: confidence threshold, content safety, format
        # Return: sanitized output or raise exception
```

### Chain-of-Thought Isolation

```reasoning: optional<string>  # Internal only - not exposed to user
result: string                # User-facing output only
```

---

## 8. Versioning & Compatibility

### Contract Versioning

| Version | Status | Changes |
|---------|--------|---------|
| 1.0.0 | active | Initial release |
| 1.1.0 | draft | Added confidence field |

### Compatibility Promise

- **Major version** (x.0.0): Breaking changes to signatures
- **Minor version** (1.x.0): Backward-compatible additions
- **Patch** (1.0.x): Bug fixes, no contract changes

---

## 9. Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| dspy | >= 2.5.0 | Core framework |
| langgraph | >= 0.2.0 | Durable execution |
| langsmith | latest | Observability |

---

## 10. Example Usage

```python
# Initialize component
component = ComponentPipeline()

# Execute with full observability
with trace("component-name") as span:
    input_data = ComponentInput(
        query="What is the status?",
        context={"user_id": "user_123"}
    )
    result = component.forward(input_data)
    span.set_output(result.dict())

# Output structure
{
    "result": "The system is operational.",
    "confidence": 0.92,
    "reasoning": "Checked system status via health endpoint..."
}
```

---

## Change Log

| Date | Version | Change | Author |
|------|---------|--------|--------|
| | | Initial template | |