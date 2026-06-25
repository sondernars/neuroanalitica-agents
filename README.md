# NeuroAnalítica — Multi-Agent Content System

An agentic system for automated clinical content production, lead qualification, and learning loops on Instagram, built with n8n and the Claude API. Designed for **NeuroAnalítica**, an interdisciplinary clinical center integrating neuropsychology and psychoanalysis.

---

## Overview

This system replaces manual weekly content planning and lead triage with a pipeline of 9 specialized AI agents. Each agent has a fixed role, defined inputs, structured JSON outputs, and clear escalation rules to human review when clinical or ethical judgment is required.

**Status: all 9 agents built and validated in a lab/sandbox environment.** Each agent runs correctly end-to-end with test data in n8n and Google Sheets. Production deployment — real Instagram DMs, real publishing, real lead volume — requires the integration step described in [Production Readiness](#production-readiness) below.

---

## Architecture

```
Google Sheets (memory layer)
        │
        ▼
┌─────────────────────────────────────────┐
│           LEVEL 1 — PRODUCTION          │
│                                         │
│  Strategist → Hooks → Scriptwriter      │
│       └──────────────────┘              │
│              Repurposing                │
│              Publisher                  │
└─────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────┐
│         LEVEL 2 — INTERACTION           │
│                                         │
│  Community → Qualifier → CRM Sync       │
└─────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────┐
│          LEVEL 3 — LEARNING             │
│                                         │
│  Analyst → feeds back to Strategist     │
└─────────────────────────────────────────┘
```

**Orchestration layer:** n8n handles all state, chaining agent outputs as inputs to the next node. Claude has no memory between calls — the workflow carries the state.

**Memory layer:** Google Sheets stores metrics, ideas bank, editorial plan, scripts, hooks, repurposed assets, calendar, leads, scoring, CRM records, and weekly reports. Every agent reads from and writes to it.

---

## Agents

| # | Agent | Input | Output | Status |
|---|-------|-------|--------|--------|
| 1 | Strategist | Weekly metrics | Editorial plan (14 pieces, JSON) | ✅ Built & tested |
| 2 | Hooks | Editorial plan | 3 hook variations per piece (A/B/C) | ✅ Built & tested |
| 3 | Scriptwriter | Editorial plan | Full script per piece (hook + body + CTA) | ✅ Built & tested |
| 4 | Repurposing | Master script | Carousel, stories, ad copy | ✅ Built & tested |
| 5 | Publisher | Editorial plan | Scheduled calendar (date + time per piece) | ✅ Built & tested — logic only, not yet wired to Meta API |
| 6 | Community | Incoming DM/comment | Classification + auto-response or human escalation | ✅ Built & tested — logic only, not yet wired to Instagram |
| 7 | Qualifier | Escalated leads | Lead score (0–100), priority | ✅ Built & tested |
| 8 | CRM Sync | Lead + score | Consolidated CRM record with next action | ✅ Built & tested |
| 9 | Analyst | Metrics + CRM records | Weekly report + recommendation to Strategist | ✅ Built & tested |

All 9 agents are complete as n8n workflows with working system prompts, JSON-structured outputs, and error handling. They have been validated with synthetic and semi-real data (one real Instagram post, simulated leads and metrics).

---

## Production Readiness

This section is intentionally explicit: **what works today vs. what's needed for real-world deployment.**

| Component | Lab status | Production requirement |
|---|---|---|
| Content generation (Strategist, Hooks, Scriptwriter, Repurposing) | ✅ Fully functional | None — ready to use as-is |
| Publisher | ✅ Scheduling logic works | Needs Meta Graph API or Buffer integration to actually publish |
| Community | ✅ Classification + escalation logic works | Needs Meta Business API (Instagram DMs/comments webhook) — requires business verification, takes several days |
| Qualifier, CRM Sync, Analyst | ✅ Fully functional | None — ready to use once Community is feeding it real leads |

The decision to build the full logic before connecting live APIs was deliberate: it allows validating the entire decision tree (classification, scoring, escalation rules) without risk to real patients or real conversations, and makes the integration step a matter of swapping one node rather than redesigning the system.

---

## Content Framework

**Editorial rule 80/15/5:**
- 80% reach content (anxiety, relationships, attachment, desire)
- 15% authority content (clinical patterns, neuropsychology, evaluation)
- 5% conversion content (services, process, agenda CTA)

**Content pillars:**
- `anxiety_and_overthinking` — reach
- `relationships_and_attachment` — reach
- `desire_and_sexuality` — reach + differentiation
- `clinical_patterns` — authority
- `services_and_process` — conversion

---

## System Prompts

Each agent receives a specialized system prompt that defines:
- Its fixed role and scope
- What it can and cannot do (no clinical diagnoses, no medical claims)
- Escalation rules to human review
- Exact JSON output schema

System prompts are in Spanish because all content is directed at a Spanish-speaking clinical audience. This is intentional, not a limitation.

See `/prompts/` folder for all system prompts.

---

## Governance & Clinical Constraints

This system operates in a mental health context. Hard limits apply across all agents:

- No clinical diagnoses via DM or automated response
- No guaranteed results or treatment promises
- Immediate escalation to human clinician for: suicidal ideation, crisis, abuse, formal evaluation requests, or any ethically ambiguous message
- No grey-area automations (no scraping, no follow/unfollow bots)
- Content clearly separated from clinical intervention
- The Community agent never auto-responds to anything classified as `riesgo` (risk) — it only logs and escalates

---

## Stack

| Layer | Tool |
|-------|------|
| Orchestration | n8n cloud |
| LLM | Claude Sonnet (Anthropic API) |
| Memory | Google Sheets |
| Scheduling | n8n Schedule Trigger / Manual Trigger |
| Distribution | Buffer / Meta API (pending integration) |

---

## Cost

Running the full 9-agent system weekly costs approximately **$2–6 USD/month** using Claude Sonnet via the Anthropic API at current testing volume. Real production volume (active DM traffic) would add a modest amount depending on message count — still well under $20/month at small-clinic scale.

---

## Market Reference

For context on what a system of this scope would cost as a commissioned project: comparable n8n + LLM agentic systems for clinics and SMEs in the Spanish-speaking market are commonly quoted between **€1,500–4,000** (or ~2,000,000–4,500,000 CLP) as a one-time implementation project, with monthly maintenance/operation in the €50–400 range — independent of the underlying API costs, which remain low (see above).

---

## Design Decisions

**Why 9 specialized agents instead of one general agent?**
Discrete agents with fixed roles produce more consistent, auditable outputs. A single general agent would require complex prompting, produce less predictable results, and make debugging difficult. Separation also allows independent execution, testing, and iteration of each layer.

**Why Google Sheets as the memory layer?**
Sheets provides a human-readable, easily editable shared state that non-technical team members (clinicians, editors) can review and modify without touching the automation layer. It also acts as an audit trail.

**Why escalate to human for clinical cases?**
Automation in mental health without governance is dangerous. The system is designed to accelerate content production and lead qualification — not to replace clinical judgment. Any message involving risk, formal evaluation, or ethical ambiguity bypasses automation entirely.

**Why build all 9 agents in a lab environment before connecting live APIs?**
Connecting to Meta's Instagram API requires business verification (multi-day process) and carries real risk if the classification/escalation logic has bugs — a missed crisis escalation in production is unacceptable. Validating the full decision tree with synthetic data first, then swapping in live data sources, isolates integration risk from logic risk.

**Why Claude over other LLMs?**
Claude's instruction-following precision and its ability to maintain consistent tone, format, and clinical constraints across structured JSON outputs makes it well-suited for this type of agentic pipeline.

---

## Author

Built by Sebastián Arismendi — Psychologist & MSc in Neuroscience (in progress), transitioning into AI Engineering. This project combines clinical domain expertise with applied LLM orchestration and agentic systems design.
