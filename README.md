# codex-review

A Claude Code plugin that sends your implementation plan to OpenAI Codex CLI for iterative review. Claude and Codex go back-and-forth refining the plan until Codex approves it.

## How It Works

1. You create a plan in Claude Code (via plan mode or discussion)
2. Run `/codex-review:codex-review` to send it to Codex for review
3. Codex reviews for correctness, risks, missing steps, alternatives, and security
4. Claude automatically revises the plan based on Codex's feedback
5. The loop continues (up to 5 rounds) until Codex approves

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

1. Start Claude Code and create or discuss an implementation plan
2. Run the skill:

```
/codex-review:codex-review
```

3. Optionally specify a different model:

```
/codex-review:codex-review o4-mini
```

4. Watch as Claude and Codex iterate on your plan until it's approved

## Configuration

- **Default model:** `gpt-5.3-codex` (override via arguments)
- **Max rounds:** 5
- **Codex sandbox:** read-only (Codex can read your codebase for context but cannot modify files)

## License

MIT
