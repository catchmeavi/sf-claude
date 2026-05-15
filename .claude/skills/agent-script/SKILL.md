---
name: agent-script
description: Scaffold a new Salesforce Agentforce Agent Script (.agent file) following the trailheadapps/agent-script-recipes standards. Use when the user wants to create a new agent, add a subagent, define agent actions, or generate an agent recipe.
argument-hint: [agent-name]
arguments: agent_name
---

## Task

You are an expert in Salesforce Agentforce Agent Script syntax. Scaffold a production-ready `.agent` file for **$agent_name** following the strict rules below, sourced from the official trailheadapps/agent-script-recipes `.airules/AGENT_SCRIPT.md`.

---

## Discovery (ask these if not already provided)

Before generating, confirm:
1. **Identity** — What is the agent's name, role, and primary goal?
2. **Subagents** — Which subagent topics does it need (e.g., greeting, escalation, order lookup)?
3. **Actions** — Does it call Flows, Apex classes, Prompt Templates, or APIs?
4. **Variables** — What state does it need to track across turns?
5. **Reasoning** — Should it use LLM reasoning, deterministic directives, or both?

If any answer is missing, ask before generating the file.

---

## File Structure Rules

Generate blocks in this **exact order** — never reorder:

```
config → variables → system → connections → knowledge → language → start_agent → subagents
```

Omit blocks that are not needed (e.g., no `connections` if no external integrations).

---

## Block Reference

### `config` block (required)
```agentscript
config {
  name: "<AgentName>"                  // letters, numbers, underscores only; max 80 chars
  description: "<one sentence>"
  agent_type: CUSTOMER_FACING          // or INTERNAL
  default_language: en_US
}
```

### `variables` block (when state is needed)
```agentscript
variables {
  mutable <type> <name>                // agent can read/write
  linked <type> <name>                 // read-only from external context
}
```
Supported types: `Text`, `Number`, `Boolean`, `Record`, `List<Record>`, `List<Text>`

### `system` block (required)
```agentscript
system {
  prompt: """
    You are <role>.
    <Persona and behavioral instructions.>
    Always be concise and professional.
  """
}
```

### `subagents` block structure
Each subagent follows this pattern:
```agentscript
subagent <SubagentName> {
  description: "<when this subagent handles the conversation>"
  availability: always                 // or: if (<condition>)

  actions {
    // reasoning OR directive actions here
  }

  directive {
    transition to <NextSubagent>       // or: end
  }
}
```

### Reasoning actions (LLM-driven)
```agentscript
reasoning {
  model: sfdc_ai__DefaultAgentModel
  prompt: """<instructions>"""
  inputs {
    <varName>: <expression>
  }
  post_action {
    on success transition to <Subagent>
    on failure transition to <FallbackSubagent>
  }
}
```

### Apex actions
```agentscript
action {
  type: apex
  class_name: <ClassName>
  method_name: <methodName>
  inputs {
    <paramName>: <expression>
  }
  outputs {
    <outputVar>: result.<fieldName>
  }
}
```

### Flow actions
```agentscript
action {
  type: flow
  flow_name: <FlowApiName>
  inputs {
    <paramName>: <expression>
  }
  outputs {
    <outputVar>: result.<fieldName>
  }
}
```

### Utility actions
```agentscript
// Escalate to human
action { type: utility; name: @utils.escalate }

// Transition to another subagent
action { type: utility; name: @utils.transition; inputs { target: "<SubagentName>" } }

// Set variables
action { type: utility; name: @utils.setVariables; inputs { <varName>: <expression> } }
```

---

## Naming Conventions

- Block names, variable names, subagent names: **letters, numbers, underscores only** — no spaces, no hyphens
- Subagent names: PascalCase (e.g., `GreetUser`, `LookupOrder`, `EscalateToHuman`)
- Variables: camelCase (e.g., `customerId`, `orderStatus`)
- Max name length: 80 characters

---

## Control Flow Rules

- In `reasoning.post_action`: use `on success transition to <Name>` / `on failure transition to <Name>`
- In `directive` blocks: use `transition to <Name>` or `end`
- Never mix the two styles in the same block
- The `start_agent` block must name the first subagent: `start_agent: <SubagentName>`

---

## Template & Expression Syntax

- Variable references: `{variables.<varName>}`
- Linked context: `{context.<fieldName>}`
- String concat: `{variables.firstName + " " + variables.lastName}`
- Conditional: `{variables.isVip ? "Priority" : "Standard"}`
- Null check: `{variables.orderId != null}`

---

## Validation Checklist

Before outputting the file, verify:
- [ ] Blocks are in correct order: `config → variables → system → ... → subagents`
- [ ] `start_agent` names a subagent that is actually defined
- [ ] All `transition to <X>` targets refer to a defined subagent or `end`
- [ ] No SOQL/DML referenced directly in the agent script (delegate to Apex/Flow)
- [ ] All variable names use only letters, numbers, underscores
- [ ] Every subagent has either a `directive` or a `reasoning.post_action` — not both at the top level
- [ ] `config.agent_type` is set to `CUSTOMER_FACING` or `INTERNAL`

---

## Common Mistakes to Avoid

1. **Wrong transition syntax** — `directive { transition: <X> }` is wrong; use `transition to <X>`
2. **Reordering blocks** — always keep the canonical order
3. **Hyphens in names** — `my-agent` is invalid; use `my_agent`
4. **Missing `start_agent`** — every agent file needs this
5. **LLM reasoning in directive blocks** — directives are deterministic; move LLM calls to `reasoning` actions
6. **Hardcoded IDs** — use variables or linked context, never hardcoded Salesforce record IDs

---

## Output

Generate:
1. The complete `$agent_name.agent` file with proper syntax
2. A brief explanation (2-3 bullets) of the key design decisions made
3. A note on any Apex classes or Flows that would need to be created separately

Use `agentscript` code fences for the file output.
