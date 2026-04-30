---
name: risify
description: >
  Complete skill for the Risify Shopify SEO app. Covers all Risify features and workflows.
  Trigger when the user wants to: generate FAQs, manage FAQs, import FAQs from CSV/Excel,
  bulk-add FAQs, upload FAQ files, view account info, check AI credits,
  manage subscription/billing/plans, add/edit/remove team contacts, view billing history,
  cancel subscription, upgrade plan, assign FAQs to products or collections,
  manage breadcrumbs, set up collection menus, configure related searches,
  generate AI navigation recommendations, activate navigation features, sync collections for AI,
  or any Risify-related task. Covers questions like: "generate FAQs for my products",
  "import FAQs", "upload FAQs from CSV", "bulk add FAQs", "add FAQs from file",
  "import questions and answers", "add these FAQs to my collections",
  "what plan am I on", "how many credits do I have", "add a team member",
  "show my billing history", "cancel my subscription", "list my FAQs",
  "set up breadcrumbs", "suggest breadcrumbs with AI", "add collection menu",
  "configure related searches", "generate navigation recommendations".
  Uses domain-specific MCP tools where available, falling back to execute_graphql for advanced operations.
---

# Risify MCP Skill

Complete workflow guide for the Risify Shopify SEO platform.

**Domain-specific tools** handle common workflows in a single call. For advanced operations not covered by a domain tool, use `execute_graphql` with raw GraphQL (Risify API direct, or Shopify Admin API via `shopifyProxy`).

## Domain Tools Available

| Tool | Purpose |
|------|---------|
| `check_credits` | Check AI credit balance |
| `get_account_info` | Account, store, and subscription details |
| `list_products` | Browse store products (returns GIDs for other tools) |
| `list_collections` | Browse store collections (returns GIDs for other tools) |
| `generate_faqs` | Generate AI FAQ content for review |
| `create_and_assign_faqs` | Save FAQs and assign to products/collections |
| `list_faqs` | View existing FAQ entries |
| `generate_meta_tags` | Generate SEO meta titles and descriptions |
| `run_seo_audit` | Run or view SEO audit results |
| `generate_navigation` | Generate AI breadcrumb/menu/search recommendations |
| `check_navigation_sync` | Check/trigger collection sync for AI navigation |

## Available Flows

| Flow | Trigger | Reference |
|------|---------|-----------|
| FAQ Generation & Assignment | User wants to generate, create, list, update, delete, or assign FAQs | `references/faq.md` + `references/faq-operations.md` |
| Account Management | User asks about account, billing, plans, contacts, credits, subscription | `references/account.md` + `references/account-operations.md` |
| Navigation | User wants breadcrumbs, collection menus, related searches, AI navigation suggestions, feature activation | `references/navigation.md` + `references/navigation-operations.md` |

## How to Use

1. **Use domain tools first** — most common operations have a dedicated tool
2. Match complex requests to a flow above and load the reference files
3. Fall back to `execute_graphql` only for operations without a domain tool (contacts, billing, manual metafield writes, feature activation, etc.)

## Quick Reference

### Common Operations (use domain tools)

**Check AI Credits:** `check_credits` tool (no parameters)

**Get Account Info:** `get_account_info` tool (no parameters)

**List Products:** `list_products` tool (optional: `first`, `after`, `query`)

**List Collections:** `list_collections` tool (optional: `first`, `after`, `query`)

**Generate FAQs:** `generate_faqs` tool with `resourceGIDs`, `count`, optional `language` and `tone`

**Save FAQs:** `create_and_assign_faqs` tool with `items` [{question, answer}] and `resourceGIDs`

**Run SEO Audit:** `run_seo_audit` tool (optional: `force_new`)
