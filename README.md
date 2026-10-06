# Prompt Fixer

A Claude plugin that turns a rough prompt into one Claude Code can run in a single pass.

You paste a vague request like "the discount is wrong, can you fix it". Prompt Fixer asks you up to four short questions: which file, what the error is, what "fixed" looks like. Then it hands back a short prompt you can copy and run, with a suggested model and effort level. It doesn't run your task or read your project files. It asks you instead, because you already know the answers.

It works with any codebase. The examples and test commands lean toward Salesforce (Apex, LWC, Flow, metadata), because that's where it started.

**Quick install:** go to [claude.ai/customize/plugins](https://claude.ai/customize/plugins), open **Discover**, search for **Prompt Fixer** and select **Add**. Other ways to install are [below](#install).

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

Prompt Fixer asks which file and method, what input gives the wrong result, and which test covers it. Then it returns:

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

Option 1 suits most people. The others are for when you can't use the directory or want to install from source.

### Option 1: From the Claude directory (easiest)

Prompt Fixer is listed in Anthropic's plugin directory. You need a Pro, Max, Team or Enterprise plan.

1. Go to [claude.ai/customize/plugins](https://claude.ai/customize/plugins). In the desktop app, this is **Customize > Plugins**.
2. Open **Discover** and search for **Prompt Fixer**.
3. Select it, then select **Add**.

The plugin is saved to your account, so it works in Claude chat, Cowork and Claude Code without installing it again. In Claude Code, run `/reload-plugins` or start a new session to load it. Updates arrive on their own.

On Team and Enterprise plans, an Owner decides whether members can see the directory. If you can't find Prompt Fixer, ask them.

### Option 2: Claude Code plugin from GitHub

This installs the plugin straight from this repository, on one machine only. Run these two commands inside Claude Code:

```text
/plugin marketplace add chatquill/prompt-fixer
/plugin install prompt-fixer@prompt-fixer
```

Then run `/reload-plugins` or start a new session.

### Option 3: Copy the skill folder into Claude Code

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

### Option 4: Upload the skill file to Claude.ai

1. Download [`prompt-fixer.skill`](https://github.com/chatquill/prompt-fixer/releases/latest/download/prompt-fixer.skill) from the latest release. It's a zip file that contains the skill folder.
2. In Claude, open **Settings** and find the **Skills** section (under **Capabilities** on most accounts).
3. Choose **Upload skill** and select `prompt-fixer.skill`.
4. Make sure the skill is switched on.

Skills need code execution to be turned on in your settings. On Team and Enterprise plans, an admin may have to allow skills first.

## How to use it

In Claude Code, type:

```text
/prompt-fixer:prompt-fixer the discount is wrong, fix it
```

You can also type `/prompt-fixer` and pick it from the list. If you copied the folder (Option 3), the command is just `/prompt-fixer`.

In Claude chat or Cowork, type `/` in the message box and pick Prompt Fixer, or paste your draft and ask Claude to improve it.

You don't have to use the slash command. Claude picks up the skill when you paste a draft prompt and ask it to improve, fix, rewrite or tighten it, or ask "how should I ask Claude this?"

Answer the questions, then copy the prompt it returns. Or reply "run it" and Claude will run the improved prompt as the task.

Prompt Fixer is written for Claude Code. In chat and Cowork it still sharpens your prompt, but tips about Claude Code commands such as `/clear` or plan mode won't apply there.

## Updating

- **Claude directory (Option 1):** updates are automatic.
- **GitHub plugin (Option 2):** run `/plugin marketplace update prompt-fixer` in Claude Code.
- **Copied folder (Option 3):** pull the latest version and copy it again:

  ```bash
  cd prompt-fixer
  git pull
  cp -r skills/prompt-fixer ~/.claude/skills/
  ```

- **Uploaded skill (Option 4):** delete the old skill in Settings and upload `prompt-fixer.skill` from the latest release.

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

To build the `.skill` file yourself:

```bash
cd skills
zip -r ../prompt-fixer.skill prompt-fixer
```

## Feedback

Found a prompt it handles badly, or have an idea? [Open an issue](https://github.com/chatquill/prompt-fixer/issues). Include your original prompt and what you expected to get back.

## Privacy

The plugin has no code and sends no data anywhere. It only works with the prompt you type in your own Claude session. See [PRIVACY.md](PRIVACY.md).

## License

[MIT](LICENSE)
