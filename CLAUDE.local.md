# Local Project Context (CLAUDE.local.md)

_Note: This file contains project-specific, environment-specific, or developer-specific instructions. It supplements the overarching architectural guidelines found in `Claude.md`._

## 1. Current Project Context

- **Project Name:** [Insert Project Name, e.g., Acme Corp Transformation]
- **Current Sprint:** [Insert Sprint, e.g., Sprint 24]
- **Active Focus:** [Insert current feature/focus, e.g., Enhancing Order Management LWC and refactoring OrderTrigger logic]

## 2. Naming Conventions & Project Rules

- **Component/Class Prefix:** `[Prefix]_` (e.g., `Acme_OrderTriggerHandler`)
- **LWC Prefix:** `[prefix]` (e.g., `c-acme-order-datatable`)
- **API Version:** [e.g., 60.0]
- **Specific Libraries/Packages in use:** [e.g., fflib Apex Common, custom datatable components]

## 3. Local Development Environment

- **Tooling:** Salesforce CLI (Prefer modern `sf` commands over deprecated `sfdx`).
- **Scratch Org Definition:** `config/project-scratch-def.json` (Features: PersonAccounts, MultiCurrency).
- **Default Dev Org Alias:** `dev-scratch`

## 4. Common Local Commands

- **Deploy to Org:** `sf project deploy start`
- **Retrieve from Org:** `sf project retrieve start`
- **Run Apex Tests (Local):** `sf apex run test -l RunLocalTests -c -w 10`
- **Run LWC Jest Tests:** `npm run test:unit:coverage`
- **Create Scratch Org:** `sf org create scratch -f config/project-scratch-def.json -a dev-scratch -d 30 -y 10`

## 5. Current WIP / Developer Notes

- **Active Ticket:** [e.g., JIRA-4045]
- **Current Goal for Claude:** Focus heavily on mocking Apex responses in LWC Jest tests for the current ticket and ensure we reach 90% code coverage locally before generating a Pull Request.
