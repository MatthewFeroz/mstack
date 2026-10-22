# Matthew's agent setup

This repo backs up my shared rules and four core skills for Claude Code and Codex on macOS. The installer restores the copies under `global/` and the four skill folders below.

## What's included

| File or folder | Purpose |
| --- | --- |
| [`global/AGENTS.md`](global/AGENTS.md) | Shared rules for permissions, verification, reports, commits, and tooling. |
| [`global/CLAUDE.md`](global/CLAUDE.md) | Imports the shared rules and assigns review, debugging, and visual checks to Astra. |
| [`skills/unslop`](skills/unslop/SKILL.md) | Removes AI writing patterns. Includes the local rule that protects code, paths, URLs, and timestamps. |
| [`skills/mock`](skills/mock/SKILL.md) | Builds UI comparisons from the included HTML template. |
| [`skills/writing-for-agents`](skills/writing-for-agents/SKILL.md) | Guides agent instructions, with supporting references and the third-party license. |
| [`skills/reproduce-bug`](skills/reproduce-bug/SKILL.md) | Recreates bugs in the running product, isolates the trigger, and prepares evidence for properly formatted issues. Includes a T3 Code adapter. |

The installer restores this core setup. The other skills already in this repo remain available as separate workflows.

## Restore on a Mac

Install Git, Python 3.9 or later, Claude Code, and Codex. Set up their sign-ins and model access on the new machine. For the Claude/Astra workflow in T3, use Claude Opus as the main agent and make `gpt-6-astra` available on the `codex` provider. Outside T3, `CLAUDE.md` uses the Codex CLI to reach Astra.

Clone this repo into a permanent folder:

```sh
git clone https://github.com/MatthewFeroz/mstack.git ~/mstack
cd ~/mstack
python3 install.py --dry-run
python3 install.py
```

The installer copies the rulebook and skill packages into your user directories. It uses relative symlinks so both agents read the same skill files:

```text
~/.agents/AGENTS.md
~/.agents/skills/unslop/
~/.agents/skills/mock/
~/.agents/skills/writing-for-agents/
~/.agents/skills/reproduce-bug/
~/.codex/AGENTS.md -> ../.agents/AGENTS.md
~/.codex/skills/<skill> -> ../../.agents/skills/<skill>
~/.claude/CLAUDE.md
~/.claude/skills/<skill> -> ../../.agents/skills/<skill>
```

Before replacing a different file, directory, or symlink, it moves the previous copy into `~/.agents/backups/md-<timestamp>/`, preserving its original relative path. Matching copies stay in place, so rerunning the installer makes no changes. Existing symlinks to other skill folders are preserved as links in the backup; their targets stay untouched.

To undo a replacement, move the installed copy aside, then move the corresponding backup item back to its original path under your home directory. Move a backed-up symlink itself. A relative symlink resolves from its original location again once restored.

The installer uses the standard Codex directory at `~/.codex`. A custom `CODEX_HOME` needs corresponding paths set up separately. It stops before writing if a managed parent directory, such as `~/.claude/skills`, is a symlink or a file.

Start a fresh Claude Code or Codex session after installation. Ask it to summarize the shared rules and list `unslop`, `mock`, `writing-for-agents`, and `reproduce-bug`. In Claude, ask it which jobs go to Astra. Then use a small real task to check the workflow, such as asking for two UI variants with `mock` or investigating a reported bug with `reproduce-bug`.

OpenAI's [AGENTS.md documentation](https://learn.chatgpt.com/docs/agent-configuration/agents-md#create-global-guidance) describes the global rulebook location. Its [skills documentation](https://learn.chatgpt.com/docs/customization/overview#skills) describes global skills and discovery.

## Keep the backup current

Edit the copies in this repo, commit and push the changes, then run `python3 install.py` to update your local installation. On another Mac, pull the latest commit and rerun the installer. It creates a backup whenever it replaces a different installed copy.

If you edit an installed file first, copy that change back into the matching repo file before reinstalling. The installer restores the repo version.

## Test the installer

```sh
python3 -m unittest discover -s tests -v
```

The tests run the real installer in temporary user directories, checking fresh installation, repeated runs, previews, preservation of existing files, and incomplete backups. Use `python3 install.py --home <temporary-directory>` to try a restoration yourself.

## Credits

The rulebook and core skills were adapted from [SunkenInTime/software-factory](https://github.com/SunkenInTime/software-factory). `writing-for-agents` comes from [mattpocock/skills](https://github.com/mattpocock/skills); its MIT license is included in that skill folder.

`reproduce-bug` adapts [T3 Code's test-t3-app workflow](https://github.com/pingdotgg/t3code/blob/b5a226e49c52ecab4ddc2d678a7012d97e651a6a/.agents/skills/test-t3-app/SKILL.md) and SQLite fixture guidance. Its T3 Code MIT notice is included in the skill folder.
