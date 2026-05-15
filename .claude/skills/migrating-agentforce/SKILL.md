---
name: migrating-agentforce
description: Migrates legacy Salesforce agent metadata (Bot, BotVersion, GenAiFunction, GenAiPlannerBundle, GenAiPromptTemplate, GenAiPlugin) to modern Agent Script DSL (.agent files). Starts in PLAN mode to document the migration strategy.
trigger: when the user asks to migrate, upgrade, or convert legacy bots, plugins, planner bundles, or prompt templates into the new Agentforce Agent Script format, or mentions migrating to .agent.
do_not_trigger: when the user wants to create a net-new agent (use developing-agentforce), test an agent (use testing-agentforce), or analyze production traces (use observing-agentforce).
---

# Agentforce Migration Skill

This skill is responsible for analyzing legacy Salesforce Agent metadata and generating modern Agent Script (`.agent`) files. Agent Script (introduced in Spring '26) is a TypeScript-based hybrid reasoning framework that replaces previous "pure prompt-driven" configurations, offering deterministic guardrails combined with LLM reasoning.

## 1. Context & Definitions

- **Legacy Metadata Components:**
  - `Bot` & `BotVersion`: Traditional Einstein Bot configurations and foundational metadata.
  - `GenAiPlannerBundle`: Legacy routing and topic classification mechanisms.
  - `GenAiPlugin` & `GenAiFunction`: Legacy wrappers for exposing Apex/Flow tools to the AI.
  - `GenAiPromptTemplate` & `GenAiPromptTemplateActv`: Legacy prompt-based reasoning instructions.
- **Modern Target (Agent Script):**
  - `.agent` file: A single-file, indentation-sensitive language (YAML/Python-like) for agent behavior.
  - `subagents` (formerly topics): Domain-specific handling blocks.
  - `actions`: Tool invocations natively defined inside the agent script.
  - `reasoning`: Execution blocks utilizing `|` for LLM handoffs and `->` for deterministic transitions.

## 2. Migration Workflow

### Phase 1: PLAN Mode - Analysis & Documentation

1.  **Initialize PLAN Mode:** Upon triggering, immediately enter PLAN mode before writing any script code.
2.  **Scan & Assess:** Search the user's Salesforce DX project (`force-app/main/default/`) for legacy AI metadata files (`/bots/`, `/genAiPlannerBundles/`, `/genAiPlugins/`, `/genAiFunctions/`, `/genAiPromptTemplates/`).
3.  **Draft Migration Strategy:** Create a comprehensive document mapping the legacy components to their new `.agent` structures (System Config, Subagents, Actions, and Reasoning).
4.  **Visual Subagent Map:** Generate a **mermaid.js** flowchart diagram inside the migration document to visualize the subagent routing map and action associations. The diagram should clearly map the router/start node to corresponding subagents (e.g., `ambiguous_question`, `FAQ_Assistance`, `Apartment_Search`, `Lead_Creation`, `Escalation`) based on conditions like "unclear intent", "out of scope", or "business hours check".
5.  **Save Plan:** Save this plan in the local workspace inside a `doc` folder using the exact naming convention: `doc/{AGENTNAME}_MIGRATION.md` (replacing `{AGENTNAME}` with the actual name of the Bot/Agent).
6.  Wait for user approval on the plan before proceeding to code generation.

### Phase 2: Discovery & Extraction

1.  Parse the legacy XML files identified in the plan.
2.  Extract conversational intents, tool target mappings, prompt templates, variables, and routing logic.

### Phase 3: Metadata Mapping Rules

Translate the extracted legacy XML into Agent Script architecture:

- **Bot/BotVersion ➡️ System & Root Config**
  - Extract the main bot description and greeting message.
  - Map to the `system:` block and `messages: welcome:` node in the `.agent` file.
- **GenAiPlannerBundle ➡️ Subagents**
  - Map legacy planner nodes into `subagents:`.
  - Convert classification logic into subagent descriptions and `reasoning:` rules.
- **GenAiFunction / GenAiPlugin ➡️ Actions**
  - Translate plugin definitions into `actions:` arrays assigned to the relevant subagent.
  - Verify the integration target (e.g., `Apex.MyCustomClass` or `Flow.My_Auto_Flow`).
- **GenAiPromptTemplate / Actv ➡️ Reasoning Instructions**
  - Migrate open-ended prompts into Agent Script's `reasoning:` logic.
  - Convert prompt-based business rules into Deterministic Logic (e.g., `if order_age > 30:`).
  - Retain fuzzy intent matching using the LLM Handoff operator (`| "Determine the user's intent"`).

### Phase 4: Generation & Formatting

1.  **Utilize Reference Repositories:** Use the provided reference links (e.g., Agentforce ADLC, AFV library, and Salesforce Developer Docs) to thoroughly understand the modern Agent Script structure, capabilities, and syntax limitations before authoring.
2.  **Invoke the Generator:** Explicitly use the `agent-script` skill to generate the `.agent` file in the `aiAuthoringBundles/<DeveloperName>/` directory.
3.  Enforce strict Agent Script syntax during generation:
    - **Indentation:** Must use exactly 4 spaces (no tabs, as tabs break the Agent Script compiler).
    - **Booleans:** Python-style capitalization (`True` / `False`).
    - **Variables:** Explicitly define them within the script scope.
    - **Control Flow:** Utilize `before_reasoning:` or `after_reasoning:` blocks for setting state before the LLM loop begins or after it exits.

### Phase 5: Validation & Cleanup

1.  Output the completed Agent Script to the user.
2.  Run structural validation by cross-referencing `actions:` with the existing project codebase to ensure no broken references.
3.  Suggest running the migration validation command: `sf agent validate authoring-bundle --api-name <AGENT_API_NAME>`.
4.  Update the `doc/{AGENTNAME}_MIGRATION.md` file with the final status and offer to delete the deprecated `GenAi*` and `Bot*` XML files.

## 3. Reference Output Example

When generating the `.agent` file, output structural format similar to this:

```agentscript
system:
    instructions: "You are a helpful banking assistant."
    messages:
        welcome: "Welcome! How can I help you today?"
        error: "I encountered an error processing that request."

subagents:
    AccountManagement:
        description: "Handles account balance and detail requests."
        instructions: "Never expose full account numbers."
        actions:
            GetBalance:
                target: "Apex.AccountService"
                description: "Retrieves the current balance for an authenticated user."
        reasoning:
            before_reasoning:
                if session.isAuthenticated == False:
                    -> AuthenticationSubagent
            instructions:
                | "Understand if the user is asking for checking or savings balance, then use the GetBalance tool."
```
