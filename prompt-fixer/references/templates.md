# Prompt templates

Replace `<placeholders>`. Delete lines that do not apply. Read only the section you need.

Contents: Debugging and fixing · Building · Understanding, planning, review

## Debugging and fixing

### Exception with a stack trace
```text
@<File.cls> throws <exception type> at line <n> when <condition>.
Stack trace (top 5 lines): <paste>
Find the root cause; do not suppress the error.
Write a failing test that reproduces it, then fix.
Done when <TestClass> passes.
```

### Failing test
```text
<TestClass>.<method> fails: <assertion message only>.
Decide whether the test or the code is wrong and say which before changing anything.
Change only <File.cls> or the test, not both.
Done when: sf apex run test --tests <TestClass>.<method>
```

### Governor limit
```text
<method> in @<File.cls> hits "<limit message>" with <n> records.
Bulkify: no SOQL or DML in loops; collect, then query once with a Map.
Change only this method.
Done when <TestClass>.<bulkTest> passes with 200 records.
```

### Deployment error
```text
Deploy of <path> failed with:
<error lines only>
Fix the cause in the source files. Do not change unrelated metadata.
Done when: sf project deploy start --dry-run --source-dir <path>
```

### LWC error
```text
@lwc/<component>: <user action> gives <actual>; expected <expected>.
Console error: <one line>
Fix and add a Jest test for this case.
Done when: npm run test:unit -- <component>
```

### Flow fault
```text
Flow <Flow_Api_Name> fails at element <element> with: <fault message>.
Triggering record: <object>, <key field values>.
Read only that element and the ones feeding it. Explain the cause in 3 lines, then propose the fix. Do not edit yet.
```

### Bug on one specific record
```text
<Object> record <Id> shows <wrong value> in <field>; expected <right value>.
Query only that record and its related <child object> rows (named fields, LIMIT 20).
Trace which automation sets <field>: check <Trigger/Flow names> only.
Report the cause. Do not change data.
```

### Regression (it worked before)
```text
<feature> worked before <date or commit>. Now: <symptom>.
Run git log and git diff on <path> since then. Do not read other folders.
Name the commit and lines that caused it, then fix.
Done when <TestClass> passes.
```

### Cause unknown (investigate first)
```text
Symptom: <what happens>. Expected: <what should>. Seen in: <where>.
Use a subagent to investigate <files or folder> only.
Return the 3 most likely causes, ranked, with file and line. Max 10 lines.
Do not change any code.
```

### Debug log analysis
```text
Use a subagent on @<path/to/log>.
Report: first exception, the 5 lines before it, and any LIMIT_USAGE over 80%.
Max 15 lines. Do not restate the log.
```

### After two failed fixes
```text
/clear
<file and method>: <symptom>.
Already tried and did not work: <approach 1>, <approach 2>.
Try a different approach. Done when <check>.
```

## Building

### New Apex class or method
```text
Create <ClassName>.<method>(<params>) in <folder>.
Behaviour: <one or two sentences>.
Follow the pattern in @<ExistingClass.cls>.
Cases: <normal>, <empty or null>, <bulk 200>.
Done when <TestClass> passes. Create the test class too.
```

### Trigger and handler
```text
Create <Object>Trigger (<events>) calling <Object>TriggerHandler.
Logic: <rule>.
Follow @<ExistingTriggerHandler.cls>. One trigger per object.
Done when <Object>TriggerHandlerTest passes, including a 200-record test.
```

### Test class
```text
Write <TestClass> for @<Class.cls>.
Use TestDataFactory. Cover: <case 1>, <case 2>, <bulk>, <error case>.
Assert results, not just coverage.
Done when it passes.
```

### LWC component
```text
Create LWC <componentName> for <page or parent>.
Shows: <fields or data>. Actions: <buttons or events>.
Data: <wire or imperative> Apex <Controller.method>.
Follow @lwc/<existingComponent>. SLDS only.
Done when Jest passes for data, empty and error states.
```

### Field, validation rule or permission
```text
On <Object__c>, create <field / validation rule / permission> <Api_Name>:
<type, values, formula or condition, error message>.
Create or edit only <that metadata file>.
Done when: sf project deploy start --dry-run --source-dir <path>
```

### Flow change
```text
In flow <Flow_Api_Name>, <add / change> <element>: <condition and outcome>.
Change nothing else in the flow.
Done when a deploy dry-run succeeds.
```

### Refactor
```text
In @<File.cls>, <extract / rename / move> <what> into <where>.
No behaviour change. Keep public signatures.
Done when <TestClass> passes without edits to the tests.
```

## Understanding, planning, review

### Quick question about code
```text
In @<File>, what does <method> do when <condition>? Answer in 5 lines.
```

### Find where something is used
```text
Use a subagent to find every reference to <Object.Field__c or ClassName> in classes, triggers, flows and LWC.
Return file paths and line numbers only.
```

### Plan before a bigger change (plan mode)
```text
I want to <goal>. Read only <files or folder>.
List the files to change and the approach in under 15 lines.
Flag risks to existing automation on <Object>. Do not write code yet.
```

### Large or fuzzy feature (interview)
```text
I want to build <short description>. Interview me about requirements, edge cases and trade-offs.
Skip obvious questions. Then write the spec to SPEC.md, ending with how to verify it.
```

### Review before commit
```text
Use a subagent to review the current diff for: SOQL or DML in loops, missing null checks, missing bulk tests, hard-coded Ids.
Report only issues that affect correctness. File and line, one line each.
```
