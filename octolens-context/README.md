# Octolens Context Plugin

Claude Code plugin for querying Octolens customer data, billing, mentions, and analytics.

## Install

```bash
claude plugin add /path/to/octolens-context
# or from GitHub:
claude plugin add https://github.com/nicoepp/octolens-context
```

## Setup

Set the `OCTOLENS_TOKEN` environment variable with your admin API key:

```bash
export OCTOLENS_TOKEN="your-token-here"
```

Add it to your shell profile (`~/.zshrc` or `~/.bashrc`) to persist across sessions.

## What's included

**MCP Server** — Connects to the remote Octolens context API with these tools:

| Tool | Description |
|---|---|
| `product_db_schema` | Get Prisma schema for specific tables |
| `product_db_query` | Read-only SQL against PlanetScale (MySQL) |
| `product_tb_schema` | Get Tinybird datasource schema |
| `product_tb_query` | Read-only SQL against Tinybird (ClickHouse) |
| `customers_find` | Search organizations by name, email, or slug |
| `onboarding_recording` | Get PostHog session recording URL |
| `impersonate_org` | Generate a one-time sign-in URL |

**Skill** — `octolens-context` skill triggers automatically when you ask about Octolens customers, plans, mentions, or analytics.

## Usage

Just ask naturally:

- "What plan is Acme on?"
- "How many mentions did org_xxx get last month?"
- "Show me the onboarding recording for Vercel"
- "Let me sign in as Acme"
