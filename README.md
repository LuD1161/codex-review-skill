# codex-review

A Claude Code plugin that sends your plans and code to OpenAI Codex CLI for iterative review. Claude and Codex go back-and-forth until Codex approves.

## Skills

| Skill | Description |
|---|---|
| `/codex-review:plan` | Review an implementation plan. Claude revises the plan based on Codex feedback until approved. |
| `/codex-review:code` | Review code changes from the current session. Claude fixes issues Codex finds until approved. |

## How It Works

1. Claude sends your plan or code diff to Codex for review
2. Codex reviews for bugs, security, performance, correctness, and more
3. Claude automatically revises the plan or fixes the code based on feedback
4. The loop continues (up to 5 rounds) until Codex approves

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- [OpenAI Codex CLI](https://github.com/openai/codex) installed (`npm install -g @openai/codex`)
- An OpenAI API key configured for Codex

## Installation

### From a marketplace

If this plugin is listed in a marketplace you've added:

```bash
/plugin install codex-review@<marketplace-name>
```

### Local development

```bash
claude --plugin-dir /path/to/codex-review-skill
```

## Usage

### Review a plan

```
/codex-review:plan
```

Creates or discusses a plan, then sends it to Codex for iterative review.

### Review code changes

```
/codex-review:code
```

Gathers the git diff from your current session and sends it to Codex for review. Claude automatically fixes issues Codex finds.

### Specify a different model

Both skills accept a model override as an argument:

```
/codex-review:plan o4-mini
/codex-review:code o4-mini
```

## Configuration

- **Default model:** `gpt-5.3-codex` (override via arguments)
- **Max rounds:** 5
- **Codex sandbox:** read-only (Codex can read your codebase for context but cannot modify files)

## License

MIT
