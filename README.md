# prompt-fixer

A Claude skill that turns a rough prompt into one Claude Code can run in a single pass.

You paste a vague request like "the discount is wrong, can you fix it". The skill asks you up to four short questions: which file, what the error is, what "fixed" looks like. Then it gives you back a short prompt you can copy and run, with a suggested model and effort level. It doesn't run your task and doesn't read your project files. It only asks you, because you already know the answers.

It works for any codebase. The examples and test commands lean toward Salesforce (Apex, LWC, Flow, metadata), because that's where it started.

## What it does

- Asks only for what's missing from your prompt, all in one batch.
- Rewrites the prompt to start with a verb, name the exact file and method, include the real error text, and end with a `Done when:` check Claude can run.
- Removes filler that wastes tokens: greetings, "make no mistakes", "you are a senior architect", ALL CAPS warnings.
- Splits a prompt that mixes unrelated tasks into separate prompts.
- Tells you when you don't need a prompt at all. A standing rule goes in `CLAUDE.md`, and some jobs are better done with a built-in command like `/rewind` or `/clear`.
- Strips out any API key, token or password you pasted by accident.
- Adds a confirmation step before risky work such as deleting data or deploying to production.

### Example

You type:

```text
hi, the discount is wrong can you please fix it and make no mistakes thanks
```

The skill asks which file and method, what input gives the wrong result, and which test covers it. Then it returns:

```text
In @force-app/main/default/classes/OpportunityService.cls, calculateDiscount returns 0
when Quantity is over 100; expected 15%.
Write a failing test first, then fix. Change only this method.
Done when: sf apex run test --class-names OpportunityServiceTest
```

```text
Run with: sonnet, effort medium
Run it now, or edit first?
```

## Install

Pick the option that matches where you use Claude.

### Option 1: Claude Code plugin (recommended)

This repository is also a plugin marketplace, so you can install the skill straight from GitHub. Run these two commands inside Claude Code:

```text
/plugin marketplace add chatquill/prompt-fixer
/plugin install prompt-fixer@prompt-fixer
```

Then run `/reload-plugins` or start a new session. The skill is now available in every project.

To get new versions later, run `/plugin marketplace update prompt-fixer`.

### Option 2: Copy the skill folder into Claude Code

Use this if you'd rather not install a plugin.

1. Clone this repository:

   ```bash
   git clone https://github.com/chatquill/prompt-fixer.git
   ```

2. Copy the skill folder. For all your projects, copy it into your personal skills folder:

   ```bash
   mkdir -p ~/.claude/skills
   cp -r prompt-fixer/skills/prompt-fixer ~/.claude/skills/
   ```

   On Windows (PowerShell):

   ```powershell
   New-Item -ItemType Directory -Force "$HOME\.claude\skills"
   Copy-Item -Recurse prompt-fixer\skills\prompt-fixer "$HOME\.claude\skills\"
   ```

   For one project only, copy it into that project instead, then commit the folder so your team gets it too:

   ```bash
   mkdir -p .claude/skills
   cp -r /path/to/prompt-fixer/skills/prompt-fixer .claude/skills/
   ```

3. Check that `SKILL.md` is at `~/.claude/skills/prompt-fixer/SKILL.md` (or `.claude/skills/prompt-fixer/SKILL.md` in your project).

4. Start a new Claude Code session. A session that was already open won't see the new skill.

### Option 3: Claude.ai or the Claude desktop app

1. Download [`prompt-fixer.skill`](https://github.com/chatquill/prompt-fixer/releases/latest/download/prompt-fixer.skill) from the latest release. It's a zip file that contains the skill folder.
2. In Claude, open **Settings** and find the **Skills** section (under **Capabilities** on most accounts).
3. Choose **Upload skill** and select `prompt-fixer.skill`.
4. Make sure the skill is switched on.

Skills need code execution to be turned on in your settings. On Team and Enterprise plans, an admin may have to allow skills first.

## How to use it

In Claude Code, type:

```text
/prompt-fixer the discount is wrong, fix it
```

If you installed it as a plugin, the full name is `/prompt-fixer:prompt-fixer`. Typing `/prompt-fixer` and picking it from the list works too.

You don't have to use the slash command. Claude also picks up the skill when you paste a draft prompt and ask it to improve, fix, rewrite or tighten it, or ask "how should I ask Claude this?"

Answer the questions, then copy the prompt it returns. Or reply "run it" and Claude will run the improved prompt as the task.

## Files

```text
.claude-plugin/
├── plugin.json              Plugin manifest
└── marketplace.json         Lets this repo work as its own marketplace
skills/prompt-fixer/
├── SKILL.md                 The instructions Claude follows
└── references/
    └── templates.md         Ready-made prompt shapes for common tasks
```

`templates.md` covers debugging (exceptions, failing tests, governor limits, deploy errors, LWC and flow faults, regressions), building (Apex classes, triggers, test classes, LWC, metadata, flows, refactors), and planning or review. Claude reads only the section it needs.

## Updating

Plugin users: run `/plugin marketplace update prompt-fixer`.

If you copied the folder, pull the latest version and copy it again:

```bash
cd prompt-fixer
git pull
cp -r skills/prompt-fixer ~/.claude/skills/
```

For Claude.ai, delete the old skill in Settings and upload the `prompt-fixer.skill` from the latest release.

To build the `.skill` file yourself:

```bash
cd skills
zip -r ../prompt-fixer.skill prompt-fixer
```

## Privacy

The plugin has no code and sends no data anywhere. It only works with the prompt you type in your own Claude session. See [PRIVACY.md](PRIVACY.md).

## License

[MIT](LICENSE)
