# Salesforce + Claude Code — Reference Project

This repository is a **reference template** for how a Salesforce DX project should be structured when using Claude Code as an AI development partner. It shows how to configure Claude's persona, pre-built skills, and slash commands to accelerate the full delivery lifecycle: **Plan → Build → Test → Deploy**.

---

## Why This Structure Exists

Claude Code is most effective when it has context — who it is, what the project does, which rules apply, and which tools it can use. This repo externalises all of that configuration into three layers:

| Layer              | File/Folder                   | Purpose                                                          |
| ------------------ | ----------------------------- | ---------------------------------------------------------------- |
| Identity & rules   | `CLAUDE.md`                   | Persona, coding standards, architectural principles              |
| Project settings   | `.claude/settings.json`       | Shared permissions, allowed/denied commands, env vars            |
| Developer settings | `.claude/settings.local.json` | Personal org aliases, local-only tool permissions _(gitignored)_ |
| Skills             | `.claude/skills/`             | Deep domain logic Claude loads on demand                         |
| Commands           | `.claude/command/`            | Slash commands for repeatable delivery tasks                     |
| AI rules           | `.airules/`                   | Additional rule files (Apex, Mermaid, README standards)          |

---

## Project Structure

```text
my-salesforce-project/
├── .claude/
│   ├── settings.json               # Shared: allowed commands, MCP permissions, env vars
│   ├── settings.local.json         # Local only (gitignored): your org alias, personal perms
│   ├── rules/                      # Supplemental rule sets read by skills
│       ├── APEX_RULES.md           # Hard rules for Apex (governor limits, security)
│       ├── MERMAID_DIAGRAMS.md     # Diagram standards
│       ├── AGENT_SCRIPT.md         # Agent Script recipe conventions
│       └── README_STRUCTURE.md     # README authoring standards
│   ├── command/                    # Slash commands — invoke with /command-name
│   │   ├── sf-agentforce.md        # Agentforce technical design document
│   │   ├── sf-agent-topic.md       # Agent topic + action specification
│   │   ├── sf-agent-prompt.md      # Prompt Builder template with merge field mapping
│   │   ├── sf-apex-review.md       # Apex code review — limits, CRUD/FLS, bulkification
│   │   ├── sf-apex-test.md         # Generate Apex test classes (arrange/act/assert)
│   │   ├── sf-data-model.md        # Data model schema with ERD and DMO mapping
│   │   ├── sf-deploy.md            # Deployment runbook with rollback plan
│   │   ├── sf-flow.md              # Flow design document with fault paths
│   │   ├── sf-integration.md       # Integration design — REST, Platform Events, auth
│   │   ├── sf-security.md          # Security model — OWD, permission sets, sharing rules
│   │   └── sf-soql-query.md        # SOQL generation with selectivity analysis
│   └── skills/                     # Domain skills — Claude loads these automatically
│       ├── generating-apex/        # Apex classes, triggers, batch, queueable, REST
│       ├── generating-apex-test/   # Apex test classes, mocks, data factories
│       ├── generating-lwc-components/ # LWC bundles, wire service, Jest tests
│       ├── generating-flow/        # Salesforce Flows
│       ├── deploying-metadata/     # sf project deploy workflows
│       ├── running-apex-tests/     # Test execution and coverage loops
│       ├── apex-lint/              # Apex static analysis against project rules
│       ├── apex-recipe/            # Agent-script recipe authoring
│       ├── debugging-apex-logs/    # Apex debug log analysis
│       ├── developing-agentforce/  # Agentforce agent authoring (.agent files)
│       ├── testing-agentforce/     # Agentforce test suites (AiEvaluationDefinition)
│       ├── generating-mermaid-diagrams/ # Architecture diagrams
│       ├── querying-soql/          # SOQL query building
│       ├── handling-sf-data/       # Data operations (import/export/upsert)
│       └── ...                     # Full skill list in .claude/skills/
├── .vscode/
│   └── settings.json               # Suppresses Apex LS errors in template files
├── config/
│   └── project-scratch-def.json    # Scratch org feature flags
├── force-app/
│   └── main/default/
│       ├── agents/                 # Agentforce agent metadata
│       ├── classes/                # Apex classes and triggers
│       ├── flows/                  # Salesforce Flows
│       ├── genAiFunctions/         # Generative AI functions
│       ├── genAiPromptTemplates/   # Prompt Builder templates
│       ├── lwc/                    # Lightning Web Components
│       └── ...
├── docs/                           # Architecture decisions, design docs (Claude output)
├── .mcp.json                       # MCP server configuration (Salesforce CLI MCP)
├── .forceignore                    # Prevents Apex LS from scanning template dirs
├── sfdx-project.json               # SFDX project manifest
├── CLAUDE.md                       # Claude identity, persona, and coding standards
└── CLAUDE.local.md                 # Local developer notes (gitignored)
```

---

## CLAUDE.md — Teaching Claude Who It Is

`CLAUDE.md` is the most important file in the project. Claude reads it at the start of every session. It defines:

- **Role** — Senior Salesforce CTA and Lead Developer
- **Architectural principles** — Governor limits, declarative-first, security-first, loose coupling
- **Coding standards** — Bulkification, trigger frameworks, SLDS, wire service patterns
- **Response style** — Always provide trade-offs, always ask about data volumes if unclear

When you open a new Claude Code session in this directory, it already behaves as a Salesforce architect without any prompting.

**To customise for your project**, edit `CLAUDE.md` to add:

- Your project name and current sprint
- Naming conventions and class prefixes
- Specific packages in use (e.g. fflib, TAF)
- Any org constraints (edition, governor limit proximity)

---

## Skills — Deep Domain Logic

Skills are markdown files in `.claude/skills/` that contain detailed rules, workflows, templates, and output formats for a specific domain. Claude loads them automatically based on what you ask.

### How skills activate

You do not invoke skills manually. Claude detects the intent and loads the right skill:

| If you ask about...            | Claude loads                  |
| ------------------------------ | ----------------------------- |
| "Write an Apex service class"  | `generating-apex`             |
| "Write a test class for this"  | `generating-apex-test`        |
| "Build an LWC component"       | `generating-lwc-components`   |
| "Create a Flow"                | `generating-flow`             |
| "Deploy this metadata"         | `deploying-metadata`          |
| "Lint this Apex class"         | `apex-lint`                   |
| "Debug this Apex log"          | `debugging-apex-logs`         |
| "Build an Agentforce agent"    | `developing-agentforce`       |
| "Write a SOQL query"           | `querying-soql`               |
| "Draw an architecture diagram" | `generating-mermaid-diagrams` |

### What a skill does

Each skill enforces a full workflow. For example, `generating-apex` will:

1. Discover your project naming conventions
2. Choose the right class pattern (Service, Selector, Batch, Queueable, etc.)
3. Generate the `.cls` and `.cls-meta.xml` files
4. Automatically load `generating-apex-test` to generate a test class
5. Run the code analyzer (`sf code-analyzer run`) against the output
6. Execute the Apex tests and capture pass/fail and coverage
7. Produce a structured report with analyzer output and test results

Skills are self-contained — they know their own rules, anti-patterns, and output formats.

### Adding a new skill

Create a folder in `.claude/skills/your-skill-name/` with a `SKILL.md` file. Use the frontmatter format:

```markdown
---
name: your-skill-name
description: "One-line description of when Claude should load this skill."
---

## Workflow

...
```

---

## Commands — Repeatable Delivery Tasks

Commands are slash commands that produce structured documents for specific delivery artifacts. Invoke them in any Claude Code chat with `/command-name`.

### Available commands

| Command            | What it produces                                                          |
| ------------------ | ------------------------------------------------------------------------- |
| `/sf-agentforce`   | Full Agentforce technical design document (7 sections, client-ready)      |
| `/sf-agent-topic`  | Agent topic + action specification for Einstein Copilot Studio            |
| `/sf-agent-prompt` | Prompt Builder template with merge field mapping and token budget         |
| `/sf-apex-review`  | Apex code review — governor limits, CRUD/FLS, bulkification findings      |
| `/sf-apex-test`    | Production-ready Apex test class with arrange/act/assert structure        |
| `/sf-data-model`   | Data model schema with ERD, field definitions, and Data Cloud DMO mapping |
| `/sf-deploy`       | Deployment runbook with pre-deploy checklist and rollback plan            |
| `/sf-flow`         | Flow design document with fault paths and governor limit notes            |
| `/sf-security`     | Security model — OWD, permission sets, sharing rules                      |
| `/sf-integration`  | Integration design — REST/Platform Events, auth, error handling           |
| `/sf-soql-query`   | Optimised SOQL with selectivity and index analysis                        |

### How to use a command

Commands use `{{variables}}` for the values that change per invocation. Provide them inline:

```
/sf-apex-review

{{apex_code}}
public class LeadQualificationService {
    public static void qualifyLeads(List<Lead> leads) {
        for (Lead l : leads) {
            update l;   // <- this is the code to review
        }
    }
}

{{context}}
Service class called from LeadTrigger on before update.

{{api_version}}
v62.0
```

All document-producing commands save output to the `docs/` folder automatically.

### Adding a new command

Create a `.md` file in `.claude/command/`. Follow the five-section structure used by every existing command:

```markdown
## Role/Persona

## Context

## Output Format

## Constraints

## Variables
```

Use `{{double_curly_braces}}` for every value that changes per invocation.

---

## Plan → Build → Test → Deploy with Claude

This is the recommended workflow for delivering a Salesforce feature end-to-end using Claude Code.

---

### 1. Plan

Use Claude to produce formal design documents before writing any code.

**Data model first:**

```
/sf-data-model

{{business_domain}}
SDR lead qualification with BANT scoring and Agentforce automation

{{functional_requirements}}
- Store BANT fields on Lead
- Track agent conversation sessions linked to Lead
- Support bulk scoring via nightly batch

{{data_cloud_in_scope}}
Yes

{{existing_objects}}
Lead, Contact, Account, Task
```

**Then architecture:**

```
/sf-agentforce

{{requirement}}
Build an Agentforce SDR agent that qualifies leads using BANT scoring,
books meetings via Einstein Scheduling, and updates Salesforce in real time.

{{org_edition}}
Enterprise + Agentforce 1 + Data Cloud Growth

{{audience}}
Internal delivery team
```

Claude saves both documents to `docs/` and uses them as context for every subsequent step.

---

### 2. Build

With the design docs in place, ask Claude to build. Skills activate automatically.

**Apex service layer:**

```
Write a LeadQualificationService class that scores leads against BANT criteria.
It will be called from a Queueable and must handle 200+ records.
Use the data model we just designed.
```

Claude will:

- Load `generating-apex` skill
- Apply all governor limit, security, and bulkification rules from `CLAUDE.md` and `.airules/APEX_RULES.md`
- Generate `LeadQualificationService.cls` + meta XML
- Automatically generate `LeadQualificationServiceTest.cls`

**LWC component:**

```
Build an LWC component that displays a lead's BANT score with a progress bar.
It should use a wire adapter to load the lead and fire a custom event when the score changes.
```

Claude loads `generating-lwc-components`, enforces SLDS and accessibility rules, and produces the full bundle including a Jest test.

**Flow:**

```
/sf-flow

Design a Screen Flow that walks an SDR through manual BANT qualification
with branching logic based on score thresholds.
```

---

### 3. Test

Run tests through Claude rather than switching to the terminal.

**Generate test class for existing code:**

```
/sf-apex-test

{{apex_code}}
[paste LeadQualificationService.cls here]

{{scenarios}}
lead already converted, null BANT fields, batch of exactly 200 records, callout timeout

{{api_version}}
v62.0
```

**Lint Apex before committing:**

```
/apex-lint LeadQualificationService
```

Claude produces a table of CRITICAL / HIGH / MEDIUM issues with line numbers and fix suggestions.

**Run tests against the org:**

```
Run all tests in the LeadQualification namespace and show me coverage.
```

Claude executes `sf apex run test`, captures results, and flags any classes below 75% coverage.

---

### 4. Deploy

Generate a runbook, then deploy.

**Deployment runbook:**

```
/sf-deploy

{{components_to_deploy}}
- ApexClass: LeadQualificationService
- ApexClass: LeadQualificationServiceTest
- LightningComponentBundle: leadBANTScoreCard
- Flow: BANT_Qualification_Screen_Flow
- PermissionSet: SDR_Agent_Access

{{target_org}}
Full Sandbox — UAT

{{deployment_method}}
Salesforce CLI — sf project deploy start

{{deployment_window}}
2026-05-20 21:00–22:30 AEST
```

**Execute the deployment:**

```
Deploy the LeadQualification components to UAT. Run the pre-deploy checklist first.
```

Claude runs `sf project deploy start --dry-run`, checks the output, and then executes the real deployment.

---

## MCP Configuration — Sandbox vs Production URLs

The Salesforce hosted MCP servers use different base URLs depending on which org type you are connecting to:

| Org type                       | Base URL                          |
| ------------------------------ | --------------------------------- |
| Production / Developer Edition | `https://api.salesforce.com`      |
| Sandbox / Scratch org          | `https://test.api.salesforce.com` |

`.mcp.json` is **gitignored** in this project. This is intentional — the URL is environment-specific and must not be overwritten by a `git pull`.

### First-time setup

Copy the template to create your local config:

```bash
cp .mcp.json.example .mcp.json
```

Then open `.mcp.json` and set the `url` field for each hosted MCP server to match your org type:

**Sandbox / Scratch org:**

```json
"url": "https://test.api.salesforce.com/platform/mcp/v1-beta.2/platform/salesforce-api-context"
```

**Production / Developer Edition:**

```json
"url": "https://api.salesforce.com/platform/mcp/v1-beta.2/platform/salesforce-api-context"
```

Apply the same swap to the `metadata-experts` entry.

### Why changes kept reverting

`.mcp.json` was previously tracked by git (the `.gitignore` entry said `mcp.json` without the dot prefix, so it was silently ignored). Every `git pull` overwrote the local file with the production URL from the repo. The file is now correctly gitignored and untracked. Your local `.mcp.json` will never be touched by git again.

### Switching orgs mid-project

When you switch from sandbox to production (or vice versa), edit `.mcp.json` directly — the change is local-only and will not affect other developers. The `_sandboxUrl` and `_note` fields in `.mcp.json.example` serve as a reminder of both values.

---

## Settings Reference

### `.claude/settings.json` (shared, checked in)

```json
{
  "permissions": {
    "allow": [
      "Bash(sf apex run test*)",
      "Bash(sf project deploy start*)",
      "Read(**)"
    ],
    "deny": ["Bash(git push --force*)", "Bash(rm -rf*)", "Bash(sf org delete*)"]
  },
  "env": {
    "SF_API_VERSION": "62.0"
  }
}
```

Pre-approves safe read and test commands. Permanently blocks destructive operations regardless of what is asked.

### `.claude/settings.local.json` (local only, gitignored)

```json
{
  "permissions": {
    "allow": [
      "Bash(sf org open*)",
      "Bash(sf org create scratch*)",
      "mcp__*__deploy_metadata"
    ]
  },
  "env": {
    "SF_TARGET_ORG": "dev-scratch",
    "SF_TARGET_DEV_HUB": "YourDevHubAlias"
  }
}
```

Personal org aliases and local-only permissions. Each developer maintains their own copy.

---

## Quick Reference

| Goal                                              | How                                                            |
| ------------------------------------------------- | -------------------------------------------------------------- |
| Set Claude's persona and rules                    | Edit `CLAUDE.md`                                               |
| Add a repeatable document task                    | Add a file to `.claude/command/`                               |
| Add deep domain logic                             | Add a folder + `SKILL.md` to `.claude/skills/`                 |
| Pre-approve a CLI command                         | Add to `permissions.allow` in `.claude/settings.json`          |
| Set your local org alias                          | Set `SF_TARGET_ORG` in `.claude/settings.local.json`           |
| Suppress VS Code Apex LS errors on template files | Already configured in `.vscode/settings.json` + `.forceignore` |
| Store design docs                                 | Claude saves them to `docs/` automatically via commands        |
