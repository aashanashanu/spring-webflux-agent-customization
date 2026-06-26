---
name: create-mcp-tool
description: Define and integrate MCP tools for enterprise and AI-agent workflows.
---

# Skill: Create MCP Tool

## Templates
### MCP tool definition
```json
{
  "name": "searchPolicies",
  "description": "Search policy documents by keyword",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": { "type": "string" },
      "limit": { "type": "integer", "minimum": 1, "maximum": 50 }
    },
    "required": ["query"]
  }
}
```

## Best practices
- Stable schema versioning.
- Strict validation for input and output.
- Explicit timeout and error contracts.

## Implementation guidance
- Add authentication and authorization boundaries for each tool.
- Keep tool responses concise and structured.
- Include rate-limit and audit considerations.

## Example
- Build MCP tool to fetch order details from internal API gateway.

## Validation criteria
- Schema is valid and bounded.
- Error contract documented.
- Security scope defined per tool.
