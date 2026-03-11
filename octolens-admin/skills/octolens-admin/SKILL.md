---
name: octolens-admin
description: Perform admin write operations on Octolens organizations — extend trials, manage mention credits, manage keywords, and update company info. Use this skill when the user asks to perform any write action on an Octolens customer.
allowed-tools: mcp__octolens-admin__add_mentions, mcp__octolens-admin__extend_trial, mcp__octolens-admin__reset_mentions, mcp__octolens-admin__add_keyword, mcp__octolens-admin__delete_keyword, mcp__octolens-admin__update_company_name, mcp__octolens-context__customers_find
---

# Octolens Admin

Write operations for Octolens admin tasks. All tools require a Clerk organization ID (starts with `org_`).

> **Requires the `octolens-context` plugin** — the `customers_find` tool used to resolve org names comes from the context MCP server.

## Resolving organizations

When the user refers to an org by **name, email, or slug** (not an org ID), resolve it first using `customers_find`.

- Multiple matches → show them all, ask the user to confirm.
- One match → proceed with its id (starts with `org_`).
- Zero matches → tell the user, ask to clarify.

## Actions

### Add mentions

Use the `add_mentions` tool. Increment can be negative to subtract. Floor is 0. Returns the new total.

### Extend trial

Use the `extend_trial` tool. Days must be positive. Returns the new expiration timestamp.

### Reset mentions

Use the `reset_mentions` tool. Resets the mention count to 0 and restarts the billing cycle from now.

### Add keyword

Use the `add_keyword` tool. Provide the keyword string and optionally a list of platforms (e.g. `["reddit", "twitter"]`). Returns 409 if the keyword already exists.

### Delete keyword

Use the `delete_keyword` tool. Requires the keyword ID (integer). Deletes the keyword and all related data (posts, tags, webhook/slack executions). Returns the count of deleted posts.

### Update company name

Use the `update_company_name` tool. Returns 404 if the org has no company linked.

## Examples

**"Add 1 week to the Acme trial"**
1. `customers_find` with query "acme" → confirm which org if multiple
2. `extend_trial` with organizationId and days=7

**"Give org_xxx 500 more mentions"**
1. `add_mentions` with organizationId="org_xxx" and increment=500

**"Remove 100 mentions from Acme"**
1. `customers_find` with query "acme" → get org ID
2. `add_mentions` with organizationId and increment=-100

**"Reset mentions for Acme"**
1. `customers_find` with query "acme" → get org ID
2. `reset_mentions` with organizationId

**"Add keyword 'my brand' to Acme on reddit and twitter"**
1. `customers_find` with query "acme" → get org ID
2. `add_keyword` with organizationId, keyword="my brand", platforms=["reddit", "twitter"]

**"Delete keyword 42 from org_xxx"**
1. `delete_keyword` with organizationId="org_xxx" and keywordId=42

**"Change Acme's company name to Acme Corp"**
1. `customers_find` with query "acme" → get org ID
2. `update_company_name` with organizationId and name="Acme Corp"
