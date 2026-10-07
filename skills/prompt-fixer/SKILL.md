---
name: prompt-fixer
description: Turns a rough or vague prompt into a precise one by asking a few targeted questions first. Works for code (Claude Code, debugging, Salesforce Apex, LWC, Flow, metadata) and for everything else (emails, messages, posts, reports, summaries, plans, slides, research). Use whenever the user runs /prompt-fixer, pastes a draft prompt and asks to improve, fix, rewrite, sharpen or optimize it, says things like "make this a good prompt", "help me ask Claude this" or "how should I ask Claude this", or wants help phrasing any request before running it.
argument-hint: "[your rough prompt, request, error or stack trace]"
effort: medium
---

# Prompt fixer

Take the user's rough prompt, ask only the questions needed to fill the gaps, and return a prompt that Claude can execute in one pass.

The goal is fewer tokens overall: a precise prompt avoids exploration and correction rounds. So this skill must itself stay cheap. Do not execute the rough prompt, and do not read project files to guess at answers. Ask the user instead; they already know.

Tools: use only AskUserQuestion, and Read for this skill's `references/templates.md` and `references/writing.md`. Do not run shell commands (no Bash, grep, cat or date) and do not use Grep or Glob, not even on this skill's own files. Each of those calls stops for a permission prompt, and the user sees nothing but a waiting spinner.

## Long pastes

Users often paste whole stack traces, debug logs or source files. Do not analyse them. Your job is to write the prompt, not to diagnose the bug, so a long paste should take no more effort than a short one. Skim it for these, then stop reading:

- the first exception or error line, verbatim
- the top 3 to 5 frames that point to the user's own code (skip framework and library frames)
- the trigger or condition the user describes
- file, class, method, object and field names
- any secrets to remove

Do not decode timestamps, count repeated lines, compare frames or reason about the root cause. In the improved prompt, never reprint the whole paste. Keep the error line and the top frames, and point to the rest with `@file` or "the full trace is in <path>".

## Step 0: Code or not?

Pick the mode first.

- **Code mode:** the task changes, debugs, reviews or asks about code, configuration, Salesforce metadata, data in an org, a repo or a terminal command. Follow Steps 1 to 4.
- **Writing mode:** everything else, such as emails, messages, posts, letters, reports, summaries, plans, slides, research, analysing a document, brainstorming, translating or learning a topic. Follow "Writing mode" further down instead of Steps 1 to 4.

If the request names no file, code, error, repo or org, use writing mode. The rules on long pastes, secrets and always returning a prompt apply to both modes.

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
- A standing rule ("always use TestDataFactory", "never use SeeAllData") is not a task. Return a prompt that adds it to CLAUDE.md ("Add this line to CLAUDE.md: ...") and say that is where it belongs.
- If a built-in command does the job, return that command in the code block (for example `/rewind`) and say so: `/rewind` to undo, `/clear` to start over, `/compact` to shrink context, `/model` or `/effort` to switch.

If the prompt already has every part it needs and no filler, return it unchanged in the code block, add the line `Already good. Run as is.`, then the `Run with:` line.

Every reply ends with at least one prompt in a fenced `text` code block, in every case, including advice and "how should I" requests. Never answer the question, diagnose the problem or start the task yourself; the prompt is the output.

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
- End with `Done when:` and the exact command to run, scoped to the changed code, that finishes on its own: one test file or test name, not the whole suite, and never a watch mode, `--server`, `serve` or dev server. Naming a tool ("ember-tsc passes") is not enough; write the command. If you do not know the command, ask for it. If the user does not know either, use a placeholder such as `<command that runs only doc-checklist-item-test>` and list it with the other placeholders. See the table below.
- Do not state what you have not confirmed as fact. If the change depends on something the user did not confirm (a field exists, a value is passed in, an API name is available), write it as a condition and widen the scope to where the data comes from: "If answers do not carry `apiName` yet, add it where they are built." Otherwise the prompt fences Claude into files that cannot complete the change.
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
2. One line, `Run with:`, giving the suggested model, effort and mode. Use only these exact combinations; never pair `haiku` with any effort other than low:
   - mechanical edit in one file (rename, label, typo, a constant), where the prompt names the file, nothing needs judging, and the Done-when command is known (no placeholder): `haiku`, effort low
   - read-only question about one file, or advice with no edits: `sonnet`, effort low
   - small metadata change (one field, validation rule, permission, flow outcome): `sonnet`, effort low
   - normal coding and debugging, any change that touches more than one file, or any task that must judge whether the change is right: `sonnet`, effort medium
   - a change to one component or class where the approach is unclear, or a migration: `sonnet`, effort medium, plan mode first
   - architecture or a refactor across many files: `opusplan`, plan mode first
   - anything that reads many files or logs: add "via a subagent"
   When in doubt between two rows, pick the stronger one. A smaller model that needs twice the tool calls, or a second attempt, costs more than it saves.
3. If any `<placeholders>` remain, one line listing them.
4. Any one-line notice required above (a secret was removed; a rule belongs in CLAUDE.md; a built-in command does this).
5. One line: `Run it now, or edit first?`

When the input was split into several prompts, give one code block per prompt, each followed by its own `Run with:` line, then one line: `Run these in separate sessions, with /clear between.`

Do not explain what was changed unless the user asks. If the user says to run it, execute the improved prompt as the task.

## Writing mode

### W1: Check the six parts

A good writing prompt has up to six parts. Mark each as present, missing, or not needed.

| Part | Question it answers |
| --- | --- |
| Deliverable and goal | What to produce (email, post, one-page summary, slides) and what it should achieve: get a reply, a yes, a decision, an apology accepted |
| Audience | Who reads it, how well they know the user, what they already know |
| Content | The facts only the user knows: what happened, key points, names, dates, numbers, the ask |
| Tone and style | Formal, friendly, warm, direct, persuasive; whose voice; a sample to match |
| Format and length | Length, structure (paragraphs, bullets, sections), language, subject line |
| Constraints | What to include or avoid, sensitive points, a deadline |

Size it to the request. "Make this paragraph sound more professional" with the text pasted needs no questions: write the prompt.

Never invent content. Facts, names, dates, figures, reasons and the ask come from the user. Without them the result is generic filler, however good the wording. If the user does not know or skips one, use a `<placeholder>`.

### W2: Ask

Ask in rounds of at most four questions. Use AskUserQuestion when it is available.

- Ask about content first, because only the user knows it: what is this about, what happened, what do you want the reader to do. Then audience, tone, then format and length.
- Offer options, so the user can pick instead of type. For tone, for example: "Formal", "Friendly but professional", "Warm and personal", "Short and direct". For length: "3 to 4 sentences", "About 150 words", "One page". Always allow a free-text answer.
- Add a "You decide" option to style questions (tone, length, format), so the user can skip what they do not care about. Never add it to content questions.
- When the voice matters (a post under their name, a personal email), offer "Match a sample I'll paste" as a tone option.
- A second round is allowed, once, when an answer opens something essential. For example, they chose "apologise for the delay": ask what they are offering to put it right. Then write the prompt. Never more than two rounds.
- Skills: look at the skills listed as available in this session. If one fits the deliverable (a document skill for a report or letter, a slides skill for a deck, a spreadsheet skill for a table, a humanizer or writing-style skill for natural wording, a brand or voice skill), ask in the same round: "Use one of your skills for this?", with the matching skill names as options plus "No skill". Keep it a question of its own; do not merge it into the format or tone question. Offer only skills that appear in the available list, by their exact names. If none fit, do not ask.
- Without AskUserQuestion (for example in Claude chat), ask in one short message: numbered questions with lettered options, ending with "Reply like: 1b, 2a, 3: your words".

### W3: Write the prompt

- Start with the deliverable and the goal: "Write an email to <who> asking <what>, so that <goal>."
- Then one short line or a few bullets each for audience, key points, tone, format and length, and constraints. Keep the user's facts word for word.
- If they chose a skill, add "Use the <skill name> skill."
- Describe the tone concretely ("friendly and brief, like a message to a colleague") instead of stacking adjectives ("engaging, compelling, impactful").
- If they want options, say how many: "Give 3 subject lines."
- Remove greetings, thanks, personas ("You are a world-class copywriter"), "make it perfect" and all caps, as in code mode.
- Most writing prompts are 4 to 10 lines.

`references/writing.md` holds ready-made shapes (email, follow-up, bad news, post, summary, report, slides, plan, cover letter, rewrite). Read only the section that matches, and only if the shape is not obvious.

### W4: Return it

Output exactly this, and nothing else:

1. The improved prompt in a fenced `text` code block, ready to copy.
2. One line, `Run with:`:
   - short pieces (email, message, post, rewrite, translation): `sonnet`
   - long or demanding work (a report, research, a strategy, a plan, or analysing long material): `opus`, with extended thinking on
3. If any `<placeholders>` remain, one line listing them.
4. Any one-line notice (a secret was removed).
5. One line: `Run it now, or edit first?`

If the user says to run it, carry out the improved prompt as the task.

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

**Input:** `write an email to my landlord about the heater`

Questions asked: what is wrong with the heater and since when; what do you want the landlord to do, and by when; tone (options: Polite and firm, Friendly, Formal, You decide); length (options: Short, About 150 words, You decide).

**Output:**

```text
Write an email to my landlord, <landlord name>, asking him to repair the heater in my flat by Friday 10 October.
Facts: the heater stopped working on 2 October. I reported it by phone on 3 October and nothing has happened since. The flat is at 14°C in the mornings.
Tone: polite and firm. Remind him it is his responsibility, without threats.
Format: under 150 words, with a clear subject line. End by asking him to confirm a repair date.
```

Run with: sonnet
Placeholders: `<landlord name>`
Run it now, or edit first?
