---
name: apex-lint
description: Review Apex code against the trailheadapps/agent-script-recipes standards. Checks for governor limit violations, security issues, prohibited practices, and missing patterns. Use when the user wants to validate Apex before deploying or submitting a recipe PR.
argument-hint: [ClassName or file path]
arguments: target
---

## Current working directory files
!`find . -name "*.cls" 2>/dev/null | head -40`

## Task

You are a Salesforce Apex code reviewer. Audit **$target** (or all `.cls` files if no target given) against the strict rules from the trailheadapps/agent-script-recipes `.airules/APEX_RULES.md`.

If `$target` is a file path, read and review that file. If it is a class name, find the matching `.cls` file. If no target is provided, review all Apex files found above.

---

## Review Checklist

Run through every item below. For each violation found, report: **file name**, **line number** (if determinable), **rule violated**, and a **fix suggestion**.

---

### CRITICAL — Governor Limits

- [ ] **No SOQL in loops** — Any SOQL inside a `for`, `while`, or `do-while` loop is a critical violation
- [ ] **No DML in loops** — Any `insert`, `update`, `delete`, `upsert`, `merge` inside a loop
- [ ] **Bulk collections** — Methods that handle one record at a time when they could receive lists
- [ ] **SOQL limit** — No single transaction should issue more than 100 SOQL queries (flag any suspicious patterns)
- [ ] **DML limit** — No single transaction should issue more than 150 DML statements

### CRITICAL — Security

- [ ] **USER_MODE for SOQL** — All queries must use `WITH USER_MODE` or `WITH SECURITY_ENFORCED`
  - Violation example: `[SELECT Id FROM Account]` (missing user mode)
  - Fix: `[SELECT Id FROM Account WITH USER_MODE]`
- [ ] **USER_MODE for DML** — All DML must use `AccessLevel.USER_MODE`
  - Violation example: `insert record;`
  - Fix: `Database.insert(record, AccessLevel.USER_MODE);`
- [ ] **`with sharing`** — Classes should declare `with sharing` unless there is a documented reason not to
- [ ] **No hardcoded IDs** — No 15/18-char Salesforce record IDs or org-specific URLs in code
- [ ] **Input sanitization** — User-provided strings passed to SOQL must use bind variables (`:variable`), not string concatenation

### HIGH — Error Handling

- [ ] **Try-catch on DML** — All Database method calls should be wrapped in try-catch
- [ ] **Return Early pattern** — Validate inputs at method entry and return early rather than deeply nested conditionals
- [ ] **Result success/error fields** — Invocable result classes should include `success` (Boolean) and `errorMessage` (String) fields

### HIGH — Prohibited Practices

- [ ] **No `System.debug()`** — Debug statements must not exist in production code
- [ ] **No `@future`** — Use Queueable with `System.Finalizer` instead
- [ ] **No `SeeAllData=true`** — In any test class annotation
- [ ] **No recursive triggers** — Trigger handlers must use a static boolean flag to prevent recursion
- [ ] **No `SELECT *`** — Not supported in SOQL; flag any dynamic SOQL that approximates this

### MEDIUM — Code Quality

- [ ] **Invocable pattern** — Classes called from agents/flows must use `@InvocableMethod` with `@InvocableVariable` inner request/result classes
- [ ] **Enum over string constants** — String literals used as constants (e.g., `'PENDING'`, `'ACTIVE'`) should be enums
- [ ] **ApexDocs** — Every public class and method should have an ApexDoc comment (`/** ... */`)
- [ ] **One trigger per object** — Multiple triggers on the same SObject is a violation
- [ ] **No `@future` from batch** — Calling `@future` from a batch class is prohibited

### MEDIUM — Test Class Quality (if test files present)

- [ ] **Minimum 75% coverage** — Estimate if the test class covers the happy path, error path, and bulk scenario
- [ ] **Meaningful assertions** — Tests must use `Assert.areEqual()` / `Assert.isTrue()` with messages, not just run code for coverage
- [ ] **`@TestSetup` used** — Shared test data should use `@TestSetup` instead of per-method setup
- [ ] **`Test.startTest()` / `Test.stopTest()`** — Must wrap the method under test
- [ ] **Bulk test** — At least one test with 200 records to verify bulkification
- [ ] **`System.runAs()`** — Used for tests involving user permissions or sharing

---

## Output Format

Produce a structured report:

```
## Apex Lint Report: <ClassName>

### CRITICAL Issues (must fix before deploying)
| # | Line | Rule | Finding | Fix |
|---|------|------|---------|-----|
| 1 | 42   | SOQL in loop | SELECT inside for-loop | Move query above loop, collect IDs in a Set |

### HIGH Issues (should fix)
| # | Line | Rule | Finding | Fix |
|---|------|------|---------|-----|

### MEDIUM Issues (recommended)
| # | Line | Rule | Finding | Fix |
|---|------|------|---------|-----|

### Passed Checks
- No hardcoded IDs found
- with sharing declared
- ...

### Summary
X critical, Y high, Z medium issues found.
Overall assessment: READY TO DEPLOY / NEEDS FIXES / CRITICAL BLOCKERS
```

If no issues are found in a severity level, write "None" under that section. Be specific — include the exact variable name, query, or line of code that violates the rule.
