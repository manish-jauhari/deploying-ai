# Assignment Chat (`05_src/assignment_chat`)

This project implements Assignment 2 as a chat-based AI assistant with **three services**, memory, guardrails, and a Gradio interface.

## Submission Strategy (Consolidated)

- **Primary notebook for submission/demo:** `06_end_to_end_demo.ipynb`
- Supporting notebooks (`01` to `05`) are retained as implementation evidence and incremental build artifacts.
- The consolidated notebook includes all required logic (Services A/B/C, routing, memory, guardrails, and end-to-end trace).

## Chat Client Overview

Assistant name/persona used in integration notebooks:
- **Atlas**: calm, concise, practical academic assistant.

Core behavior:
- Routes user messages to one of three services.
- Maintains short-term memory across turns.
- Applies guardrails before service execution.

---

## Services Implemented

## Service A — API Calls
Notebook: `01_service_api.ipynb`

- Backend: Open-Meteo public APIs (geocoding + current weather).
- Input: city/weather-style query.
- Output: transformed natural-language weather summary (not raw JSON).
- Includes input validation and safe error handling.

## Service B — Semantic Query
Notebook: `02_service_semantic_search.ipynb`

- Backend: local semantic retrieval with Chroma persistent storage.
- Data source: local repo files (small corpus under assignment constraints).
- Retrieval flow: chunk -> embed -> persist -> query top-k -> grounded answer.
- Supports local-first execution and optional OpenAI path flags where configured.

### Embedding Process (as requested by assignment)
- Chunking strategy: character-based chunks with overlap.
- Default local embedding model: `sentence-transformers/all-MiniLM-L6-v2`.
- Vectors stored in Chroma persistent collections in `05_src/assignment_chat/chroma_store/`.
- Query flow uses the same embedding backend as indexing for dimension consistency.

## Service C — Function Calling
Notebook: `03_service_function_calling.ipynb`

- Local function-calling implementation using `@tool` decorators.
- Tools:
  - `tool_calculate`
  - `tool_current_datetime`
  - `tool_word_count`
- Deterministic local router invokes tools via `.invoke(...)`.

---

## Guardrails

Implemented and validated in:
- `05_guardrails_and_eval.ipynb`
- used in `04_chat_integration_gradio.ipynb` and `06_end_to_end_demo.ipynb`

Guardrails enforced:
1. Prevent prompt reveal / prompt modification attempts.
2. Refuse restricted topics:
   - cats or dogs
   - horoscopes or zodiac
   - Taylor Swift

Evaluation notebook includes blocked/allowed test suites and pass/fail reporting.

---

## Chat Integration (Unified System)

Notebook: `04_chat_integration_gradio.ipynb`

Integrated features:
- Unified handler for Services A/B/C.
- Intent router (weather -> A, retrieval -> B, tools/math/date/count -> C).
- Memory list with trim policy (`MAX_MEMORY_TURNS`).
- Guardrails before service dispatch.
- Gradio `ChatInterface` scaffold ready to launch.

---

## Local UI (Current Demo UI)

Notebook: `06_end_to_end_demo.ipynb`

UI behavior implemented in Section 7:
- Gradio local chat app on `http://127.0.0.1:1234`.
- Uses the same unified `handle_turn` pipeline as the demo logic.
- Input submit supports both:
  - pressing Enter in the textbox
  - clicking the **Send** button
- Includes a helper stop cell to close the local server before relaunching.

Recommended UI run sequence in `06_end_to_end_demo.ipynb`:
1. Run setup and service cells (Sections 1 to 6).
2. Run Section 7a to launch the UI.
3. If rerunning, execute Section 7b first to stop the prior server instance, then run Section 7a again.

---

## End-to-End Demo

Notebook: `06_end_to_end_demo.ipynb`

Demonstrates in one run:
- Service A success case
- Service B success + follow-up memory usage
- Service C tool invocation success
- Restricted-topic refusal
- Prompt-protection refusal
- Local interactive Gradio chat UI for manual testing on port `1234`

---

## Run Order (Recommended)

### Fast path (clean submission demo)
1. `06_end_to_end_demo.ipynb`

### Full evidence path (implementation walkthrough)
1. `01_service_api.ipynb`
2. `02_service_semantic_search.ipynb`
3. `03_service_function_calling.ipynb`
4. `04_chat_integration_gradio.ipynb`
5. `05_guardrails_and_eval.ipynb`
6. `06_end_to_end_demo.ipynb`

---

## Environment Notes

- Use the course virtual environment: `deploying-ai-env`.
- All notebooks were implemented to run with standard course setup.
- Local-first design minimizes dependency on external model calls.

---

## Design Decisions (Short)

- Kept implementation simple and modular to align with assignment scope.
- Used deterministic local logic where possible for reproducibility.
- Added robust fallbacks and clear error messages for smoother grading/demo.
