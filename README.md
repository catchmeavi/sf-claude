# Salesforce Project

This repository follows the Salesforce DX (SFDX) project structure, enhanced with a robust `.claude` configuration for AI agent orchestration and Agentforce development.

## Project Structure

```text
my-salesforce-project/
├── .claude/                            # The Claude configuration
│   ├── settings.json                   # Overrides global settings
│   ├── settings.local.json             # Local overrides (Git Ignored)
│   ├── rules/                          # Rules for specific file patterns
│   ├── command/                        # Commands to execute predefined tasks
│   │   ├── sf-agent-prompt.md
│   │   ├── sf-agent-topic.md
│   │   ├── sf-agentforce.md
│   │   ├── sf-apex-review.md
│   │   ├── sf-apex-test.md
│   │   ├── sf-data-model.md
│   │   ├── sf-deploy.md
│   │   ├── sf-flow.md
│   │   ├── sf-integration.md
│   │   ├── sf-security.md
│   │   └── sf-soql-query.md
│   └── skill/                          # Agent skills and logic
│       ├── agent-script/
│       │   └── SKILL.md
│       ├── apex-lint/
│       │   └── SKILL.md
│       ├── apex-recipe/
│       │   └── SKILL.md
│       ├── migrating-agentforce/
│       │   └── SKILL.md
│       ├── recipe-docs/
│       │   └── SKILL.md
│       └── standup/
│           └── SKILL.md
├── .sfdx/                              # SFDX project configuration
├── config/                             # Scratch org definition files
├── force-app/                          # Main Salesforce metadata
│   └── main/
│       └── default/
│           ├── agents/                 # Agentforce Agents
│           ├── bots/                   # Einstein Bots
│           ├── classes/                # Apex Classes
│           ├── flowDefinitions/        # Flow Definition metadata
│           ├── flows/                  # Salesforce Flows
│           ├── genAiFunctions/         # Generative AI Functions
│           ├── genAiPlannerBundles/    # Planner configurations
│           ├── genAiPlugins/           # AI Plugins
│           ├── genAiPromptTemplates/   # Prompt Builder templates
│           ├── lightningTypes/         # Custom Lightning types
│           ├── lwc/                    # Lightning Web Components
│           └── messagingChannels/      # Service Cloud Messaging
├── docs/                               # Architecture & design docs
├── CLAUDE.md                           # Essential: Instructions & rules for Claude
└── README.md
```

This repo doubles as a Claude Code custom-command library for Salesforce and Agentforce development workflows. Commands follow a strict five-section structure: **Role/Persona**, **Context**, **Output Format**, **Constraints**, **Variables**.

| Command            | Purpose                                                                   |
| ------------------ | ------------------------------------------------------------------------- |
| `/sf-agentforce`   | Full Salesforce Agentforce technical design document                      |
| `/sf-agent-topic`  | Agentforce agent topic + action specification for Einstein Copilot Studio |
| `/sf-agent-prompt` | Prompt Builder template with merge field mapping and token budget         |
| `/sf-apex-review`  | Apex code review — governor limits, CRUD/FLS, bulkification               |
| `/sf-apex-test`    | Generate Apex test classes with arrange/act/assert structure              |
| `/sf-soql-query`   | Generate and optimize SOQL with selectivity and index analysis            |
| `/sf-flow`         | Salesforce Flow design document with fault paths and limit notes          |
| `/sf-security`     | Security model design — OWD, permission sets, sharing rules               |
| `/sf-data-model`   | Data model schema with ERD, field types, and Data Cloud DMO mapping       |
| `/sf-deploy`       | Deployment runbook with pre-deploy checklist and rollback plan            |
| `/sf-integration`  | Integration design — REST/Platform Events, auth, error handling           |

When adding a new command, match the five-section template used in all existing command files. Use `{{double_curly_braces}}` for every value that changes per invocation.

## Output

Results will be stored inside the folder "**doc**". If the folder is not present, it will be created.
For results where the metadata will be created like "Generate Test Class", results will be saved inside the proper metadata folder.
