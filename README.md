# Tiny SQL Expert — NL → SQL using a Local Small LLM (<4B)

**Author:** Aslam Suneer  
**Task chosen:** The “Tiny” SQL Expert (SLM Optimization) — convert natural-language questions into SQL using a small local LLM, with schema-aware prompting and a validation + self-correction loop.

---

## Summary
This project converts English questions into valid SQL using a local small language model (TinyLlama 1.1B in mock/real demos).
The system injects the database schema into the prompt, uses few-shot examples, and applies a validation-and-repair mechanism:

The LLM generates SQL.

SQLite validates it using EXPLAIN.

If invalid, the system re-prompts the model with the actual SQLite error to correct the query.

All attempts are stored in logs/pipeline.log as JSONL (failure → retry → success).

**Key features**
- SLM (<4B) inference (TinyLlama 1.1B recommended)
- Schema injection (users, products, orders)
- Forbidden keyword filter (DROP / DELETE / ALTER / TRUNCATE)
- SQLite EXPLAIN-based validation
- Self-correction loop (retries with error-fix prompts)
- Unit tests (pytest) for pipeline components
- JSONL logging for traceability
- Mock mode (no model required)
---

## Repo contents
- `tiny_sql_expert.py` — main NL → SQL pipeline
- `schema.sql` — sample DDL used in prompts
- `seed_and_run.py` — seeds data & runs mock tests
- `metrics.py` — batch runner for success statistics
- `tests/` — unit tests for cleaner/validator/pipeline
- `requirements.txt` — all dependencies
- `.pre-commit-config.yaml` — lint & format hooks
- `demo/` — screenshots of real runs
- `logs/` — auto-generated JSONL logs

---

## Quick start (Windows, PowerShell) — minimal (mock)
1. Create & activate a virtual environment
```powershell
cd C:\Users\aslam\Project
python -m venv venv
venv\Scripts\activate
```
2. Install dependencies
pip install -r requirements.txt
```powershell
3. Run in mock mode (no model required)
python tiny_sql_expert.py "List users who bought electronics" --raw --mock
```
Example output:

Attempt 1 → invalid
Attempt 2 → valid
SELECT u.name, u.email FROM users ...

4. Run with a real local model (<4B)
Requires a GGUF file such as:
```powershell
models/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf
```
Then run:
```powershell
python tiny_sql_expert.py "Show top 5 products by revenue" --raw --model models/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf
```
5. Run tests
```powershell
pytest
```

# Why These Libraries & Models?
- TinyLlama 1.1B (GGUF):
    Fits the <4B requirement
    CPU-friendly & fully offline
    GGUF optimized for llama-cpp
    Sufficient reasoning for SQL generation

- llama-cpp-python:
    Fast CPU inference
    Works with GGUF
    Minimal setup, no GPU needed

- SQLite
    Safe validation using EXPLAIN
    No risk of executing harmful SQL
    Lightweight and portable

- sqlparse
    Cleans LLM output
    Removes code fences, normalizes text

- pytest
    Ensures correctness of validation, cleaning, and repair logic

# Demo Evidence
    A folder /demo/ includes screenshots showing:
    PowerShell running mock mode
    Real model loading (metadata + KV cache logs)
    Validation error + retry loop
    Corrected SQL outputs
    Directory tree and final result files
