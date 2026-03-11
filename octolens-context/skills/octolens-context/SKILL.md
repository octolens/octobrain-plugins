---
name: octolens-context
description: Query Octolens customer data, look up billing/plan info, check mention analytics, debug keyword configs, and investigate API usage. Use this skill whenever the user asks about an Octolens customer, organization, subscription, mentions, keywords, notifications, API events, or any read-only lookup — even if they just mention a company name in the context of Octolens support.
allowed-tools: mcp__octolens-context__product_db_schema, mcp__octolens-context__product_db_query, mcp__octolens-context__product_tb_schema, mcp__octolens-context__product_tb_query, mcp__octolens-context__customers_find, mcp__octolens-context__onboarding_recording, mcp__octolens-context__impersonate_org
---

# Octolens Context

Read-only access to Octolens customer data via MCP tools.

## Data routing

Octolens has two backends. Using the wrong one gives wrong or empty results:

| Data you need | Tool | Backend |
|---|---|---|
| Mentions / posts (counts, trends, content, sources) | `product_tb_query` | Tinybird (ClickHouse) — time-series event data at scale |
| API / MCP usage events | `product_tb_query` | Tinybird (ClickHouse) — event stream with 90-day TTL |
| Everything else (settings, plan, keywords config, company, notifications, views, billing, orgs) | `product_db_query` | PlanetScale (MySQL) — config, settings, relationships |

## Query workflow

1. **Get the schema** for the backend you need:
   - `product_db_schema(tables: ["Settings", "Keyword", ...])` → Prisma schema for specific tables and their related enums. The response also lists all available table names.
   - `product_tb_schema` → Tinybird datasource schema (posts | keywords | author_followers | api_events)
2. **Construct your query** using `product_db_query` or `product_tb_query`. Both are read-only.
3. **Iterate** if the first query doesn't give you what you need — refine, join other tables, or check the schema again.

## Resolving organizationId to name

To get the human-readable name for an organizationId, query the Settings table:
`SELECT organizationId, name FROM Settings WHERE organizationId = 'org_xxx'`
Always do this when returning results that include organizationId — users don't know org IDs.

## Tinybird gotchas

- Always filter `_is_deleted = 0` on posts (soft deletes).
- Tables use `ReplacingMergeTree` — for exact counts, use `FINAL` or `argMax(col, updatedAt) ... GROUP BY organizationId, id`.
- Array columns (like `keywordIds`, `tags`) need array functions: `hasAny()`, `has()`, `arrayJoin()`.
- `api_events` uses underscore naming (`organization_id`, not `organizationId`).
- `api_events` has a 90-day TTL — older data is gone.

## Resolving organizations

When the user refers to an org by **name, email, or slug** (not an org ID), resolve it first using `customers_find`.

- Multiple matches → show them all, ask the user to confirm.
- One match → proceed with its id (starts with `org_`).
- Zero matches → tell the user, ask to clarify.

Skip this step entirely when the request doesn't need a specific org (e.g., "how many orgs are on the Pro plan?" or "which sources have the most mentions globally?").

To go from ID → name, query: `SELECT organizationId, name FROM Settings WHERE organizationId = 'org_xxx'`.

## Examples

**"What plan is Vercel on?"**
1. `customers_find` with query "vercel" → get org ID
2. `product_db_query` with `SELECT plan, ... FROM Settings WHERE organizationId = 'org_xxx'`

**"How many mentions did org_xxx get from Reddit last month?"**
1. `product_tb_query` with `SELECT count() FROM posts FINAL WHERE organizationId = 'org_xxx' AND _is_deleted = 0 AND source = 'reddit' AND timestamp > now() - INTERVAL 30 DAY`

**"Which orgs have the most mentions?"**
1. `product_tb_query` with `SELECT organizationId, count() as cnt FROM posts FINAL WHERE _is_deleted = 0 GROUP BY organizationId ORDER BY cnt DESC LIMIT 10`
2. `product_db_query` with `SELECT organizationId, name FROM Settings WHERE organizationId IN ('org_aaa', ...)` — resolve names via Settings, not with customers_find

**"Check API usage for org_xxx"**
1. `product_tb_query` with `SELECT endpoint, count() as calls, avg(duration_ms) as avg_ms FROM api_events WHERE organization_id = 'org_xxx' GROUP BY endpoint ORDER BY calls DESC`

**"Show me the onboarding recording for Acme"**
1. `customers_find` with query "acme" → get org ID
2. `onboarding_recording` with organizationId → returns PostHog replay URL

**"Let me sign in as Acme"**
1. `customers_find` with query "acme" → get org ID
2. `impersonate_org` with organizationId → returns one-time Clerk sign-in URL (valid 10 min)
