## Built a conversational AI assistant enabling employees to perform HR operations using natural language. TypeScript • Node.js • React • OpenAI API • RAG • PostgreSQL • LLM • REST APIs

# 📘 AI-Powered HR Assistant (Interview Revision Notes)

---

# Project Overview

**Project: AI-Powered Conversational HR Assistant**

### Tech Stack

| Layer          | Technology                                 |
| -------------- | ------------------------------------------ |
| Frontend       | React, TypeScript                          |
| Backend        | Node.js, TypeScript, Express/NestJS        |
| AI             | OpenAI GPT, LangChain                      |
| RAG            | OpenAI Embeddings + pgvector (or Pinecone) |
| Database       | PostgreSQL                                 |
| Cache          | Redis                                      |
| Queue          | BullMQ                                     |
| Authentication | JWT + RBAC                                 |
| Deployment     | Docker, AWS                                |
| Monitoring     | OpenTelemetry, Winston                     |

---

# Project in One Sentence

> Built an AI-powered conversational assistant that allows employees to perform HR operations and ask HR-related questions using natural language instead of navigating multiple screens.

---

# Business Problem

Traditional HR portals require users to

- Navigate multiple menus
- Search through documents
- Remember where features exist
- Contact HR for common questions

Example

Without AI

```
Dashboard
↓

HR Module
↓

Leave
↓

Apply Leave
↓

Choose Date
↓

Submit
```

With AI

```
"Apply casual leave for tomorrow."

↓

Done ✅
```

---

# What Services Does It Provide?

The chatbot sits **on top of the HRMS**.

It can perform

- Apply Leave
- Cancel Leave
- Check Leave Balance
- Attendance Summary
- Download Payslip
- Payroll Details
- Employee Search
- Holiday Calendar
- HR Policy Q&A
- Insurance Questions
- Organization Chart
- Notifications

---

# High-Level Architecture

```
Employee

↓

React Chat UI

↓

Node.js API Gateway

↓

Authentication (JWT)

↓

Conversation Manager

↓

Decision Layer

↓

Need AI?

├── No
│      ↓
│   Direct API Call
│
└── Yes
       ↓
   OpenAI + LangChain

↓

HR Services

↓

PostgreSQL

↓

Response
```

---

# When is the LLM Used?

Only when reasoning is required.

LLM is used for

✅ Intent Detection

```
"I need leave next Friday"
↓

Apply Leave
```

---

Entity Extraction

```
Apply casual leave tomorrow

↓

Date

Leave Type

Employee
```

---

Planning

```
I'm on leave tomorrow.

Cancel meetings.

Notify manager.

↓

Break into tasks.
```

---

Policy Questions

```
How many maternity leaves are allowed?

↓

RAG

↓

Answer
```

---

Summarization

```
Summarize attendance

Summarize payroll

Summarize leave history
```

---

Natural Language Response

```
Backend returns JSON

↓

LLM converts into friendly response.
```

---

# When is LLM NOT Used?

Never use LLM for deterministic tasks.

Examples

```
Get Leave Balance

Download Payslip

Get Employee Profile

Attendance Records

CRUD Operations
```

Backend directly calls APIs.

---

# Why Not Use LLM for Everything?

Because

- Expensive
- Slower
- Less reliable
- Hallucination risk

Backend already knows

- Employee ID
- Database schema
- APIs

No reasoning needed.

---

# RAG Workflow

```
User Question

↓

Embedding

↓

Vector Search

↓

Top 5 Chunks

↓

OpenAI

↓

Answer
```

Example

```
Can my wife be added to insurance?
```

Retrieve

Insurance Policy PDF

↓

LLM answers only using retrieved context.

---

# HR Operations Workflow

User

```
Apply leave tomorrow.
```

LLM

↓

Intent Detection

↓

Extract Date

↓

Tool Selection

↓

Leave Service

↓

PostgreSQL

↓

Return Success

---

# Microservices

Instead of one monolithic backend

```
API Gateway

↓

Leave Service

Attendance Service

Payroll Service

Employee Service

Policy Service

Notification Service

Authentication Service
```

---

## Leave Service

Responsible for

- Apply Leave
- Cancel Leave
- Leave Balance

Database

```
Leave Tables
```

---

## Attendance Service

Responsible for

- Attendance
- Working Hours
- Missed Days

---

## Payroll Service

Responsible for

- Payslip
- Salary
- Tax
- Deductions

---

## Employee Service

Responsible for

- Employee Profile
- Manager
- Department
- Organization Chart

---

## Notification Service

Responsible for

- Email
- SMS
- Push Notification

---

## Policy Service

Responsible for

- HR Policies
- Insurance
- Travel
- Employee Handbook

Works with RAG.

---

# Database

Structured

```
PostgreSQL

Employees

Leave

Attendance

Payroll

Departments
```

Indexes

```
employee_id

department_id

leave_date

attendance_date
```

---

Unstructured Data

```
Employee Handbook

HR Policy

Insurance

Travel Policy

Holiday Calendar
```

↓

Embedding

↓

Vector Database

---

# Authentication

```
JWT

↓

User Identity

↓

Role

↓

Permissions
```

Example

Employee

```
Apply Leave

✔

Approve Leave

✖
```

LLM never checks permissions.

Backend does.

---

# Conversation Management

Instead of sending

```
Entire Chat
```

Send

```
Conversation Summary

+

Last 5 Messages
```

Reduces token usage.

---

# Tool Calling

LLM never executes SQL.

LLM returns

```
Tool

↓

applyLeave()
```

Backend executes

```
Leave Service

↓

PostgreSQL
```

---

# Production Flow

```
Employee

↓

Chat UI

↓

Authentication

↓

Need AI?

↓

No

↓

Direct API

↓

Database

↓

Response

-------------------

Need AI?

↓

Intent Detection

↓

Entity Extraction

↓

Reasoning

↓

Tool Selection

↓

Microservice

↓

Database

↓

LLM formats response

↓

Employee
```

---

# Why Do We Need a Chatbot?

Because employees don't care where functionality exists.

They just want to say

```
Apply leave.

Download payslip.

Show attendance.

How many holidays are left?
```

The chatbot becomes the

**Natural Language Interface**

to the HRMS.

---

# Advantages

- Better User Experience
- Faster HR Operations
- Reduced HR Support Tickets
- 24×7 Availability
- Less Employee Training
- Faster Policy Search
- Single Entry Point for HRMS

---

# Interview Questions & Answers

---

## Q1 Explain your project.

**Answer**

> We built an AI-powered conversational assistant for an HRMS that enables employees to perform HR operations and retrieve HR information using natural language. Instead of navigating multiple modules, users simply describe what they need. The assistant understands the request, invokes the appropriate backend microservice for operational tasks, or uses RAG for policy-related questions. The LLM is used only where reasoning is required, while business logic and security remain in backend services.

---

## Q2 Why do we need a chatbot?

**Answer**

Because users don't want to navigate multiple screens.

They simply say

```
Apply Leave

Download Payslip

Check Attendance
```

The chatbot becomes the easiest interface.

---

## Q3 Why use LLM?

Answer

Only for reasoning.

Not for CRUD.

---

## Q4 Why not use LLM for every request?

Because

- Cost
- Latency
- Hallucinations

Simple operations use backend APIs.

---

## Q5 What is RAG?

Answer

Retrieve relevant company documents first, then provide those as context to the LLM so answers are based on current HR policies instead of the model's memory.

---

## Q6 Why LangChain?

Answer

To orchestrate LLM calls, manage prompts, integrate tools, maintain conversation memory, and build multi-step AI workflows more easily.

---

## Q7 What happens when someone asks

```
Apply leave tomorrow.
```

Answer

```
Chat

↓

Intent Detection

↓

Entity Extraction

↓

applyLeave()

↓

Leave Service

↓

Database

↓

Response
```

---

## Q8 What happens when someone asks

```
How many maternity leaves are allowed?
```

Answer

```
Embedding

↓

Vector Search

↓

Relevant Policy

↓

OpenAI

↓

Answer
```

---

## Q9 How do you reduce token cost?

- Conversation summary instead of full history
- Send only the last few messages
- Retrieve only relevant RAG chunks
- Cache common responses
- Skip the LLM for deterministic requests

---

## Q10 How do you secure the system?

- JWT Authentication
- RBAC Authorization
- Backend validation
- Audit logging
- LLM cannot directly access the database or execute SQL

---

## Q11 Does the LLM access the database directly?

No.

The LLM only selects or recommends a tool.

The backend executes the business logic and database operations.

---

## Q12 What if the LLM is unavailable?

Operational requests can still be served through direct APIs or fail gracefully with an appropriate message. Critical business operations do not depend on the LLM.

---

## Q13 Why use microservices?

- Independent deployment
- Easier scaling
- Team ownership
- Fault isolation
- Technology flexibility for different domains

---

## Q14 How would you improve this project?

- Add an AI Agent for multi-step task planning
- Use Model Context Protocol (MCP) for standardized tool integration
- Add semantic caching for repeated questions
- Introduce event-driven notifications with Kafka or RabbitMQ
- Stream LLM responses for a better user experience
- Add multilingual support
- Fine-tune prompts and implement evaluation metrics for answer quality

---

# Final Interview Summary (30 Seconds)

> "Our AI-powered HR Assistant provides a natural language interface over an existing HRMS. Employees can perform tasks like applying leave, checking attendance, downloading payslips, and asking policy questions through a chat interface. We use React, Node.js, TypeScript, PostgreSQL, LangChain, OpenAI, and RAG. The LLM is used only for understanding, reasoning, planning, and generating responses, while all business logic, security, and database operations are handled by backend microservices. This architecture improves user experience, reduces support effort, lowers AI costs, and keeps the system secure and scalable."
