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

# 3\. AI Attack Surface

Prompt Layer → Context Layer → Retrieval Layer → Model Layer → Tool Layer → Memory Layer → Output Layer

| Layer | Primary Risks |
| :---- | :---- |
| Prompt | Prompt Injection, Jailbreaks |
| Context | Context Poisoning |
| Retrieval | Unauthorized Access |
| Model | Data Leakage, Model Extraction |
| Tools | Tool Abuse |
| Memory | Memory Poisoning |
| Output | Sensitive Data Exposure |

---

# 4\. Prompt Injection & Jailbreaks

## Prompt Injection

Attempts to override intended instructions.

### Direct Prompt Injection

Attacker directly provides malicious instructions.

### Indirect Prompt Injection

Malicious instructions are hidden inside retrieved content.

Document → Retriever → LLM

The document becomes the attack vector.

## Prompt Leakage

Attempts to expose hidden system prompts.

## Data Exfiltration

Attempts to retrieve unauthorized information.

## Jailbreaks

Attempts to bypass model safety controls.

### Key Principle

**Prompts are not security controls.**

---

# 5\. Enterprise RAG Security

## Architect Insight: Retrieval Security Is Most Important

Retrieval security is the most important enterprise AI security topic.

Because: Enterprise knowledge \= Enterprise risk.

Most AI security incidents stem from retrieval failures, not prompt attacks or model issues.

## Golden Rule

Users should never gain access through AI to information they could not access without AI.

## Secure Retrieval Flow

User → Authentication → Authorization → Retriever → Authorized Documents → LLM

### Critical Principle

Authorization BEFORE Retrieval

## Core Controls

- Retrieval Authorization  
- Document-Level Security  
- Row-Level Security  
- Metadata Security Filtering  
- Retrieval Auditability

---

# 6\. Tenant Isolation

Authentication \= Who are you?

Authorization \= What can you access?

Tenant Isolation \= Whose data can you access?

## Isolation Models

| Model | Isolation | Cost | Typical Use |
| :---- | :---- | :---- | :---- |
| Shared Collection | Low | Low | Startup |
| Collection Per Tenant | Medium | Medium | Enterprise |
| Database Per Tenant | High | High | Defence/Government |

## Cross-Tenant Leakage

NAB User → Retrieves → ANZ Documents

Major security incident.

## Tenant Context Propagation

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

# 7\. LLM Security

For most enterprises:

RAG Security \> LLM Security

## Major Risks

- Model Theft  
- Model Extraction  
- Training Data Leakage  
- Unsafe Fine-Tuning  
- Open Source Model Risks

Key Questions:

- Where did it come from?  
- Can it be trusted?  
- Who maintains it?

---

# 8\. Agent Security

Traditional LLM:

Wrong Answer

Agent:

Wrong Action

## Major Risks

- Excessive Permissions  
- Tool Abuse  
- Memory Risks  
- Autonomous Actions

## Least Privilege

Agents should receive only minimum required permissions.

## Human-in-the-Loop

Agent → Approval → Execution

## Agent Permission Levels

| Level | Capability |
| :---- | :---- |
| Read | View Information |
| Recommend | Suggest Actions |
| Draft | Prepare Actions |
| Execute | Perform Actions |
| Autonomous | Execute Without Approval |

---

# 9\. MCP Security

## Mental Model

RAG \= Document Discovery

MCP \= Tool Discovery

RAG requires Document Authorization.

MCP requires Tool Authorization.

## Major Risks

- Tool Discovery  
- Tool Authorization  
- Tool Isolation  
- Data Leakage

Core Principle:

An agent should only discover and use authorized tools.

---

# 10\. Defense-in-Depth

User → Input Controls → Authorization → Retriever → LLM → Tool Controls → Output Controls

If one control fails, another control catches it.

---

# 11\. Threat vs Control Matrix

| Threat | Primary Control |
| :---- | :---- |
| Prompt Injection | Input Controls |
| Jailbreak | Safety Controls |
| Unauthorized Retrieval | Authorization |
| Sensitive Documents | Metadata Filtering |
| Cross-Tenant Leakage | Tenant Isolation |
| Tool Abuse | Least Privilege |
| Dangerous Actions | Human Approval |
| Data Leakage | Output Controls |
| Training Data Leakage | Model Governance |

---

# 12\. Architecture Review Checklist

## Identity

- How are users authenticated?  
- How is authorization enforced?

## Retrieval

- Can unauthorized documents be retrieved?  
- Are metadata filters enforced?

## Tenant Isolation

- Where is tenant context established?  
- How is tenant context propagated through retrieval, agents, and tools?  
- Can retrieval cross tenant boundaries?  
- Can agents cross tenant boundaries?  
- What happens if tenant filtering fails?

## Agents

- Which tools can be accessed?  
- Which actions require approval?

## MCP

- Which tools can be discovered?  
- How is tool authorization enforced?

## Outputs

- Is sensitive information protected?  
- Are outputs validated?

---

# 13\. Key Takeaways

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

