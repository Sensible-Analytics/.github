# Smart GL - DSPy Component Architecture

## Project Overview

**Smart GL** is an AI-powered accounting application for Australian small businesses that automates transaction categorization.

**Core AI Flow:**
```
Basiq → Transactions → OpenAI Embeddings → Semantic Match
                                    ↓
                              Claude Sonnet → Category + GST
                                    ↓
                              Supabase + Formance Ledger
```

---

## Component Breakdown

### Component 1: TransactionPreprocessor

**Purpose:** Clean and normalize raw transaction data before AI processing.

**Current Implementation:** `clean_description()` in `categorise.py`

#### Input Signature (DSPy Style)

```python
class TransactionPreprocessorInput(Signature):
    """Raw transaction data from Basiq."""
    raw_description: str = Field(description="Raw bank transaction description")
    amount_cents: int = Field(description="Transaction amount in cents")
    merchant_name: Optional[str] = Field(description="Merchant name if available")
    basiq_category: Optional[str] = Field(description="Basiq's category hint")
```

#### Output Signature

```python
class TransactionPreprocessorOutput(Signature):
    """Cleaned transaction ready for categorisation."""
    cleaned_description: str = Field(description="Normalized description")
    direction: str = Field(description="income or expense")
    amount_aud: float = Field(description="Amount in AUD")
    is_receipt: bool = Field(description="Whether this is a receipt (positive amount)")
```

#### Module Implementation

```python
class TransactionPreprocessor(Module):
    def __init__(self):
        self.cleaner = Predict(
            "raw_description, amount_cents -> cleaned_description, direction, amount_aud"
        )
    
    def forward(self, raw_description: str, amount_cents: int) -> TransactionPreprocessorOutput:
        # Current rule-based cleaning (could be enhanced with DSPy)
        cleaned = clean_description(raw_description)
        direction = "income" if amount_cents > 0 else "expense"
        return TransactionPreprocessorOutput(
            cleaned_description=cleaned,
            direction=direction,
            amount_aud=abs(amount_cents) / 100,
            is_receipt=amount_cents > 0
        )
```

#### Contract Tests

```python
def test_preprocessor_contract():
    """Verify preprocessor output matches downstream expectations."""
    input_sig = TransactionPreprocessorInput.__annotations__
    output_sig = TransactionPreprocessorOutput.__annotations__
    
    # Contract: downstream (Embedder) expects 'cleaned_description' field
    assert "cleaned_description" in output_sig
    # Contract: downstream (Categoriser) expects 'direction' field
    assert "direction" in output_sig
```

---

### Component 2: EmbeddingGenerator

**Purpose:** Convert cleaned descriptions to vector embeddings for semantic search.

**Current Implementation:** `get_embedding()` in `categorise.py`

#### Input Signature

```python
class EmbeddingInput(Signature):
    """Text to embed."""
    text: str = Field(description="Cleaned transaction description")
    tenant_id: str = Field(description="Tenant for isolation")
```

#### Output Signature

```python
class EmbeddingOutput(Signature):
    """Vector embedding."""
    embedding: list[float] = Field(description="1536-dim embedding vector")
    model: str = Field(description="Model used (text-embedding-3-small)")
    dimensions: int = Field(description="Embedding dimensions (1536)")
```

#### Module Implementation

```python
class EmbeddingGenerator(Module):
    def __init__(self, model: str = "text-embedding-3-small", dimensions: int = 1536):
        self.model = model
        self.dimensions = dimensions
    
    async def forward(self, text: str) -> EmbeddingOutput:
        resp = await openai_client.embeddings.create(
            model=self.model,
            input=text,
            dimensions=self.dimensions
        )
        return EmbeddingOutput(
            embedding=resp.data[0].embedding,
            model=self.model,
            dimensions=self.dimensions
        )
```

#### Contract Tests

```python
def test_embedding_contract():
    """Verify embedding output format."""
    output = EmbeddingOutput(
        embedding=[0.1] * 1536,
        model="text-embedding-3-small",
        dimensions=1536
    )
    # Contract: semantic_matcher expects 1536-dim vectors
    assert len(output.embedding) == 1536
    assert output.dimensions == 1536
```

---

### Component 3: SemanticMatcher

**Purpose:** Find similar transactions from historical data using vector similarity.

**Current Implementation:** `supabase.rpc("match_embeddings")` in `categorise.py`

#### Input Signature

```python
class SemanticMatcherInput(Signature):
    """Embedding + matching parameters."""
    embedding: list[float] = Field(description="Query embedding")
    tenant_id: str = Field(description="Tenant isolation")
    threshold: float = Field(default=0.88, description="Similarity threshold")
    match_count: int = Field(default=1, description="Number of matches")
```

#### Output Signature

```python
class SemanticMatcherOutput(Signature):
    """Matched historical transaction."""
    has_match: bool = Field(description="Whether a match was found")
    matched_account_id: Optional[str] = Field(description="Account ID from match")
    matched_gst_code: Optional[str] = Field(description="GST code from match")
    similarity: float = Field(description="Similarity score 0-1")
```

#### Module Implementation

```python
class SemanticMatcher(Module):
    def __init__(self, threshold: float = 0.88):
        self.threshold = threshold
    
    def forward(
        self, 
        embedding: list[float], 
        tenant_id: str
    ) -> SemanticMatcherOutput:
        result = supabase.rpc(
            "match_embeddings",
            {
                "query_embedding": embedding,
                "tenant_id": tenant_id,
                "match_threshold": self.threshold,
                "match_count": 1,
            }
        ).execute()
        
        if result.data and len(result.data) > 0:
            top = result.data[0]
            return SemanticMatcherOutput(
                has_match=top["similarity"] >= self.threshold,
                matched_account_id=top["account_id"],
                matched_gst_code=top["gst_code"],
                similarity=float(top["similarity"])
            )
        
        return SemanticMatcherOutput(has_match=False)
```

#### Contract Tests

```python
def test_semantic_matcher_fallback_contract():
    """Verify contract when no match - triggers LLM fallback."""
    output = SemanticMatcherOutput(has_match=False)
    # Contract: downstream (LLMCategoriser) must handle has_match=False
    assert output.has_match == False
```

---

### Component 4: LLMCategoriser (Core AI Component)

**Purpose:** Categorize transactions using Claude when semantic match fails.

**Current Implementation:** `categorise_transaction()` in `categorise.py`

#### Input Signature

```python
class LLMCategoriserInput(Signature):
    """Transaction details + Chart of Accounts for LLM."""
    description: str = Field(description="Cleaned transaction description")
    amount_aud: float = Field(description="Amount in AUD")
    direction: str = Field(description="income or expense")
    merchant_name: Optional[str] = Field(description="Merchant name")
    basiq_category: Optional[str] = Field(description="Basiq category hint")
    chart_of_accounts: list[dict] = Field(description="COA with code, name, type, gst_code")
```

#### Output Signature

```python
class LLMCategoriserOutput(Signature):
    """Categorisation result from LLM."""
    account_code: Optional[str] = Field(description="Account code (e.g., '5000')")
    account_id: Optional[str] = Field(description="Account UUID from DB")
    gst_code: Optional[str] = Field(description="GST code (G1, G2, G3, G4, G9, G11, N-T)")
    confidence: float = Field(description="LLM confidence 0-1")
    requires_review: bool = Field(description="Whether human review needed")
    reasoning: str = Field(description="LLM's reasoning for choice")
```

#### Module Implementation (DSPy Style)

```python
class LLMCategoriser(Module):
    def __init__(self, model: str = "claude-sonnet-4-20250514"):
        self.model = model
        self.categorise = ChainOfThought(
            "description, amount, direction, merchant, basiq_category, coa -> "
            "account_code, gst_code, confidence, requires_review, reasoning"
        )
    
    def forward(self, input_data: LLMCategoriserInput) -> LLMCategoriserOutput:
        coa_text = "\n".join(
            f"{a['code']} | {a['name']} | {a['account_type']} | GST:{a['gst_code']}"
            for a in input_data.chart_of_accounts
        )
        
        prompt = f"""You are an Australian bookkeeper for a small plumbing and trades business.
Categorise the following bank transaction to the correct account in the Chart of Accounts.

Transaction details:
- Description: {input_data.description}
- Amount: ${input_data.amount_aud:.2f} AUD ({input_data.direction})
- Merchant: {input_data.merchant_name or "unknown"}
- Bank category: {input_data.basiq_category or "unknown"}

Chart of Accounts:
{coa_text}

Rules:
1. Return ONLY the account code number (e.g. "5000") and nothing else on the first line.
2. On the second line, return the GST code that applies (G1, G2, G3, G4, G9, G11, or N-T).
3. On the third line, return a confidence score between 0.00 and 1.00.
4. On the fourth line, give a one-sentence reason for your choice.

If you cannot determine with confidence above 0.70, return "REVIEW" on the first line."""
        
        response = anthropic_client.messages.create(
            model=self.model,
            max_tokens=150,
            messages=[{"role": "user", "content": prompt}]
        )
        
        # Parse response (current implementation)
        # DSPy would handle parsing automatically via signature
        return self._parse_response(response.content[0].text)
```

#### Optimizer Configuration

```python
# DSPy Optimizer would tune:
# - Prompt templates
# - Few-shot examples
# - Temperature / max_tokens

categoriser_optimizer = DSPyOptimizer(
    metric=categorisation_accuracy,  # Compare against human labels
    max_iterations=50,
    strategy="bootstrap",
    trainset=historical_categorisations
)
```

#### Contract Tests

```python
def test_llm_categoriser_output_contract():
    """Verify LLM output matches downstream requirements."""
    output = LLMCategoriserOutput(
        account_code="5000",
        account_id="uuid-here",
        gst_code="G1",
        confidence=0.85,
        requires_review=False,
        reasoning="Office supplies typically go to 5000"
    )
    
    # Contract: downstream (LedgerPoster) needs account_id
    assert output.account_id is not None
    # Contract: downstream needs valid GST code
    assert output.gst_code in ["G1", "G2", "G3", "G4", "G9", "G11", "N-T"]
    # Contract: confidence must be 0-1
    assert 0 <= output.confidence <= 1
```

---

### Component 5: CategorisationPipeline (Composite)

**Purpose:** Orchestrates the full categorization flow.

#### Pipeline Composition

```python
class CategorisationPipeline(Module):
    def __init__(self):
        self.preprocessor = TransactionPreprocessor()
        self.embedder = EmbeddingGenerator()
        self.matcher = SemanticMatcher(threshold=0.88)
        self.llm_categoriser = LLMCategoriser()
        self.validator = OutputValidator()
    
    async def forward(
        self,
        raw_description: str,
        amount_cents: int,
        merchant_name: Optional[str],
        basiq_category: Optional[str],
        chart_of_accounts: list[dict],
        tenant_id: str
    ) -> CategorisationOutput:
        # Step 1: Preprocess
        preprocessed = self.preprocessor.forward(raw_description, amount_cents)
        
        # Step 2: Semantic match
        embedding = await self.embedder.forward(preprocessed.cleaned_description)
        match = self.matcher.forward(embedding.embedding, tenant_id)
        
        if match.has_match:
            return CategorisationOutput(
                account_id=match.matched_account_id,
                gst_code=match.matched_gst_code,
                confidence=match.similarity,
                tier="embedding",
                reasoning=None
            )
        
        # Step 3: LLM fallback
        llm_input = LLMCategoriserInput(
            description=preprocessed.cleaned_description,
            amount_aud=preprocessed.amount_aud,
            direction=preprocessed.direction,
            merchant_name=merchant_name,
            basiq_category=basiq_category,
            chart_of_accounts=chart_of_accounts
        )
        llm_output = self.llm_categoriser.forward(llm_input)
        
        if llm_output.requires_review or llm_output.confidence < 0.70:
            return CategorisationOutput(
                account_id=None,
                gst_code=None,
                confidence=llm_output.confidence,
                tier="human",
                reasoning=llm_output.reasoning
            )
        
        return CategorisationOutput(
            account_id=llm_output.account_id,
            gst_code=llm_output.gst_code,
            confidence=llm_output.confidence,
            tier="llm",
            reasoning=llm_output.reasoning
        )
```

---

## Contract Enforcement Strategy

### 1. Static Contract Tests (Pact-style)

```python
# tests/contracts/test_categorisation_pipeline.py
import pytest
from pact import Consumer, Provider, Term

def test_preprocessor_to_embedder_contract():
    """Preprocessor output must match embedder input."""
    consumer = Consumer("CategorisationPipeline")
    provider = Provider("EmbeddingGenerator")
    
    pact = consumer.has_pact_with(provider)
    pact.given("embedder accepts text input")
    pact.when(
        method="POST",
        path="/embed",
        body={
            "text": Term(generate="Coffee at cafe", matcher=r".+"),
            "tenant_id": Term(generate="tenant-123", matcher=r".+")
        }
    )
    pact.then(
        status=200,
        body={
            "embedding": Term(generate=[0.1] * 1536, matcher=r"\[.+\]"),
            "model": "text-embedding-3-small",
            "dimensions": 1536
        }
    )
```

### 2. DSPy Signature Validation

```python
# Validate all components adhere to signatures
def validate_component_contracts():
    # Input/Output signatures must be type-annotated
    assert hasattr(LLMCategoriserInput, '__annotations__')
    assert hasattr(LLMCategoriserOutput, '__annotations__')
    
    # Required fields must be present
    required_input = ['description', 'amount_aud', 'direction', 'chart_of_accounts']
    required_output = ['account_code', 'gst_code', 'confidence', 'requires_review']
    
    for field in required_input:
        assert field in LLMCategoriserInput.__annotations__
    for field in required_output:
        assert field in LLMCategoriserOutput.__annotations__
```

### 3. Integration Contract Tests

```python
# tests/contracts/test_pipeline_integration.py
@pytest.mark.contract
async def test_full_pipeline_contract():
    """End-to-end contract test with mocked external services."""
    pipeline = CategorisationPipeline()
    
    # Mock embedder to avoid external calls
    with mock_embedder(embedding=[0.1]*1536):
        result = await pipeline.forward(
            raw_description="COFFEE CAFÉ MERCHANDISE",
            amount_cents=-650,
            merchant_name="Cafe",
            basiq_category="food",
            chart_of_accounts=coa_fixture,
            tenant_id="tenant-123"
        )
    
    # Contract assertions
    assert isinstance(result.account_id, (str, type(None)))
    assert result.gst_code in ["G1", "G2", "G3", "G4", "G9", "G11", "N-T", None]
    assert 0 <= result.confidence <= 1
    assert result.tier in ["embedding", "llm", "human"]
```

---

## Observability Requirements

| Metric | Type | Alert Threshold |
|--------|------|-----------------|
| categorisation_accuracy | Gauge | < 85% |
| embedding_latency_ms | Histogram | > 500ms |
| llm_latency_ms | Histogram | > 3000ms |
| semantic_match_rate | Counter | Track % using embeddings vs LLM |
| human_review_rate | Counter | > 20% |
| confidence_avg | Gauge | < 0.75 |

---

## Migration Path (Current → DSPy)

| Phase | Action | Benefit |
|-------|--------|---------|
| 1 | Add signature types to existing components | Type safety, explicit contracts |
| 2 | Add contract tests | Verify interfaces don't break |
| 3 | Replace hardcoded prompts with DSPy modules | Reusable, testable |
| 4 | Add optimizer | Self-improving categorisation |
| 5 | Add observability | Full trace visibility |

---

## Files Created

- `.firecrawl/dspy-component-template.md` - Generic template
- `.firecrawl/smart-gl-dspy-breakdown.md` - This file