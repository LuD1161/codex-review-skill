<div align="center">

# codex-review

**Let Claude and Codex review each other's work.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-blueviolet)](https://code.claude.com/docs/en/plugins)
[![OpenAI Codex](https://img.shields.io/badge/OpenAI-Codex_CLI-412991)](https://github.com/openai/codex)
[![Blog Post](https://img.shields.io/badge/Blog-Read_the_story-orange)](https://aseemshrey.in/blog/claude-codex-iterative-plan-review/)

---

A Claude Code plugin that sends your plans and code to OpenAI Codex CLI for **iterative, multi-round review**. Claude and Codex go back-and-forth — fixing, refining, and improving — until Codex approves.

[Installation](#installation) · [Usage](#usage) · [How It Works](#how-it-works) · [Configuration](#configuration)

</div>

## Skills

| Skill | What it does |
|:--|:--|
| `/codex-review:plan` | Review an implementation plan. Claude revises based on Codex feedback until approved. |
| `/codex-review:code` | Review code changes from the current session. Claude fixes issues until approved. |

## How It Works

<p align="center">
<img src="https://aseemshrey.in/static/a82ed27bb0a18671cbf0f78bb4a0826b/3ed24/codex-review-loop.png" alt="Codex Review Loop" width="600" />
</p>

## Prerequisites

- [**Claude Code**](https://docs.anthropic.com/en/docs/claude-code) — installed and authenticated
- [**OpenAI Codex CLI**](https://github.com/openai/codex) — `npm install -g @openai/codex`
- **OpenAI API key** — configured for Codex

## Installation

**From a marketplace**

```bash
/plugin install codex-review@<marketplace-name>
```

**Local / development**

```bash
claude --plugin-dir /path/to/codex-review-skill
```

## Usage

### Review a plan

```
/codex-review:plan
```

Send your implementation plan to Codex. Claude automatically revises the plan based on feedback until Codex approves.

### Review code changes

```
/codex-review:code
```

Gather the git diff from your session and send it to Codex. Claude automatically fixes issues Codex finds.

### Override the model

Both skills accept a model argument:

```
/codex-review:plan o4-mini
/codex-review:code o4-mini
```

## Configuration

| Option | Default | Description |
|:--|:--|:--|
| Model | `gpt-5.3-codex` | Override per-invocation via arguments |
| Max rounds | `5` | Prevents infinite review loops |
| Sandbox | `read-only` | Codex can read your codebase but never writes |

## Project Structure

```
codex-review-skill/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/
│   ├── plan/
│   │   └── SKILL.md         # Plan review skill
│   └── code/
│       └── SKILL.md         # Code review skill
├── LICENSE
└── README.md
```

## Contributing

PRs welcome! If you have ideas for new review skills or improvements to the review prompts, open an issue or submit a pull request.

## License

[MIT](LICENSE)

---

<div align="center">
<sub>Built with Claude Code + OpenAI Codex — because two heads are better than one.</sub>
</div>
