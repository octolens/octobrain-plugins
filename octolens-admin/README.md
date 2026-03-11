# Octolens Admin Plugin

Claude Code plugin for administering Octolens organizations — extend trials, manage mention credits, keywords, and company info.

## Prerequisites

Install the [octolens-context](https://github.com/nicoepp/octolens-context) plugin first — the admin plugin uses `customers_find` from the context MCP server to resolve organization names.

## Install

```bash
claude plugin add /path/to/octolens-admin
# or from GitHub:
claude plugin add https://github.com/nicoepp/octolens-admin
```

## Setup

Set the `OCTOLENS_TOKEN` environment variable with your admin API key:

```bash
export OCTOLENS_TOKEN="your-token-here"
```

Add it to your shell profile (`~/.zshrc` or `~/.bashrc`) to persist across sessions.

## What's included

**MCP Server** — Connects to the remote Octolens admin API with these tools:

| Tool | Description |
|---|---|
| `add_mentions` | Add or subtract mention credits |
| `extend_trial` | Extend an organization's trial period |
| `reset_mentions` | Reset mention count and billing cycle |
| `add_keyword` | Add a keyword to monitor |
| `delete_keyword` | Delete a keyword and all related data |
| `update_company_name` | Update the company name for an org |

**Skill** — `octolens-admin` skill triggers automatically when you ask to modify an Octolens customer's account.

## Usage

Just ask naturally:

- "Extend the Acme trial by 7 days"
- "Give org_xxx 500 more mentions"
- "Add keyword 'my brand' to Acme on reddit"
- "Change Acme's company name to Acme Corp"
