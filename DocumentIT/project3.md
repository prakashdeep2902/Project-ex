# AI-Powered Conversational HR Assistant — Interview Revision Notes

Project: AI-Powered Conversational HR Assistant
Used by: Employees, Managers, HR team, and the CEO (leadership-level users, not just employees)
My Role: Backend microservices, intent detection + tool-calling (LangChain/OpenAI), RAG pipeline, Authentication (JWT + RBAC)

---

### Q1. What problem was the team facing, and how did you resolve it?

**Problem:**

This assistant was mainly built for HR team, managers, and the CEO — people in leadership or decision-making positions, not just regular employees. Their problem was different from a normal employee's.

They needed to check things like team attendance, leave approvals, payroll summaries, headcount, or policy details — but doing this meant logging into the HRMS, going through multiple menus, filtering by department or team, and manually reading reports. For someone like the CEO or a manager, who's busy and doesn't have time to click through several screens just to check "how many people are on leave today" or "approve this leave request," this was slow and inefficient.

Also, approvals were a bottleneck — managers had pending leave requests sitting in the system because logging in and finding them took effort, so approvals got delayed.

**Solution:**

We built a conversational AI assistant on top of the HRMS. Instead of navigating screens, a manager or CEO could just type or ask in natural language — like "show me who's on leave this week in my team" or "approve Rahul's leave request" — and the assistant would understand the intent, call the right backend service, and either fetch the answer or perform the action directly.

For operational tasks like approvals or fetching data, the backend handled it directly — no AI reasoning needed there, just intent detection and calling the right API. For policy-related or knowledge questions — like "what's our maternity leave policy" — we used RAG, pulling from actual company policy documents so the answer was accurate and specific to our company, not generic.

This made it much faster for leadership to get information or take action, without navigating the full HRMS.

---

### Q2. What was your role, and what technical issues did you face?

**My role:** Backend microservices (leave, attendance, payroll, employee, policy), intent detection and tool-calling flow with LangChain + OpenAI, RAG pipeline for policy questions, and authentication/role-based access (since HR, managers, and the CEO all have different access levels).

**Issue 1 — Sensitive data access based on role**

> **Q: How did you make sure a manager or employee couldn't see or act on data they're not allowed to?**
> A manager should see only their team's attendance/leave, not the whole company's, and shouldn't be able to approve another team's leave. I made sure the LLM never decided permissions — it only recommends an action or tool call. The actual permission check happens in the backend using JWT + RBAC. Even if someone asks for something outside their access, the backend blocks it before hitting the database, regardless of what the AI suggested.

**Issue 2 — LLM picking the wrong intent for ambiguous requests**

> **Q: How did you handle requests that could mean different things depending on who's asking?**
> Requests like "leave status" could mean "my leave balance" (employee) or "team's pending approvals" (manager). The assistant sometimes picked the wrong intent. I fixed this by passing the user's role into the prompt context, so the LLM interprets "leave status" differently for a manager vs. an employee. This reduced wrong tool selection.

**Issue 3 — Token cost and slow responses for policy questions**

> **Q: How did you keep AI responses fast and affordable, especially for busy leadership users?**
> Sending full conversation history and large policy documents to OpenAI made responses slow and expensive. I used RAG properly — only retrieved top relevant chunks from the vector DB instead of full documents, and sent conversation summary + last few messages instead of full chat history.

**Issue 4 — LLM trying to execute actions in an unstructured way**

> **Q: What happened when the LLM's output wasn't in the expected format, and how did you fix it?**
> Sometimes the LLM's response wasn't in a strict tool-call format, causing failures when the backend tried to execute it. I strictly defined tool schemas — the LLM could only return a predefined tool name with specific parameters (e.g., `applyLeave(date, type)`), nothing else. Backend validated the structure before executing anything.

---

### Q3. How did you tackle the overall solution?

Broke the problem into two clear parts and handled each differently instead of routing everything through the LLM:

- **Operational tasks** (leave balance, approve leave, attendance summary): LLM only did intent detection + entity extraction (date, leave type, employee), mapped to a specific backend function like `applyLeave()` or `approveLeave()`. Execution, database calls, and permission checks always happened in the backend, never the AI.

- **Policy questions** (maternity leave rules, insurance queries): Used RAG — retrieved only relevant chunks from company documents in a vector database, passed just that context to OpenAI, so answers stayed accurate and specific.

- **Access control**: Role and permission checks always happened in the backend using JWT + RBAC. The AI never decided who can see or do what — it could only suggest an action, and the backend allowed or blocked it.

- **Cost and speed**: Sent conversation summaries instead of full chat history, retrieved only top RAG chunks instead of full documents, and skipped the LLM entirely for simple deterministic requests, calling APIs directly.

**Core principle:** Let AI handle understanding and reasoning, but keep all execution, security, and business logic strictly in the backend — this is what made the solution reliable enough for leadership-level users like the CEO to trust and use.

---

### Q4. What tech stack did you use, and where?

| Layer          | Technology                            | Where used                                                                                                                                                      |
| -------------- | ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend       | React, TypeScript                     | Chat interface for employees, managers, and CEO (built by colleague/frontend team)                                                                              |
| Backend        | Node.js, TypeScript, Express/NestJS   | Leave, Attendance, Payroll, Employee, Policy, Notification microservices; intent routing; tool execution; API endpoints                                         |
| AI             | OpenAI GPT, LangChain                 | Intent detection, entity extraction, natural language responses; LangChain for orchestration, prompt management, tool calling, conversation memory              |
| RAG            | OpenAI Embeddings + pgvector/Pinecone | Policy questions — company docs (HR policy, insurance, travel policy) embedded and stored; similarity search retrieves relevant chunks before sending to OpenAI |
| Database       | PostgreSQL                            | Structured data — employees, leave, attendance, payroll, departments; indexed on `employee_id`, `leave_date`, `attendance_date`                                 |
| Cache          | Redis                                 | Caching frequent responses to reduce repeated DB/AI calls                                                                                                       |
| Queue          | BullMQ                                | Background jobs — e.g., notifying employee after leave approval, without blocking main request                                                                  |
| Authentication | JWT + RBAC                            | Identifying users and enforcing role-based access — backend-only check, never the AI                                                                            |
| Deployment     | Docker, AWS                           | Containerizing and deploying services for scalability                                                                                                           |
| Monitoring     | OpenTelemetry, Winston                | Logging and tracing requests across services — debugging slow AI responses or failed tool calls                                                                 |

**My main hands-on areas:** Backend (Node.js/TypeScript), AI integration (OpenAI + LangChain), RAG pipeline, PostgreSQL, and Authentication (JWT + RBAC).

---

## Quick Recap (30-second version)

> "This was a conversational AI assistant built on top of our HRMS, used by HR, managers, and the CEO — leadership-level users who needed fast access to team data and approvals without navigating multiple screens. I built the backend microservices, the intent detection and tool-calling flow using LangChain and OpenAI, the RAG pipeline for policy questions, and the role-based access control using JWT. The key design principle was: AI only handles understanding and reasoning — like detecting intent or answering policy questions via RAG — while all execution, permissions, and business logic stay strictly in the backend. This kept the system fast, secure, and cost-effective enough for leadership to actually rely on."
