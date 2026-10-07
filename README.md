# Prompt Fixer

A Claude plugin that turns a rough request into a clear prompt Claude can get right the first time.

You type something vague, like "write an email to my landlord" or "the discount is wrong, fix it". Prompt Fixer asks you a few quick questions, mostly as options you can click. Then it hands back a short prompt you can copy and run, with a suggested model. It doesn't do the task itself or read your files. It asks you instead, because you already know the answers.

It works in two modes:

- **Writing and everyday tasks**, for anyone: emails, messages, posts, reports, summaries, slides, plans and rewrites. It asks what the piece is about, who it's for, the tone, the length and anything to include or avoid. If you have a skill installed that fits, like a document or slides skill, it asks whether to use it.
- **Code**, in Claude Code: it asks which file, what the error is and how to check the fix, then writes a prompt that ends with a command to prove it worked. It works with any codebase. The examples and test commands lean toward Salesforce (Apex, LWC, Flow, metadata), because that's where it started.

**Quick install:** go to [claude.ai/customize/plugins](https://claude.ai/customize/plugins), open **Discover**, search for **Prompt Fixer** and select **Add**. Other ways to install are [below](#install).

## What it does

- Asks only for what's missing from your prompt, in one batch (two for writing, when the details need it).
- For writing, asks about content first, then audience, tone and length. Style questions have a "You decide" option, so you can skip what you don't care about.
- Offers your installed skills when one fits the job, and names it in the prompt.
- Never makes up facts. Names, dates and numbers you didn't give become `<placeholders>` for you to fill in.
- For code, rewrites the prompt to start with a verb, name the exact file and method, include the real error text, and end with a `Done when:` check Claude can run.
- Removes filler that wastes tokens: greetings, "make no mistakes", "you are a senior architect", ALL CAPS warnings.
- Splits a prompt that mixes unrelated tasks into separate prompts.
- For code, tells you when a built-in command or a rule does the job better. A standing rule goes in `CLAUDE.md`, and some jobs are better done with a built-in command like `/rewind` or `/clear`.
- Strips out any API key, token or password you pasted by accident.
- Adds a confirmation step before risky code work such as deleting data or deploying to production.

### Example: an email

You type:

```text
write an email to my landlord about the broken heater
```

Prompt Fixer asks what's wrong and since when, what you want the landlord to do, the tone (polite and firm, friendly, formal, or "you decide"), and whether to use your humanizer skill. Then it returns:

```text
Write an email to my landlord, <landlord name>, asking him to repair the heater in my flat by Friday 10 October.
Facts: the heater stopped working on 2 October. I reported it by phone on 3 October and nothing has happened since.
Tone: polite and firm. Remind him repairs are his responsibility, without threats.
Format: under 150 words, with a clear subject line. End by asking him to confirm a repair date.
```

### Example: code

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

In Claude chat or Cowork, type `/` in the message box and pick Prompt Fixer, or paste your draft and ask Claude to improve it. For example: "help me write a prompt for an email to my landlord".

You don't have to use the slash command. Claude picks up the skill when you paste a draft prompt and ask it to improve, fix, rewrite or tighten it, or ask "how should I ask Claude this?"

Answer the questions, then copy the prompt it returns. Or reply "run it" and Claude will run the improved prompt as the task.

Writing requests work the same everywhere. For code requests, some tips (Claude Code commands such as `/clear` or plan mode) only apply in Claude Code.

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
    ├── templates.md         Prompt shapes for code tasks
    └── writing.md           Prompt shapes for emails, posts, documents and plans
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
