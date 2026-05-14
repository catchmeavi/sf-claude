# Salesforce Project

This repository follows the Salesforce DX (SFDX) project structure, enhanced with a robust `.claude` configuration for AI agent orchestration.

## Project Structure

```text
my-salesforce-project/
├── .claude/                            # The Claude configuration
│   ├── settings.json                   # Overrides global settings
│   ├── settings.local.json             # Local overrides (Git Ignored)
│   ├── rules/                          # Rules loaded for specific file patterns
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
│   └── skills/                          # The skills required by the agents
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
│           ├── classes/
│           ├── objects/
│           ├── triggers/
│           ├── lwc/
│           └── ...
├── docs/                               # Architecture & design docs
├── CLAUDE.md                           # Essential: Instructions & rules for Claude
└── README.md
```
