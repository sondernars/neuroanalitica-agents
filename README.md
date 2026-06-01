# neuroanalitica-agents
Multi-agent system for automated clinical content production on Instagram, built with n8n and Claude API
# NeuroAnalítica — Multi-Agent Content System

An agentic system for automated clinical content production on Instagram, built with n8n and the Claude API. Designed for **NeuroAnalítica**, an interdisciplinary clinical center integrating neuropsychology and psychoanalysis.

---

## Overview

This system replaces manual weekly content planning with a pipeline of specialized AI agents. Each Monday at 9am, the system automatically produces a full editorial plan, complete scripts, and hook variations for 14 Instagram pieces — without human intervention at the production stage.

The architecture separates strategy, writing, and optimization into discrete agents. Each agent has a fixed role, defined inputs, structured JSON outputs, and clear escalation rules to human review when clinical or ethical judgment is required.

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

**Memory layer:** Google Sheets stores metrics, ideas bank, editorial plan, scripts, and hooks. Every agent reads from and writes to it.

---

## Agents

| # | Agent | Input | Output | Status |
|---|-------|-------|--------|--------|
| 1 | Strategist | Weekly metrics | Editorial plan (14 pieces, JSON) | ✅ Built |
| 2 | Scriptwriter | Editorial plan | Full script per piece (hook + body + CTA) | ✅ Built |
| 3 | Hooks | Editorial plan | 3 hook variations per piece (A/B/C) | ✅ Built |
| 4 | Repurposing | Master reel | Carousel, stories, ad copy | ✅ Built |
| 5 | Publisher | Approved assets | Scheduled calendar | ✅ Built |
| 6 | Community | DMs + comments | Classified responses, escalations | ✅ Built |
| 7 | Qualifier | Incoming messages | Lead score, intent, priority | 🔜 Planned |
| 8 | CRM Sync | Qualified leads | CRM record + follow-up tasks | 🔜 Planned |
| 9 | Analyst | Metrics + leads + closes | Weekly report + recommendations | 🔜 Planned |

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

---

## Stack

| Layer | Tool |
|-------|------|
| Orchestration | n8n cloud |
| LLM | Claude Sonnet (Anthropic API) |
| Memory | Google Sheets |
| Scheduling | n8n Schedule Trigger |
| Distribution | Buffer / Meta API (planned) |

---

## Cost

Running the full 9-agent system weekly costs approximately **$2–6 USD/month** using Claude Sonnet via the Anthropic API, depending on DM volume.

---

## Design Decisions

**Why 9 specialized agents instead of one general agent?**
Discrete agents with fixed roles produce more consistent, auditable outputs. A single general agent would require complex prompting, produce less predictable results, and make debugging difficult. Separation also allows independent execution, testing, and iteration of each layer.

**Why Google Sheets as the memory layer?**
Sheets provides a human-readable, easily editable shared state that non-technical team members (clinicians, editors) can review and modify without touching the automation layer. It also acts as an audit trail.

**Why escalate to human for clinical cases?**
Automation in mental health without governance is dangerous. The system is designed to accelerate content production and lead qualification — not to replace clinical judgment. Any message involving risk, formal evaluation, or ethical ambiguity bypasses automation entirely.

**Why Claude over other LLMs?**
Claude's instruction-following precision and its ability to maintain consistent tone, format, and clinical constraints across structured JSON outputs makes it well-suited for this type of agentic pipeline.

---

## Author

Built by Sebastián Arismendi, Psychologist & MSc in Neuroscience — AI systems, automation architecture, and clinical content strategy for NeuroAnalítica.
