---
name: apex-recipe
description: Scaffold a production-ready Salesforce Apex service class and its test class for use in Agentforce agent recipes. Use when the user needs to create an Invocable Apex class that an agent can call, or needs a test class for existing Apex.
argument-hint: [ClassName]
arguments: class_name
---

## Task

You are a Salesforce Apex expert. Generate a production-ready Apex service class and a complete test class for **$class_name** following the strict rules below, sourced from the official trailheadapps/agent-script-recipes `.airules/APEX_RULES.md`.

---

## Discovery (ask if not provided)

Before generating, confirm:
1. **Purpose** — What does this class do? (e.g., look up an order, process a payment, fetch a user profile)
2. **Agent Action** — Will this be called from an agent script action? If yes, it needs `@InvocableMethod`.
3. **Input/Output** — What inputs does the agent pass in? What does the class return?
4. **Data** — Which Salesforce objects does it query or modify?
5. **Async** — Does any operation need to run asynchronously?

---

## Apex Class Rules

### Class Structure (always in this order)
```apex
/**
 * @description <What this class does>
 * @author <Author>
 */
public with sharing class $class_name {

    // Inner request/result classes (for @InvocableMethod)
    // Static methods
    // Private helper methods

}
```

### Invocable Method Pattern (required when called from agent/flow)
```apex
@InvocableMethod(label='<Human Readable Label>' description='<What it does>')
public static List<Result> execute(List<Request> requests) {
    List<Result> results = new List<Result>();
    // process in bulk — always iterate the list
    for (Request req : requests) {
        results.add(processSingle(req));
    }
    return results;
}
```

### Inner Request/Result Classes
```apex
public class Request {
    @InvocableVariable(label='<Label>' description='<Desc>' required=true)
    public String inputParam;
}

public class Result {
    @InvocableVariable(label='<Label>' description='<Desc>')
    public String outputParam;

    @InvocableVariable(label='Success' description='Whether the operation succeeded')
    public Boolean success;

    @InvocableVariable(label='Error Message' description='Error details if unsuccessful')
    public String errorMessage;
}
```

### Database Operations
- Always use `WITH USER_MODE` for SOQL: `[SELECT Id FROM Account WITH USER_MODE]`
- Always use `AccessLevel.USER_MODE` for DML: `Database.insert(records, AccessLevel.USER_MODE)`
- Use Database Methods with exception handling, never raw DML
- Return Early pattern — validate inputs at the top, return early on failure

```apex
private static Result processSingle(Request req) {
    Result res = new Result();
    // Return early on invalid input
    if (String.isBlank(req.inputParam)) {
        res.success = false;
        res.errorMessage = 'inputParam is required';
        return res;
    }
    try {
        // SOQL in user mode
        List<Account> accounts = [
            SELECT Id, Name
            FROM Account
            WHERE Name = :req.inputParam
            WITH USER_MODE
            LIMIT 1
        ];
        res.success = true;
    } catch (Exception e) {
        res.success = false;
        res.errorMessage = e.getMessage();
    }
    return res;
}
```

### Enums (use instead of string constants)
```apex
public enum Status {
    PENDING,
    IN_PROGRESS,
    COMPLETED,
    FAILED
}
```

### Async (use Queueable, never @future)
```apex
public class AsyncProcessor implements Queueable, System.Finalizer {
    public void execute(QueueableContext ctx) { /* ... */ }
    public void execute(FinalizerContext ctx) { /* handle success/failure */ }
}
```

---

## Prohibited Practices

- No SOQL or DML inside loops
- No `System.debug()` statements
- No hardcoded IDs or URLs
- No `@future` methods — use Queueable with Finalizer
- No `SeeAllData=true` in tests
- No recursive trigger logic

---

## Test Class Rules

```apex
@IsTest
private class $class_nameTest {

    @TestSetup
    static void makeData() {
        // Create minimal test data here
    }

    @IsTest
    static void testHappyPath() {
        // Arrange
        $class_name.Request req = new $class_name.Request();
        req.inputParam = 'TestValue';

        Test.startTest();
        // Act
        List<$class_name.Result> results = $class_name.execute(
            new List<$class_name.Request>{ req }
        );
        Test.stopTest();

        // Assert — meaningful assertions, not just coverage
        Assert.areEqual(1, results.size(), 'Expected one result');
        Assert.isTrue(results[0].success, 'Expected success');
    }

    @IsTest
    static void testBulk() {
        // Always test with 200 records to verify bulkification
        List<$class_name.Request> requests = new List<$class_name.Request>();
        for (Integer i = 0; i < 200; i++) {
            $class_name.Request req = new $class_name.Request();
            req.inputParam = 'Value' + i;
            requests.add(req);
        }

        Test.startTest();
        List<$class_name.Result> results = $class_name.execute(requests);
        Test.stopTest();

        Assert.areEqual(200, results.size(), 'Expected 200 results');
    }

    @IsTest
    static void testInvalidInput() {
        $class_name.Request req = new $class_name.Request();
        req.inputParam = null;

        Test.startTest();
        List<$class_name.Result> results = $class_name.execute(
            new List<$class_name.Request>{ req }
        );
        Test.stopTest();

        Assert.isFalse(results[0].success, 'Expected failure on null input');
        Assert.isNotNull(results[0].errorMessage, 'Expected error message');
    }
}
```

### Test Requirements Checklist
- [ ] Minimum 75% code coverage
- [ ] Meaningful assertions (not just coverage runs)
- [ ] `@TestSetup` for shared test data
- [ ] `Test.startTest()` / `Test.stopTest()` wrapping the action
- [ ] Bulk test with 200 records
- [ ] Invalid/null input test
- [ ] No `SeeAllData=true`
- [ ] `System.runAs()` for permission-sensitive tests

---

## Output

Generate:
1. **`$class_name.cls`** — The complete Apex service class
2. **`$class_nameTest.cls`** — The complete test class
3. **Agent Script snippet** — A 5-line example showing how to call this class from a `.agent` file using an `apex` action
4. **Coverage estimate** — Confirm the test class should achieve ≥75% coverage and note any gaps
