# codex-auditor

A Claude Code skill that uses the OpenAI Codex CLI as an independent second reviewer for code that Claude Code wrote or changed.

Claude Code builds. Codex audits. Different models catch different mistakes.

## What it is

- A single `SKILL.md` that tells Claude Code when and how to call `codex review` and `codex exec` on changed files.
- A standard workflow: find changed files with `git diff`, send each file or diff to Codex, and sort the findings into `MUST_FIX`, `SHOULD_FIX` and `INFO`.
- Ready-made prompts for focused audits: security, performance and architecture.
- An optional `PostToolUse` hook that runs `codex review` after every edit.

## What it is not

- It is not a replacement for tests. It does not run your code.
- It is not a static analyzer or a linter.
- It does not ship any API key, token or account. You use your own OpenAI account.
- It does not fix code on its own. Claude Code reads the findings and decides what to fix.

## How it works

```
Claude Code changes code
        |
git diff --name-only  ->  list of changed files
        |
codex exec "review this diff ..."  (one call per file)
        |
Findings sorted: MUST_FIX / SHOULD_FIX / INFO
        |
MUST_FIX   -> Claude Code fixes before delivering
SHOULD_FIX -> reported to you
INFO       -> kept in the report
```

## Prerequisites (onboarding)

1. Claude Code installed.
2. Node.js 18 or newer.
3. OpenAI Codex CLI: `npm install -g @openai/codex`.
4. Sign in: `codex login` (ChatGPT account or OpenAI API key).
5. Check: `codex --version` prints a version.
6. A git repository, so `git diff` can find the changed files.

## Installation

```bash
git clone https://github.com/ojuliocouto/skill-codex-auditor.git
mkdir -p ~/.claude/skills/codex-auditor
cp skill-codex-auditor/SKILL.md ~/.claude/skills/codex-auditor/
```

Restart Claude Code. The skill loads on phrases such as "auditar com codex", "codex review", "segunda opinião" or `/codex-auditor`.

## Usage

| Mode | Command | When |
|------|---------|------|
| Quick review | `codex review` | After changes |
| Full audit | The workflow in `SKILL.md` | Before deploy |
| Second opinion | `codex exec "Analyze..."` | Technical decision |
| Security | Focused security prompt | Auth, payments, personal data |

### Optional hook

Add this to `~/.claude/settings.json` to run a review after every edit:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "codex review 2>/dev/null || true"
      }
    ]
  }
}
```

Warning: the hook calls Codex on every edit and uses your OpenAI quota each time. Prefer manual use, except on critical code.

## Contents

| File | Purpose |
|------|---------|
| `SKILL.md` | The skill: triggers, workflow, prompts, hook and limits (in Portuguese) |
| `README.md` | This file |
| `LICENSE` | MIT license |

## Limitations

- Each Codex call uses OpenAI quota or credits.
- Answers can be generic without enough context. Always send the full file or diff.
- Split files over about 500 lines into sections.

## Security

- The repository holds no secrets. Authentication stays in your local Codex CLI login.
- The prompts send your source code to OpenAI. Do not use it on code you are not allowed to share with a third-party provider.

## License

MIT. See `LICENSE`.
