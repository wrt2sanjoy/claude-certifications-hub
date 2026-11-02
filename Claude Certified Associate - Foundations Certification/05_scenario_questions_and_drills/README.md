# 05. High-Yield Scenario Questions & Exam Drills

This module contains realistic, multi-paragraph scenario questions mirroring the official **Claude Certified Associate - Foundations (CCAO-F)** Pearson VUE examination.

Each question includes a deep **Correct Answer Rationale** and a detailed **Distractor Analysis** for every incorrect option to train you to spot test traps instantly.

---

## 🎯 Question 1: Product & Model Selection (Domain 3)

**Scenario**:
A financial services firm processes 250,000 incoming customer inquiries per day. The incoming requests require intent classification (e.g., "Account Balance", "Loan Inquiry", "Fraud Report") and routing to appropriate human teams. The system requires an end-to-end latency of under 800 milliseconds and has a strict monthly infrastructure budget.

Which Claude model and configuration should the Lead Architect recommend?

- **A)** Claude 3 Opus with a 200k token system prompt detailing all historical loan records.
- **B)** Claude 3.5 Haiku via the Messages API with a concise system prompt and structured XML intent categories.
- **C)** Claude Projects with uploaded PDF policy manuals accessed by agents via web browser.
- **D)** Claude 3.5 Sonnet with temperature set to 1.0 to maximize classification diversity.

---

### 💡 Answer & In-Depth Rationale

**Correct Answer**: **B**

#### Why B is Correct:
- **Claude 3.5 Haiku** provides sub-second latency and an ultra-low cost profile ($0.80 / million input tokens), perfectly meeting the 250,000/day volume constraint.
- Routing via the **Messages API** enables automated headless microservice orchestration.
- Structured XML tags (`<categories>`, `<rules>`) ensure deterministic classification.

#### Distractor Analysis:
- **Why A is Incorrect**: **Claude 3 Opus** is designed for deep academic and complex multi-step reasoning. At 250,000 queries/day, it would be prohibitively expensive and cannot meet the sub-800ms latency requirement. Furthermore, feeding historical loan records into the system prompt for basic routing wastes tokens.
- **Why C is Incorrect**: **Claude Projects** is a collaborative web interface on `claude.ai` designed for human teams, not a headless, automated real-time routing microservice handling 250,000 daily API calls.
- **Why D is Incorrect**: While **Claude 3.5 Sonnet** is powerful, setting `temperature=1.0` introduces randomness and hallucinations into classification tasks where deterministic categorization (`temperature=0.0`) is required. Sonnet is also more expensive and slower than Haiku for basic triage.

---

## 🎯 Question 2: Prompting & Output Determinism (Domain 1)

**Scenario**:
A developer is building an automated billing microservice that extracts invoice items from vendor emails and converts them into JSON for an ERP database. During testing, the developer observes that Claude occasionally includes introductory conversational text such as *"Certainly! Here is your extracted JSON invoice:"* which causes the application's `json.loads()` parser to crash.

Which technique is the Anthropic-recommended best practice to solve this issue?

- **A)** Add *"Please do not say hello and do not output any markdown"* repeatedly in the user message.
- **B)** Switch to Claude 3 Opus because larger models never output conversational preambles.
- **C)** Prefill the assistant's turn in the Messages API with `{` so Claude continues outputting strictly raw JSON.
- **D)** Increase the `temperature` parameter to 0.9 so Claude avoids predictable phrases.

---

### 💡 Answer & In-Depth Rationale

**Correct Answer**: **C**

#### Why C is Correct:
- **Prefilling the Assistant's message** with `{` directly constrains Claude's generation from token 0. Because Claude's first token is forced to be the opening brace, it is impossible for the model to emit a greeting or preamble.

#### Distractor Analysis:
- **Why A is Incorrect**: Negative prompting (*"Do not say hello"*) is fragile and often fails under edge cases or varying context loads.
- **Why B is Incorrect**: Model size does not prevent polite conversational preambles; in fact, standard conversational alignment across all models defaults to helpful introductory phrases unless explicitly constrained or prefilled.
- **Why D is Incorrect**: Increasing temperature to 0.9 increases output entropy and randomness, making schema violations and erratic conversational text much more likely.

---

## 🎯 Question 3: Enterprise Prompt Caching Mechanics (Domain 5)

**Scenario**:
A legal tech startup analyzes 100-page contracts (~45,000 tokens) against 50 different compliance checks per contract. The engineering team enables Anthropic Prompt Caching on Claude 3.5 Sonnet. In their implementation, they structure the API request as follows:

```json
{
  "model": "claude-3-5-sonnet-20241022",
  "system": "You are a legal auditor.",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "Specific Question: Does clause 12 violate GDPR?"},
        {
          "type": "text",
          "text": "<contract>[45,000 tokens of contract text]</contract>",
          "cache_control": {"type": "ephemeral"}
        }
      ]
    }
  ]
}
```

The team notices their caching hit rate is 0% across consecutive calls. Why is the cache failing to hit?

- **A)** The contract length (45,000 tokens) exceeds the maximum allowed cache size.
- **B)** Prompt caching is only supported on Claude 3.5 Haiku, not Claude 3.5 Sonnet.
- **C)** The dynamic user question was placed before the static contract, breaking the required byte-for-byte prefix match.
- **D)** Prompt caching requires at least 100,000 tokens before activation.

---

### 💡 Answer & In-Depth Rationale

**Correct Answer**: **C**

#### Why C is Correct:
- Anthropic Prompt Caching uses **prefix matching**. Claude caches the key-value activations from token 0 up to the cache breakpoint. Because the dynamic question changed on every call and was placed *before* the contract, the prefix at token 0 differed every time, causing 100% cache misses. The static contract must be placed first!

#### Distractor Analysis:
- **Why A is Incorrect**: Prompt caching easily supports documents up to the full 200k context window.
- **Why B is Incorrect**: Prompt caching is fully supported on Claude 3.5 Sonnet, Claude 3.5 Haiku, and Claude 3 Opus.
- **Why D is Incorrect**: The minimum threshold for Claude 3.5 Sonnet is **1,024 tokens**, not 100,000 tokens.

---

## 🎯 Question 4: Governance & Data Privacy (Domain 6)

**Scenario**:
An enterprise healthcare client wishes to deploy Claude to summarize clinical trial notes containing proprietary drug formulation data. The client's Chief Information Security Officer (CISO) refuses approval, expressing concern that Anthropic will use their proprietary pharmaceutical formulas to retrain Claude's future commercial models.

How should the Solutions Architect address the CISO's concern based on Anthropic's official enterprise policies?

- **A)** Inform the CISO that Anthropic retrains models only on anonymous summary embeddings.
- **B)** Clarify that Anthropic does NOT train its foundation models on customer prompts or completions submitted via the commercial Messages API.
- **C)** Suggest using Claude Web Free Tier because free accounts have stronger privacy controls than the API.
- **D)** Promise that setting `temperature=0.0` disables model training logging.

---

### 💡 Answer & In-Depth Rationale

**Correct Answer**: **B**

#### Why B is Correct:
- Anthropic has an explicit contractual and institutional guarantee: **customer data submitted through the commercial Messages API is NEVER used to train foundation models**.

#### Distractor Analysis:
- **Why A is Incorrect**: Anthropic does not train on commercial customer API data at all—not even on "anonymous embeddings".
- **Why C is Incorrect**: The consumer free web tier has standard consumer terms and may use conversational interactions to improve services unless opted out; the commercial API has the strongest enterprise privacy guarantees.
- **Why D is Incorrect**: `temperature` is an inference sampling parameter controlling randomness; it has zero relationship with server-side data retention or training policies.

---

## 📊 Self-Assessment Diagnostic Scorecard

Use this scorecard after completing the drills:

| Score | Rating | Recommended Action |
| :---: | :--- | :--- |
| **4 / 4 (100%)** | 🏆 Master Level (Distinction Track) | Move directly to full timed mock exams in Phase 4. |
| **3 / 4 (75%)** | 🥈 Passing Associate Level | Review the specific missed domain and reread the corresponding architecture guide. |
| **< 3 (50% or below)** | ⚠️ Foundation Gap | Revisit `01_core_concepts_and_fundamentals` and `02_architecture_and_deep_dives`. |
