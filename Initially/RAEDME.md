# CareDesk Agent

## Execution and Inspection Log

- Ran list files in workspace
- Ran inspect CareDesk notebook cells · Exited with code 1
- Ran parse CareDesk notebook using PowerShell · Exited with code 1
- Ran check matching cells in notebook
- Ran print cell contents
- Ran print cells 18 to 28

**Error:** Our servers are experiencing high traffic right now, please try again in a minute.  
**Error ID:** `22d8289b-73a3-4d72-be01-45b585aadc0e-13-2010`

- Ran inspect cells 23 to 26 in full
- Ran check available PDF/Python binaries · Exited with code 1
- Ran extract text from CareDesk_Video_Code_Guide.pdf
- Canceled check installed packages in py
- Checked task Check installed packages in py
- Killed task Check installed packages in py
- Analyzed CareDesk_Video_Code_Guide.pdf

---

Based on the files in this folder (**[CareDesk_Midterm.ipynb](file:///c:/My%20pc/Work/Agentic%20RAG%20Echo/Session%205/Repo%20folder/CareDesk_Midterm.ipynb)** and **[CareDesk_Video_Code_Guide.pdf](file:///c:/My%20pc/Work/Agentic%20RAG%20Echo/Session%205/Repo%20folder/CareDesk_Video_Code_Guide.pdf)**), here are the four details for the **CareDesk Agent**:

## 1. Use Case

- **Hospital Staff Operations Assistant** for NileCare General Hospital.
- It answers internal operational questions from nurses and HR staff regarding:
  - Annual leave requests and remaining leave balances.
  - Ward shift coverage validation (checking if leave violates minimum staff requirements).
  - Staff isolation room access clearance.
  - Department lead escalation and contact routing.
- **Strict Out-of-Scope Boundary:** It is strictly for internal staff operations and administrative queries; it explicitly rejects and refuses medical diagnosis, treatment protocols, or patient care questions (returning `NOT FOUND`).

## 2. LLM Used and Provider

- **LLM:** `Qwen/Qwen2-1.5B-Instruct` (loaded locally in 4-bit quantization via `BitsAndBytesConfig` and PyTorch).
- **Provider / Model Family:** **Alibaba Cloud / Qwen Team** (available via the **Hugging Face** Hub; run locally on GPU / Colab T4 via Hugging Face Transformers, not through an external paid API).
- **Auxiliary Model for Embeddings:** `sentence-transformers/all-MiniLM-L6-v2` via Hugging Face.

## 3. Tools of the Agent

CareDesk wraps four Python functions as LangChain `StructuredTool` objects that query the hospital's staff roster sheet:

1. **`get_staff`**
   - **Purpose:** Looks up an employee in the roster by name.
   - **Returns:** Role, ward, remaining leave balance, start date, on-shift status, isolation clearance, manager email, days employed, and probation status (`< 90 days`).

2. **`check_leave_balance`**
   - **Purpose:** Compares requested leave days against remaining leave balance.
   - **Returns:** Whether the balance is sufficient and verifies whether probation leave rules are satisfied (max 2 days allowed during probation).

3. **`check_shift_coverage`**
   - **Purpose:** Verifies whether granting leave would drop the ward below minimum mandatory staffing levels (ICU: 3, ER: 2, Pediatrics: 2).

4. **`get_department_contact`**
   - **Purpose:** Retrieves the department lead's name and email address for a given employee or department (ICU, ER, Pediatrics, or HR).

In addition, the agent uses a **FAISS vector retriever** (`retrieve_chain`) over the chunked hospital staff policy PDF.

## 4. System Prompt / Prompt Templates

Because a compact 1.5B model is prone to hallucinating facts or contradicting business rules, CareDesk is orchestrated via **LangChain LCEL** (`parse | tools | retrieve | decide | speak | remember`). Rather than relying on an autonomous agent with a single monolithic chat system prompt, it uses targeted prompt templates:

### Policy Quote Prompt (`quote_chain`)

```text
Use only this context. Reply in one short sentence.
Context: {context}
Question: {question}
Short answer:
```

### Memory Summarization Prompt (`refresh_summary`)

```text
Summarize these hospital staff questions and answers in 3 bullet points. Factual only.
Chat:
{buffer_text}
Summary:
```

### Final Decision & Response Guard

The spoken response generation is anchored by a deterministic rule-based builder (`build_answer`) using the Python decision output (`ALLOW`, `DENY`, `ESCALATE`, `INFO`, `NOT FOUND`) alongside tool facts, preventing the model from overriding policy decisions or leaking prompt instructions.
