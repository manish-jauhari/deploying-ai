# Assignment 2 Execution Plan (`assignment_chat`)

## 1) Assignment Goal (Interpreted)
Build a chat-based AI system in `./05_src/assignment_chat` with:
1. **At least 3 services**:
   - Service A: API-backed service with transformed (not verbatim) output.
   - Service B: Semantic (or hybrid) search service.
   - Service C: Open-ended service using one of: Function Calling, Web Search, or MCP.
2. **Chat UI** (preferably Gradio) with:
   - Distinct assistant personality.
   - Conversation memory across turns.
3. **Guardrails**:
   - Prevent revealing/changing system prompt.
   - Refuse restricted topics: cats/dogs, horoscopes/zodiac, Taylor Swift.
4. **Implementation constraints**:
   - Use standard course environment only.
   - Place implementation in `05_src/assignment_chat`.
   - Include `readme.md` documenting system and design decisions.

---

## 2) Requirements Breakdown and Definitions

### 2.1 Service A (API Calls)
**Definition:** A user intent routed to an external API backend.  
**Requirement detail:** Output cannot be raw API response; must be transformed.

**Planned implementation:**
- Choose one free API (example: exchange rates, weather, trivia, world time, etc.).
- Wrap API call in a service function that:
  1) validates user input,
  2) calls API,
  3) maps response to concise natural-language answer.

**Concepts used:**
- HTTP request/response lifecycle
- Structured data transformation
- Prompt-based rewriting/summarization (optional)

---

### 2.2 Service B (Semantic Query)
**Definition:** User asks natural-language question; system retrieves relevant chunks by meaning.  
**Requirement detail:** Must be semantic search (or hybrid lexical + semantic).

**Planned implementation:**
- Use a small local document set (`<= 40 MB`) inside repo.
- Build/persist a local **ChromaDB PersistentClient** vector store.
- Retrieval pipeline:
  1) query embedding,
  2) top-k retrieval,
  3) context assembly,
  4) grounded answer generation.

**Concepts used:**
- Embeddings (dense vectors)
- Vector similarity (cosine/dot-product depending on backend)
- Top-k retrieval and grounding
- Hallucination reduction by source-constrained generation

---

### 2.3 Service C (Open choice using required tooling)
**Definition:** Third service with custom behavior, but must use Function Calling, Web Search, or MCP.

**Planned implementation choice:**
- Implement **Function Calling** orchestration service (lowest dependency risk in course environment).
- Provide 1–2 internal tools (e.g., calculator, date utility, structured formatter) and route intent through tool calls.

**Concepts used:**
- Tool schema (name, description, params)
- Tool execution loop
- Observation-to-answer synthesis

---

### 2.4 Chat UI + Personality + Memory
**Definition:** A single conversational interface that routes requests to the right service while preserving context.

**Planned implementation:**
- Gradio chat app with a clear persona (e.g., calm academic TA style).
- In-memory chat history (`list[dict]` or equivalent role-message format).
- Routing layer decides service by intent classification rules (lightweight keyword + fallback LLM routing).

**Concepts used:**
- Session state
- Turn-level memory
- Intent routing and fallback logic

---

### 2.5 Guardrails
**Definition:** Rules that block sensitive prompt-injection and prohibited topics.

**Planned implementation:**
1. **Pre-input guardrail** (user message filter):
   - Block restricted topics.
   - Block attempts to reveal/override system prompt.
2. **System instruction hardening**:
   - Explicit non-disclosure and non-modification policy.
3. **Post-output guardrail** (optional lightweight check):
   - Ensure final response does not contain prompt internals.

**Concepts used:**
- Prompt-injection defense in depth
- Policy refusal templates
- Deterministic pattern filters + model-side policy instruction

---

## 3) End-to-End Data Flow

### 3.1 High-level flow
1. User sends message in Gradio chat.
2. Input guardrails run:
   - If restricted/prompt-injection intent detected -> refusal response.
3. Router chooses service (API / semantic / function-calling).
4. Chosen service executes:
   - API service: fetch + transform.
   - Semantic service: retrieve + grounded answer.
   - Function service: tool call loop + synthesis.
5. Output post-check (optional).
6. Assistant response appended to memory and returned to UI.

### 3.2 Internal data objects
- `ChatTurn`: role + content + timestamp (optional)
- `RouterDecision`: `service_name`, confidence/reason
- `RetrievedContext`: chunk text + source metadata
- `ServiceResponse`: normalized output object before final rendering

### 3.3 Error handling flow
- API failure -> user-friendly fallback + retry hint.
- Empty retrieval -> explain no evidence found + suggest rephrasing.
- Tool failure -> safe error message, no stack traces in UI.

---

## 4) Notebook Deliverables (Solution Output)
The implementation output will be **Python notebooks**. Recommended minimal set:

1. `01_service_api.ipynb`
   - API selection, wrapper function, transformed output examples.

2. `02_service_semantic_search.ipynb`
   - Data loading/chunking, embedding, persistent Chroma index, retrieval QA tests.

3. `03_service_function_calling.ipynb`
   - Tool schemas, call loop, tool execution examples.

4. `04_chat_integration_gradio.ipynb`
   - Persona, memory, routing, unified chat demo.

5. `05_guardrails_and_eval.ipynb`
   - Restricted-topic tests, prompt-injection tests, failure cases, acceptance checklist.

6. `06_end_to_end_demo.ipynb`
   - Final integrated run-through with representative conversations.

> Optional: consolidate into one final notebook if requested by instructors, but multi-notebook structure is easier to debug.

---

## 5) Implementation Phases and Action Plan

### Phase 0 — Setup and scope lock
- Confirm dataset scope and API choice.
- Define persona and refusal tone.
- Freeze service contracts (input/output signatures).

**Exit criteria:** Decisions recorded in `readme.md` draft.

### Phase 1 — Service A complete
- Implement API utility and formatter.
- Add input validation and robust error handling.
- Demonstrate transformed responses for multiple queries.

**Exit criteria:** API responses are never returned verbatim.

### Phase 2 — Service B complete
- Prepare/chunk documents.
- Build embeddings and Chroma persistent store.
- Implement retrieval + grounded answer generation.
- Save retrieval evidence with source references.

**Exit criteria:** Relevant top-k retrieval on test questions.

### Phase 3 — Service C complete
- Define tool schemas and dispatcher.
- Implement function-calling loop.
- Add examples proving tool-trigger behavior.

**Exit criteria:** Service C uses required tooling and returns stable outputs.

### Phase 4 — Integrate chat interface
- Build Gradio UI.
- Add memory storage and retrieval per turn.
- Add intent router across 3 services.

**Exit criteria:** One UI successfully serves all 3 services.

### Phase 5 — Guardrails and hardening
- Add prompt disclosure/modification protection.
- Add restricted-topic refusal policy.
- Test jailbreak variants and adversarial prompts.

**Exit criteria:** Required refusals pass consistently.

### Phase 6 — Documentation and packaging
- Write `readme.md` with architecture, services, embedding process, and run steps.
- Ensure all files remain within assignment constraints.

**Exit criteria:** Repo is runnable under standard environment without extra installs.

---

## 6) Acceptance Checklist (Mapped to Assignment)
- [ ] 3 services implemented.
- [ ] Service A uses external API and transforms output.
- [ ] Service B uses semantic or hybrid search with Chroma persistence.
- [ ] Service C uses Function Calling/Web Search/MCP.
- [ ] Chat UI exists (Gradio preferred).
- [ ] Distinct assistant personality implemented.
- [ ] Conversation memory maintained.
- [ ] Prompt reveal/modify attempts are blocked.
- [ ] Restricted topics refused (cats/dogs, horoscopes/zodiac, Taylor Swift).
- [ ] `readme.md` included with implementation decisions.

---

## 7) Risk Notes and Mitigations
- **Risk:** Dependency mismatch in course environment.  
  **Mitigation:** Reuse existing course libraries only; avoid adding new packages where possible.
- **Risk:** Weak retrieval quality from poor chunking.  
  **Mitigation:** Tune chunk size/overlap and evaluate with a fixed question set.
- **Risk:** Router misclassification.  
  **Mitigation:** deterministic fallback rules and service confidence checks.
- **Risk:** Prompt leakage via indirect attacks.  
  **Mitigation:** layered guardrails (pre-filter + system policy + response check).

---

## 8) Immediate Next Actions
1. Create notebook scaffolds in `05_src/assignment_chat`.
2. Implement Service A first for fast end-to-end vertical slice.
3. Build Service B persistence pipeline with sample dataset.
4. Add Service C function-calling and integrate all into Gradio.
5. Add guardrail tests and finalize `readme.md`.
