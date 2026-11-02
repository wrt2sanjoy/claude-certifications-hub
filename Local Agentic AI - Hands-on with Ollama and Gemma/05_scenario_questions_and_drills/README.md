# 05. Scenario Practice Questions & Drills: Local AI & Agents

Test your understanding of local models, Ollama, Python integration, and Agent architectures. Each question includes a **detailed correct answer analysis** and an **explanation of why each distractor is wrong**.

---

## 📝 Drill 1: Hardware Sizing for Local Models
**Scenario:** A developer wants to run a local LLM on a corporate Windows laptop equipped with 8 GB of total system RAM and an integrated Intel GPU. They want to experiment with agent tool calling and basic text summarization.

Which model should they download via Ollama for smooth, responsive performance without crashing their machine?
- **A)** `llama3:70b` (quantized 4-bit)
- **B)** `gemma2:2b` or `qwen2.5:3b`
- **C)** `claude-3-opus` local checkpoint
- **D)** An unquantized 32-bit FP32 version of Gemma 9B

<details>
<summary>👉 Click to Reveal Answer & Deep Analysis</summary>

### Correct Answer: **B**
- **Correct Answer Rationale:** On an 8 GB RAM laptop, the operating system alone consumes 3–4 GB of memory. A lightweight quantized model like `gemma2:2b` or `qwen2.5:3b` requires only ~1.6 GB to 2.5 GB of RAM, leaving ample headroom for Python and system processes to run smoothly.
- **Distractor Rationale:**
  - **A is incorrect:** A 70-billion parameter model requires at least 40 GB+ of RAM even when quantized; running it on an 8 GB machine will cause an immediate Out-Of-Memory (OOM) crash.
  - **C is incorrect:** Anthropic's Claude models are proprietary closed-weights cloud models and cannot be downloaded or run locally via Ollama.
  - **D is incorrect:** An unquantized FP32 9B model requires over 36 GB of RAM.
</details>

---

## 📝 Drill 2: Ollama API Endpoint Selection
**Scenario:** You are building an interactive multi-turn customer support chatbot using Python and Ollama. On the third turn of the conversation, the user asks: *"Can you remind me what you just said in your previous reply?"* The model responds: *"I have no memory of previous messages."*

What architectural mistake was made in the Python script?
- **A)** The script failed to install an Nvidia graphics card driver.
- **B)** The script used `POST /api/generate` with a single prompt string instead of maintaining a `messages = []` history list sent to `POST /api/chat`.
- **C)** Ollama does not support conversation memory under any circumstances.
- **D)** The model was run at temperature 0.0, which erases RAM after every turn.

<details>
<summary>👉 Click to Reveal Answer & Deep Analysis</summary>

### Correct Answer: **B**
- **Correct Answer Rationale:** Ollama's local server is completely stateless. It does not store conversation sessions internally. To maintain conversation memory, your client-side Python program must accumulate user and assistant turns in a list (`messages = [...]`) and send the entire history to the `/api/chat` endpoint on every turn.
- **Distractor Rationale:**
  - **A is incorrect:** Hardware drivers have no impact on whether an API remembers conversation turns.
  - **C is incorrect:** Ollama fully supports multi-turn chats via its `/api/chat` interface when the client passes the message history.
  - **D is incorrect:** Temperature controls output token randomness; it has zero effect on session memory.
</details>

---

## 📝 Drill 3: Agent Execution Mechanics
**Scenario:** A junior developer is building their first AI agent and claims: *"When Gemma decides to query the local SQLite database, the neural network directly opens the `.db` file on my hard drive and executes SQL inside its neural synapses."*

How should you correct this misconception?
- **A)** Agree with them, because transformers natively run SQL code inside self-attention layers.
- **B)** Explain that the LLM only outputs structured text/JSON specifying the tool and query; the host Python runtime intercepts this text, executes the SQL command, and feeds the results back to the LLM.
- **C)** Explain that LLMs can only execute SQL if they are converted into C++ binaries.
- **D)** Explain that SQLite only works with cloud AI, not local models.

<details>
<summary>👉 Click to Reveal Answer & Deep Analysis</summary>

### Correct Answer: **B**
- **Correct Answer Rationale:** LLMs are purely predictive text engines. They cannot open files, make database connections, or execute binary code. In an agent architecture, the LLM emits a tool call decision (like `{"tool": "query_db", "sql": "SELECT * FROM sales"}`). The surrounding Python application inspects that JSON, runs `sqlite3.connect()`, fetches the rows, and passes them back to the LLM as an observation.
- **Distractor Rationale:**
  - **A is incorrect:** Neural networks do not contain operating system file handlers or SQL execution engines.
  - **C is incorrect:** Python natively handles SQL execution without converting LLMs to C++.
  - **D is incorrect:** SQLite is a local file-based database that works locally.
</details>

---

## 📝 Drill 4: Agent Security & Tool Execution
**Scenario:** You are reviewing code written for an autonomous agent that calculates mathematical formulas requested by users. You discover this snippet:
```python
tool_call = gemma_output["tool_name"]  # e.g., "import os; os.system('format C:')"
eval(tool_call)
```
What is the major security vulnerability here, and how should it be remediated?
- **A)** `eval()` is too slow; it should be replaced with `exec()`.
- **B)** Arbitrary code execution vulnerability: Malicious prompts could wipe files. It should be replaced with a hardcoded dictionary mapping allowed tool names to safe Python functions.
- **C)** Gemma does not know Python syntax, so `eval()` will always return None.
- **D)** `eval()` requires an active internet connection to authenticate with Google.

<details>
<summary>👉 Click to Reveal Answer & Deep Analysis</summary>

### Correct Answer: **B**
- **Correct Answer Rationale:** Passing unvalidated LLM output directly into Python's `eval()` allows an attacker or a hallucinating model to execute arbitrary shell commands with the privileges of the Python process. The standard defense is a strict whitelist dictionary (`SAFE_TOOLS = {"calculate": safe_math_func}`) that rejects any unauthorized function names.
- **Distractor Rationale:**
  - **A is incorrect:** `exec()` is equally dangerous and introduces the exact same arbitrary code execution flaw.
  - **C is incorrect:** LLMs frequently output valid Python code, making `eval()` actively dangerous.
  - **D is incorrect:** `eval()` is a local built-in Python function that requires no network access.
</details>

---

## 📝 Drill 5: Preventing Infinite Loops
**Scenario:** An agent is tasked with scraping data from a URL. However, the target website is down and returns an HTTP 500 error. The agent's reasoning loop says: *"The site failed. I will try again."* It re-runs the tool indefinitely, consuming 100% of the laptop's CPU.

What design pattern resolves this issue?
- **A)** Increase the temperature to 1.0 so the model feels happier.
- **B)** Enforce a Circuit Breaker / `MAX_ITERATIONS` limit (e.g., maximum 5 iterations) that terminates the loop and alerts the user if the task cannot be completed.
- **C)** Delete the Python virtual environment.
- **D)** Run the agent only during nighttime hours.

<details>
<summary>👉 Click to Reveal Answer & Deep Analysis</summary>

### Correct Answer: **B**
- **Correct Answer Rationale:** A circuit breaker or maximum loop iteration threshold (`MAX_STEPS = 5`) is mandatory for all production agent loops. When an unexpected persistent failure occurs, the loop counter prevents runaway CPU consumption or endless retries, gracefully terminating the execution.
- **Distractor Rationale:**
  - **A is incorrect:** Temperature does not fix network errors and can introduce more unpredictable behavior.
  - **C is incorrect:** Deleting the environment deletes your libraries without fixing the algorithmic flaw.
  - **D is incorrect:** Irrelevant to software architecture.
</details>

---

## 🏁 Summary Checkpoint
- **5 / 5 Correct**: Exceptional! You clearly understand how local LLMs, Python APIs, and agent runtimes interact safely.
- **3–4 Correct**: Great job! Review the security patterns in [04_observability_security_and_ops](../04_observability_security_and_ops/README.md) and tool dispatching in [02_architecture_and_deep_dives](../02_architecture_and_deep_dives/README.md).
- **Under 3 Correct**: Run through the small code scripts in [03_hands_on_labs_and_code](../03_hands_on_labs_and_code/README.md) to build practical muscle memory!

