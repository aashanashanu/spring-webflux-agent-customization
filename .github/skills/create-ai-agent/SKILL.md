---
name: create-ai-agent
description: Design and implement enterprise-ready AI agents, orchestration flows, and prompt contracts.
---

# Skill: Create AI Agent

## Templates
### Agent contract template
```markdown
# Agent Contract
- Goal
- Inputs
- Tools allowed
- Guardrails
- Output schema
```

### Tool schema template
```json
{
  "name": "lookupCustomer",
  "description": "Fetch customer profile",
  "inputSchema": {
    "type": "object",
    "properties": {
      "customerId": { "type": "string" }
    },
    "required": ["customerId"]
  }
}
```

## Best practices
- Constrain tools by least privilege.
- Enforce deterministic output schemas where possible.
- Keep prompts focused on intent + constraints.

## Implementation guidance
- Single-agent: simple workflows with bounded tools.
- Multi-agent: Orchestrator + specialists with clear handoff context.
- Include memory strategy (session, short-term, long-term) and retrieval policy.
- Integrate governance checks for PII, secrets, and unsafe content.

## Example
- Build incident triage agent using ticket lookup and runbook retrieval tools.

## Validation criteria
- Agent has explicit guardrails and output schema.
- Tool definitions are minimal and purpose-scoped.
- Failure and fallback behavior is documented.
