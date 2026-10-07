---
name: prompt-fixer
description: Turns a rough or vague Claude Code prompt into a precise, token-efficient one by asking a few targeted questions first. Use whenever the user runs /prompt-fixer, pastes a draft prompt and asks to improve, fix, rewrite, sharpen or optimize it, says things like "make this a good prompt" or "how should I ask Claude this", or wants help phrasing a coding, debugging or Salesforce (Apex, LWC, Flow, metadata) request before running it.
argument-hint: "[your rough prompt, error or stack trace]"
effort: medium
---

# Prompt fixer

Take the user's rough prompt, ask only the questions needed to fill the gaps, and return a prompt that Claude Code can execute in one pass.

The goal is fewer tokens overall: a precise prompt avoids exploration and correction rounds. So this skill must itself stay cheap. Do not execute the rough prompt, and do not read project files to guess at answers. Ask the user instead; they already know.

Tools: use only AskUserQuestion, and Read for `references/templates.md`. Do not run shell commands (no Bash, grep, cat or date) and do not use Grep or Glob, not even on this skill's own files. Each of those calls stops for a permission prompt, and the user sees nothing but a waiting spinner.

## Long pastes

Users often paste whole stack traces, debug logs or source files. Do not analyse them. Your job is to write the prompt, not to diagnose the bug, so a long paste should take no more effort than a short one. Skim it for these, then stop reading:

- the first exception or error line, verbatim
- the top 3 to 5 frames that point to the user's own code (skip framework and library frames)
- the trigger or condition the user describes
- file, class, method, object and field names
- any secrets to remove

Do not decode timestamps, count repeated lines, compare frames or reason about the root cause. In the improved prompt, never reprint the whole paste. Keep the error line and the top frames, and point to the rest with `@file` or "the full trace is in <path>".

## Step 1: Check the five parts

A good prompt has up to five parts. Mark each as present, missing, or not needed.

| Part | Question it answers |
| --- | --- |
| Goal | What should change? (verb first) |
| Where | Which file and method, or which object, field, flow, component? |
| Context | What is the symptom and exact error, or which existing pattern should be copied? |
| Constraints | What must not change? |
| Done when | What runnable check proves it worked? |

Size the result to the task. If the change could be described as a one-line diff (rename, typo, label, single null check), skip the questions: tighten the wording and return it. Not every prompt needs all five parts.

If the prompt holds more than one task, split it before anything else. Tasks are related when one needs the other (a new field and the component that shows it); keep those together as numbered steps in one prompt. Unrelated tasks (a bug fix plus a new feature) become separate prompts, each with its own Done-when, and the output tells the user to run them in separate sessions with `/clear` between. The four-question limit applies per prompt; ask about the first task now and say the rest will follow.

Check whether a prompt is the right tool at all:
- A standing rule ("always use TestDataFactory", "never use SeeAllData") is not a task. Output it as a line for CLAUDE.md and say that is where it belongs.
- If a built-in command does the job, name it instead of writing a prompt: `/rewind` to undo, `/clear` to start over, `/compact` to shrink context, `/model` or `/effort` to switch.

If the prompt already has every part it needs and no filler, do not rewrite or reprint it. Reply `Already good. Run as is.` followed by the `Run with:` line, and stop.

## Step 2: Ask, once

Ask for the missing parts in a single batch of at most four questions. Use the AskUserQuestion tool if it is available; otherwise ask in one short message.

- Ask only for what is missing. Never ask for something already in the prompt, in CLAUDE.md, or earlier in the conversation.
- Offer likely answers as options where possible, so the user can pick instead of type. Always allow a free-text answer.
- Options for Where may only use names that appear in the prompt. If the prompt names no file, offer "I'll type the path" and "Don't know" rather than guessed paths: a guessed option that the user clicks becomes an invented fact.
- Prefer the questions that prevent the most rework, in this order: Where, Done when, Context (exact error text), Constraints.
- For bugs, always get the exact error message or symptom and what "fixed" looks like.
- If it is unclear whether the user wants the change made or only advice ("can you suggest", "what do you think", "should I"), ask which. Then write "Change..." for the first, or "Recommend... Do not edit" for the second.
- If the user does not know an answer, keep it as a `<placeholder>` in the output. Never invent file names, class names, field API names or error text.
- Paths: when the user names a class or component (for example `AccountService` or `quoteBuilder`), you may expand it to the standard SFDX location (`force-app/main/default/classes/AccountService.cls`, `force-app/main/default/lwc/quoteBuilder/`). Point to the component folder, not a specific file inside it, unless the user said which file. If no name was given, use a placeholder.

If the user cannot answer most of the questions, do not return a prompt full of placeholders. Use the "Cause unknown" template for a bug, or the interview prompt for a feature, filled in with whatever is known.

Some tasks are too big for four questions. Say so, and pick from `references/templates.md`:
- Fuzzy (requirements unclear, new feature): the interview prompt.
- Broad but clear (many files, known goal, and the approach matters): the plan-mode prompt.
- Broad but mechanical (lint fixes, a rename across files): neither. Write a normal prompt with a Done-when check.

## Step 3: Write the prompt

Apply these rules. Each exists because it removes a correction round or wasted tokens.

Add:
- Start with a verb: Add, Fix, Rename, Create, Extract.
- Reference files with `@path`. Name the method.
- Use API names (`Invoice__c.Status__c`), not labels.
- State the symptom and the exact error, plus the condition that triggers it.
- Point to an existing file to copy the pattern from, when one exists.
- State the scope positively: "Change only `buildQuote`."
- Give a short reason for any constraint that is not obvious ("the LWC calls this method").
- End with `Done when:` and a check Claude can run. See the table below.
- For bugs, ask for a failing test first, then the fix.
- State the symptom and the expected result, and leave the solution to Claude. Include a specific fix or technique only if the user named it; adding your own guess narrows the search to an approach nobody checked.
- Ask for a short reply when useful: "No recap", "Answer in 5 lines".
- Make the prompt self-contained. Replace "same as before", "that file" and "it" with the actual names, so the prompt still works after `/clear`.
- If no runnable check exists (visual changes, accessibility, performance with no benchmark), do not invent one. Ask for evidence instead: "List what changed and how I can verify it by hand."

Risky operations (deleting data, bulk data changes in an org, deploying to production, force-pushing): ask which org or branch if it was not stated, name it in the prompt, add a read-only first step (a count, a dry-run, a diff), and end with "Stop and wait for my confirmation before changing anything." A wrong guess here cannot be undone with `/rewind`.

Secrets: never carry a pasted key, token or password into the prompt. Replace it with a placeholder or a Named Credential reference, and tell the user in one line that it was removed.

Remove:
- Greetings, thanks, apologies, "could you kindly".
- "Make no mistakes", "be careful", "write clean code", "double-check": replace with the Done-when check.
- Expert personas ("You are a senior Salesforce architect").
- All caps, "CRITICAL", "YOU MUST", "NEVER EVER": use a plain instruction.
- "Think step by step", "think hard": effort is set with `/effort`, not words.
- "Can you suggest..." when the user wants the change made: use "Change...".
- Long lists of don'ts: replace with one statement of scope.
- Pasted whole files or logs: replace with `@file`, a line range, or the relevant lines.

Keep it short. Most good prompts are 2 to 6 lines. Do not pad a prompt to fill all five parts.

### Done-when checks (Salesforce)

| Work | Check |
| --- | --- |
| Apex | `sf apex run test --class-names <TestClass> --result-format human` passes |
| One test method | `sf apex run test --tests <TestClass>.<method>` passes |
| Coverage target | `sf apex run test --class-names <TestClass> --code-coverage` shows at least <n>% for <Class> |
| LWC | `npm run test:unit -- <componentName>` passes |
| Lint | `npm run lint` shows no errors |
| Metadata, flows | `sf project deploy start --dry-run --source-dir <path>` succeeds |
| Data | a SOQL count returns the expected number |

For other stacks, use the project's own test, build or lint command.

### Templates

`references/templates.md` holds ready-made shapes for common scenarios (exception, failing test, governor limit, deploy error, LWC error, flow fault, regression, unknown cause, new class, trigger, test class, LWC, metadata, flow change, refactor, investigate, plan, review). Read only the section that matches the task, and only if the shape is not obvious.

## Step 4: Return it

Output exactly this, and nothing else:

1. The improved prompt in a fenced `text` code block, ready to copy.
2. One line, `Run with:`, giving the suggested model, effort and mode:
   - trivial edit (rename, label, typo): `haiku`, effort low
   - read-only question about one file, or advice with no edits: `sonnet`, effort low
   - small metadata change (one field, validation rule, permission, flow outcome): `sonnet`, effort low
   - normal coding and debugging: `sonnet`, effort medium
   - a change to one component or class where the approach is unclear, or a migration: `sonnet`, effort medium, plan mode first
   - architecture or a refactor across many files: `opusplan`, plan mode first
   - anything that reads many files or logs: add "via a subagent"
3. If any `<placeholders>` remain, one line listing them.
4. Any one-line notice required above (a secret was removed; a rule belongs in CLAUDE.md; a built-in command does this).
5. One line: `Run it now, or edit first?`

When the input was split into several prompts, give one code block per prompt, each followed by its own `Run with:` line, then one line: `Run these in separate sessions, with /clear between.`

Do not explain what was changed unless the user asks. If the user says to run it, execute the improved prompt as the task.

## Examples

**Input:** `hi, the discount is wrong can you please fix it and make no mistakes thanks`

Questions asked: which file and method; what input gives the wrong result and what is expected; which test class covers it.

**Output:**

```text
In @force-app/main/default/classes/OpportunityService.cls, calculateDiscount returns 0
when Quantity is over 100; expected 15%.
Write a failing test first, then fix. Change only this method.
Done when: sf apex run test --class-names OpportunityServiceTest
```

Run with: sonnet, effort medium
Run it now, or edit first?

**Input:** `rename accList to accounts in AccountService`

No questions (one-line diff).

**Output:**

```text
Rename accList to accounts in @force-app/main/default/classes/AccountService.cls.
```

Run with: haiku, effort low
Run it now, or edit first?
