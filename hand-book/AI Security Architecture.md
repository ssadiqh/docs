# Part 9 — AI Security Architecture

## Purpose of This Chapter

This chapter introduces the security concepts required to design and review Enterprise AI solutions.

The goal is not to become a security specialist.

The goal is to understand:

- AI-specific threats  
- Enterprise RAG security  
- Tenant isolation  
- Agent security  
- MCP security  
- Security architecture review principles

---

# AI Security in One Page

Before learning individual threats, controls, and architectures, it is useful to understand the single mental model that explains most AI security risks.

Almost every AI attack is attempting to do one of three things:

1\. Change what AI thinks

2\. Change what AI sees

3\. Change what AI does

This simple framework can be used to understand, categorize, and review nearly every AI security threat.

## Change What AI Thinks

The attacker attempts to influence the model's reasoning, instructions, or decision-making.

Examples:

- Prompt Injection  
- Jailbreaks  
- Instruction Override  
- Prompt Leakage

Attacker

↓

Prompt

↓

Model Reasoning

Goal: Manipulate how the AI thinks.

This is the primary focus of Prompt Security.

## Change What AI Sees

The attacker attempts to influence the information available to the AI.

Examples:

- Unauthorized Retrieval  
- Context Poisoning  
- Malicious Documents  
- Sensitive Data Exposure  
- Cross-Tenant Data Leakage

Attacker

↓

Context / Retrieval

↓

Model

Goal: Manipulate what the AI sees.

**For enterprise AI systems, this is usually the most important security category** because enterprise knowledge represents enterprise risk.

Most enterprise AI security incidents are retrieval failures rather than model failures.

## Change What AI Does

The attacker attempts to influence the actions performed by an AI system.

Examples:

- Tool Abuse  
- Excessive Permissions  
- Unauthorized Actions  
- Unsafe Tool Invocation  
- Autonomous Agent Misuse

Attacker

↓

Agent

↓

Tool

↓

Action

Goal: Manipulate what the AI does.

As AI systems become increasingly agentic, the risk shifts from generating a wrong answer to performing a wrong action.

## Mapping to Enterprise AI Security

| Security Domain | Primary Objective |
| :---- | :---- |
| Prompt Security | Protect how AI thinks |
| RAG & Retrieval Security | Protect what AI sees |
| Agent & MCP Security | Protect what AI does |
| Defense-in-Depth | Protect all three |

## Architect Perspective

When reviewing an AI architecture, start with three questions:

Can an attacker change what the AI thinks?

Can an attacker change what the AI sees?

Can an attacker change what the AI does?

If these questions are answered, the majority of AI security risks have been identified.

## Key Takeaway

Think → See → Do

is the simplest mental model for understanding AI security.

Almost every AI attack ultimately attempts to manipulate:

- How the AI thinks  
- What the AI sees  
- What the AI does

The remainder of this chapter explores the threats, controls, and architectural patterns used to protect each of these areas.

---

# 1\. Why AI Security Is Different

Traditional application security focuses on:

User → Application → Database

AI systems introduce additional attack surfaces:

User → Prompt → Retriever → Vector Database → LLM → Tools → Memory → Response

AI Security \= Protecting the Entire AI Pipeline

---

# 2\. Security Mental Models

## Mental Model 1

Protect the Data First. Protect the Model Second.

## Mental Model 2: The LLM Is Usually Innocent

Most enterprise AI security incidents are retrieval failures, not model failures.

User

↓

Retriever fetches wrong document

↓

LLM processes it correctly

↓

Correct answer from wrong data

↓

Security incident

The LLM is doing its job. The architecture failed.

Root cause: **Retriever**, not model.

This is why enterprise architects focus on retrieval security above all else.

---

# 3\. The Seven Attack Surfaces

Understanding where attacks happen is essential for threat modeling.

| Layer | What Happens Here | Primary Risks | Defense |
| :---- | :---- | :---- | :---- |
| **Prompt** | User input arrives | Direct prompt injection | Input validation, firewall |
| **Context** | System prompts, history, memory assembled | Indirect injection, context poisoning | Sanitization, guardrails |
| **Retrieval** | Documents fetched from knowledge base | Unauthorized access, cross-tenant leakage | Authorization, metadata filtering |
| **Model** | LLM processes context | Data leakage, model extraction | Data governance, rate limiting |
| **Tools** | Agent executes actions | Tool abuse, escalation | Tool allow lists, human approval |
| **Memory** | Long-term context stored | Memory poisoning, cross-user leakage | Access controls, isolation |
| **Output** | Response sent to user | PII exposure, sensitive data leakage | Output guardrails, detection |

---

# 4\. The Golden Rules

These principles guide all AI security decisions.

| Rule | Why It Matters | When It Breaks |
| :---- | :---- | :---- |
| **Prompts are not security controls** | Prompts can be overridden or poisoned | Relying solely on system prompts for security |
| **Authorization before retrieval** | Once retrieved, data is already accessed | Retrieve-then-filter approach (too late) |
| **Tenant boundaries are architectural** | Filters can fail; architecture cannot | Using RBAC alone without storage isolation |
| **The LLM is usually innocent** | Most incidents are retrieval failures | Blaming model when retriever failed |
| **Data first, model second** | Enterprise data is enterprise risk | Focusing on model security before data security |

---

# 5\. Prompt Injection & Jailbreaks

### Prompt Injection

Attempts to override intended instructions.

**Direct Prompt Injection**

Attacker directly provides malicious instructions.

**Indirect Prompt Injection**

Malicious instructions are hidden inside retrieved content.

Document → Retriever → LLM

The document becomes the attack vector.

**Prompt Leakage**

Attempts to expose hidden system prompts.

**Data Exfiltration**

Attempts to retrieve unauthorized information.

### Jailbreaks

Attempts to bypass model safety controls.

#### Prompt Injection Framework

| Threat | Type | Detection | Prevention |
| :---- | :---- | :---- | :---- |
| **Prompt Injection** | Direct | Pattern matching (ignore, disregard, override) | Input firewall, sanitization |
|  | Indirect | Document analysis | Retrieval authorization |
| **Prompt Leakage** | Extraction | Queries asking for system prompt | Output guardrails |
| **Data Exfiltration** | Manipulation | Unusual retrieval patterns | Authorization checks |
| **Jailbreak** | Bypass | Safety mechanism circumvention attempts | Model selection, output detection |

### Key Principle

**Prompts are not security controls.**

---

# 6\. Enterprise RAG Security

### Architect Insight: Retrieval Security Is Most Important

Retrieval security is the most important enterprise AI security topic.

Because: Enterprise knowledge \= Enterprise risk.

Most AI security incidents stem from retrieval failures, not prompt attacks or model issues.

### Golden Rule

Users should never gain access through AI to information they could not access without AI.

### Secure Retrieval Flow

User → Authentication → Authorization → Retriever → Authorized Documents → LLM

#### Critical Principle

Authorization BEFORE Retrieval

### Enterprise RAG Security Checklist

| Component | Question | Correct Answer |
| :---- | :---- | :---- |
| **Authorization** | When is access checked? | BEFORE retrieval, not after |
| **Metadata Filtering** | Which documents are searchable? | Only user-authorized documents |
| **Document Security** | Who can see each document? | Access control enforced per-document |
| **Row-Level Security** | Can users see all rows in a table? | No, only authorized rows |
| **Chunking** | Can restricted data end up in searchable chunks? | No, sanitize before chunking |
| **Indexing** | What gets indexed? | Only approved, authorized content |
| **Auditability** | Can you explain why this answer was generated? | Yes, audit trail shows retrieved docs |

### Core Controls

- Retrieval Authorization  
- Document-Level Security  
- Row-Level Security  
- Metadata Security Filtering  
- Retrieval Auditability

---

# 7\. Tenant Isolation

Authentication \= Who are you?

Authorization \= What can you access?

Tenant Isolation \= Whose data can you access?

### Isolation Models

| Model | Isolation | Cost | Operations | Blast Radius | Best For |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Shared Collection** | Low | Low | Simple | All tenants | Startup |
| **Collection Per Tenant** | Medium | Medium | Moderate | Single tenant | Enterprise SaaS |
| **Database Per Tenant** | High | High | Complex | Single tenant | Defence/Government |

### Cross-Tenant Leakage

NAB User → Retrieves → ANZ Documents

Major security incident.

### Tenant Context Propagation

**Architect Insight:** Tenant boundaries must be enforced across every layer.

User (with Tenant Context)

↓

Retriever (tenant-filtered)

↓

Agent (tenant-aware)

↓

Tool (tenant-isolated)

Tenant context established at authentication and propagated through every component.

---

# 8\. LLM Security

For most enterprises:

RAG Security \> LLM Security

### Risk Hierarchy

| Risk | Likelihood | Impact | Effort to Defend | Priority |
| :---- | :---- | :---- | :---- | :---- |
| **Training Data Leakage** | Medium | High | Medium | High |
| **Unsafe Fine-Tuning** | High | Medium | Low | High |
| **Model Extraction** | Medium | Medium | Medium | Medium |
| **Model Theft** | Low | High | High | Medium |
| **Adversarial Inputs** | Low | Low | High | Low |

### Major Risks

- Model Theft  
- Model Extraction  
- Training Data Leakage  
- Unsafe Fine-Tuning  
- Open Source Model Risks

### Key Questions

- Where did it come from?  
- Can it be trusted?  
- Who maintains it?

---

# 9\. Agent Security

Traditional LLM:

Wrong Answer

Agent:

Wrong Action

### Agent Security Framework

| Risk Category | Examples | Control | Enforcement |
| :---- | :---- | :---- | :---- |
| **Excessive Permissions** | Agent has access to all tools | Tool allow lists | Policy engine |
| **Tool Abuse** | Agent sends email to all customers | Least privilege | Per-tool authorization |
| **Memory Risks** | Attacker poisons stored instructions | Memory isolation | Per-user/tenant memory |
| **Autonomous Actions** | Agent deletes customer without approval | Human-in-the-loop | Action approval workflow |

### Agent Permission Model

| Level | Capability | Risk | Approval Required |
| :---- | :---- | :---- | :---- |
| **Read** | View information | Low | No |
| **Recommend** | Suggest actions | Low | No |
| **Draft** | Prepare actions (email, ticket, etc.) | Low | No |
| **Execute** | Perform actions | Medium | Yes (for high-risk) |
| **Autonomous** | Execute without approval | High | Rarely used |

### Risky Actions Requiring Approval

- Money transfers / refunds  
- Customer deletion / account changes  
- Production deployments  
- Mass communications  
- Policy exceptions  
- Sensitive document access

### Core Controls

- Least Privilege  
- Tool Allow Lists  
- Human-in-the-Loop Pattern  
- Action Approval Workflow

---

# 10\. MCP Security

### Mental Model

RAG \= Document Discovery

MCP \= Tool Discovery

RAG requires Document Authorization.

MCP requires Tool Authorization.

### MCP \= RAG for Tools

| Concept | RAG | MCP |
| :---- | :---- | :---- |
| What's discovered | Documents | Tools |
| Discovery control | Document visibility | Tool visibility |
| Authorization | Document access control | Tool invocation control |
| Isolation | Tenant document isolation | Tenant tool isolation |

### MCP Security Model

| Aspect | Question | Correct Answer |
| :---- | :---- | :---- |
| **Discovery** | Can agent see all available tools? | Only authorized tools |
| **Visibility vs Permission** | If agent sees a tool, can it use it? | Not necessarily—see ≠ execute |
| **Authorization** | Who controls what tools agent can invoke? | Policy engine, not agent |
| **Tenant Awareness** | Can Tenant A's agent access Tenant B's tools? | No—tools are tenant-isolated |
| **Authentication** | How does tool know who's calling it? | User identity \+ agent identity |

### Major Risks

- Tool Discovery  
- Tool Authorization  
- Tool Isolation  
- Data Leakage

### Core Principle

An agent should only discover and use authorized tools.

---

# 11\. Defense-in-Depth

Layer 1: Identity & Authentication

↓ (Establish who is using the system)

Layer 2: Authorization & Access Control

↓ (Determine what they're allowed to do)

Layer 3: Input Controls

↓ (Validate and filter inputs)

Layer 4: Retrieval Security

↓ (Enforce document/data access)

Layer 5: Processing (LLM)

↓ (Model with safeguards)

Layer 6: Tool Controls

↓ (Enforce agent actions)

Layer 7: Output Controls

↓ (Detect and block sensitive data)

Layer 8: Monitoring & Audit

↓ (Log everything for investigation)

**Key Principle:** If one layer fails, the next layer catches it.

---

# 12\. Threat-to-Control Mapping

Quick reference for threat-to-control mapping.

| Threat | Root Cause | Primary Control | Secondary Control |
| :---- | :---- | :---- | :---- |
| Prompt Injection | User input | Input firewall | Output guardrails |
| Indirect Injection | Retrieved document | Retrieval authorization | Output guardrails |
| Unauthorized Retrieval | No access check | Authorization | Metadata filtering |
| Cross-Tenant Leakage | Tenant filter fails | Tenant isolation | Monitoring |
| Tool Abuse | No permission limits | Tool allow lists | Human approval |
| Memory Poisoning | Memory unprotected | Memory access control | Monitoring |
| PII Exposure | No output check | Output guardrails | DLP/detection |
| Model Extraction | API open to all | Rate limiting | Usage monitoring |

---

# 13\. Architecture Review Framework

### Identity & Access Questions

- [ ] How is tenant context established and propagated?  
- [ ] Is authorization checked BEFORE or AFTER retrieval?  
- [ ] Can a user see tenant data they're not authorized for?

### Retrieval Security Questions

- [ ] Which documents can be retrieved?  
- [ ] Are metadata filters enforced before search?  
- [ ] Can unauthorized documents be found even if filters fail?  
- [ ] What's the audit trail for document access?

### Agent Security Questions

- [ ] Which tools can the agent access?  
- [ ] Which actions require human approval?  
- [ ] What's the blast radius if agent is compromised?  
- [ ] Is agent memory isolated per user/tenant?

### MCP Security Questions

- [ ] Which tools can the agent discover?  
- [ ] Can discovery be abused to find sensitive tools?  
- [ ] How are tool invocations authorized and logged?

### Data Protection Questions

- [ ] What sensitive data exists in documents?  
- [ ] Can it be exposed through retrieval or output?  
- [ ] Are outputs checked for PII/secrets before sending to user?

---

# 14\. Decision Frameworks

### Model Selection

| Factor | Azure OpenAI | AWS Bedrock | Self-Hosted |
| :---- | :---- | :---- | :---- |
| **Security responsibility** | Provider | Provider | You |
| **Data residency control** | Limited | Medium | Full |
| **Fine-tuning governance** | Provider managed | Provider managed | Your responsibility |
| **Model extraction risk** | Low | Low | Your responsibility |
| **Training data risk** | Provider policy | Provider policy | Your data only |
| **Cost** | Pay-per-use | Pay-per-use | High fixed cost |
| **Best for** | Most enterprises | AWS shops | Sensitive data |

### Retrieval Architecture

| Consideration | Shared Collection | Collection Per Tenant | Database Per Tenant |
| :---- | :---- | :---- | :---- |
| **Cost** | Lowest | Medium | Highest |
| **Operational complexity** | Lowest | Medium | Highest |
| **Failure risk** | Highest | Low | Low |
| **Tenant isolation** | Filter-based | Architectural | Complete |
| **Blast radius if filter fails** | All tenants | Single tenant | Single tenant |
| **When to use** | Early startup | Enterprise SaaS | Defence/Government/Healthcare |

### Agent Authority Levels

| Question | Answer | Authority Level |
| :---- | :---- | :---- |
| Can agent read information? | Yes | Read+ |
| Can agent suggest actions? | Yes | Recommend+ |
| Can agent draft outputs? | Yes | Draft+ |
| Can agent execute low-risk actions? | Yes | Execute (low-risk) |
| Can agent execute high-risk actions? | Only with approval | Execute (with HITL) |
| Can agent act autonomously? | Almost never | Autonomous (rare) |

---

# 15\. Common Failure Modes

Know what breaks and how to prevent it.

| Failure | Root Cause | How It's Found | How to Prevent |
| :---- | :---- | :---- | :---- |
| **Cross-tenant leakage** | Tenant filter missing or broken | Monitoring, audit log review | Architectural isolation (not filters) |
| **Unauthorized retrieval** | Authorization checked after retrieval | User can access restricted data | Enforce auth BEFORE retrieval |
| **PII exposure** | No output guardrails | User receives sensitive data | Output guardrails mandatory |
| **Tool abuse** | Agent has too many permissions | Agent performs unintended action | Tool allow lists \+ approval |
| **Memory poisoning** | Attacker stores malicious instructions | Agent follows bad instructions | Memory isolation \+ monitoring |
| **Model extraction** | No rate limiting | Attacker builds model approximation | Rate limits \+ usage monitoring |

---

# 16\. Security Metrics to Track

| Metric | What It Reveals | Threshold |
| :---- | :---- | :---- |
| **Retrieval denials per user** | Authorization working? | 0-1% denial rate normal |
| **Cross-tenant retrieval attempts** | Isolation failures? | Should be zero |
| **Injection detection triggers** | Attack attempts? | Flag anything above baseline |
| **Tool execution denials** | Permission model working? | Should be rare |
| **Output guardrail blocks** | Data leakage prevention? | Review all blocks |
| **Failed human approvals** | Agent behavior correct? | Investigate spikes |
| **Audit log gaps** | Logging complete? | Should be zero |

---

# 17\. Key Principles Summary

The mental model that ties everything together.

| Principle | Application |
| :---- | :---- |
| **Protect data first, model second** | Focus retrieval and data governance before model selection |
| **The LLM is usually innocent** | Investigate retrieval when incidents happen, not the model |
| **Prompts are not security controls** | Never rely on system prompts for security decisions |
| **Authorization before retrieval** | Check access before fetching documents, not after |
| **Tenant boundaries are architectural** | Use storage isolation (collections, databases), not filters |
| **Least privilege for agents** | Agents get only tools they need, nothing more |
| **Human-in-the-loop for high-risk** | Money, data deletion, mass actions require approval |
| **Monitor everything** | Log prompts, retrievals, tool calls, security events |
| **Defense-in-depth always** | If one control fails, the next one catches it |

---

# 18\. Escalation Triggers

When to involve the security team.

- Retrieval across tenant boundaries  
- Agent requesting tools outside allow list  
- Repeated injection detection triggers  
- Audit logs with gaps  
- Output guardrails blocking sensitive data  
- Fine-tuning on proprietary data  
- Requests for autonomous agent authority  
- Model extraction attempts (rate limit spikes)

---

# Key Takeaways

1. Prompt & Jailbreak Security  
2. Enterprise RAG Security (Most Important)  
3. Tenant Isolation  
4. LLM Security  
5. Agent Security  
6. MCP Security

Remember:

- Protect the Data First. Protect the Model Second.  
- The LLM is usually innocent. Most incidents are retrieval failures.  
- Retrieval security is the most important enterprise AI security topic.  
- Tenant boundaries must be enforced across every layer.  
- AI Security \= Protect the Entire AI Pipeline.

