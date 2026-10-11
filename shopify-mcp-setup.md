---
title: Shopify MCP for Claude Code
summary: Wire a Shopify store into Claude Code with an MCP server, so your AI assistant can read and manage products, orders, customers, and blog posts.
best_for: Store owners and agencies who want an AI assistant wired into their Shopify Admin
difficulty: intermediate
setup_time: ~1 hour
version: 1.0
last_tested: 2026-10-10
tested_with: Claude Opus 4.8 in Claude Code (shopify-mcp v1.0.8)
author: OSEM Dynamics
license: MIT
---

# Shopify MCP Setup Guide (for Claude Code)

> **Instructions for Claude Code:** This document describes how to connect a Shopify store to Claude Code
> through an MCP server. When the user points you at this file, follow "Setup procedure" to wire up a new store.
> Never invent credentials. Ask the user for the store domain, Client ID and Client Secret. Never print or
> commit the Client Secret.

**First step:** ask the user which website/store this setup is for (store name and its `*.myshopify.com` domain), and use a project folder named after it, e.g. `{store}-mcp`. Everything below uses `{store}` as a placeholder.

---

## 1. Architecture (how it works)

```
Claude Code  <--stdio-->  shopify-mcp (local Node process)  --HTTPS GraphQL-->  Shopify Admin API
                                  |
                                  +-- OAuth client_credentials --> https://{store}.myshopify.com/admin/oauth/access_token
```

- **MCP server:** the open-source npm package [`shopify-mcp`](https://www.npmjs.com/package/shopify-mcp) (v1.0.8 in use). It is
  not custom code. There is nothing to build; you install it and configure it.
- **Transport:** stdio. Claude Code launches the `shopify-mcp` process itself from `.mcp.json`.
- **Stack inside the package:** Node (ESM), `@modelcontextprotocol/sdk`, `graphql-request`, `zod`, `minimist`, `dotenv`.
- **API:** Shopify Admin **GraphQL** API, default version `2026-01` (override with `--apiVersion`).
- **Auth:** OAuth *client credentials grant*. The server exchanges Client ID + Client Secret for a short-lived
  Admin API token (about 24h), caches it, and refreshes it automatically 5 minutes before expiry. You never handle a token.
  (Legacy alternative: a static `shpat_...` token via `--accessToken`. Shopify stopped issuing these in the UI on 2026-01-01.)
- **One store per server entry.** Each Shopify store is a separate Dev Dashboard app + a separate MCP entry.

### Tools the server exposes
| Area | Tools |
|---|---|
| Products | `get-products`, `get-product-by-id`, `create-product`, `update-product`, `delete-product`, `manage-product-variants`, `delete-product-variants`, `manage-product-options` |
| Orders | `get-orders`, `get-order-by-id`, `update-order` |
| Customers | `get-customers`, `get-customer-orders`, `update-customer` |
| Blog | `get-blogs`, `get-blog-articles`, `create-blog-article` |

### Known limits (discovered in real use)
- No tools for: themes/theme files, pages, collections, menus, metafields, files/images, discounts, inventory, redirects.
  Theme and Liquid changes are done by hand in **Online Store > Themes > Edit code**. Claude writes the code and the user pastes it in.
- `create-blog-article` cannot set the SEO title/description or the featured image. Do those in Shopify Admin
  (or use a theme snippet for structured data, see section 6).
- There is no generic "run any GraphQL" tool, so anything outside the table above is not possible without extending or forking the package.

---

## 2. Prerequisites

- Node.js 18+ (Node 22 via nvm works well)
- Claude Code
- Owner or staff access to the Shopify store that can create apps

---

## 3. Shopify side: create the app (once per store)

1. Shopify admin > **Settings > Apps and sales channels > Develop apps > Build app in dev dashboard**
   (or go to https://dev.shopify.com/dashboard directly).
2. Create a new app (name it e.g. `Claude MCP`).
3. Configure **Admin API access scopes**. Minimum for the tools above:
   - `read_products`, `write_products`
   - `read_customers`, `write_customers`
   - `read_orders`, `write_orders`
   - `read_content`, `write_content` (required for the blog tools; shown as "Online Store" in the store admin)
   Only grant what you need. Drop `write_*` scopes for read-only use.
4. **Install** the app on the store (a release/version must be created and the app installed, or token exchange will fail).
5. From the app's **Settings / API credentials**, copy:
   - **Client ID**
   - **Client Secret** (starts with `shpss_`)
6. Note the store's `*.myshopify.com` domain (Settings > Domains). Use this, not the custom domain.

### What the app looks like (reference)
- **App type:** Dev Dashboard app (dev.shopify.com/dashboard), named **Claude MCP**, distributed to a single store (not listed in the App Store; Shopify shows an "unlisted app" notice, which is normal).
- **Versions:** each scope change is a new app version (`claude-mcp-1`, `claude-mcp-2`, ...). The newest one is marked **Active**. Installs: 1.
- **Credentials panel (Dev Dashboard > app > Overview):** Client ID and Client secret, each with a copy button. These are the two values the MCP needs.
- **Permissions as shown in the store admin** (Settings > Apps > Claude MCP > Activity and permissions), all with View + Edit:

  | Admin area | Scopes it maps to |
  |---|---|
  | Products | `read_products`, `write_products` |
  | Orders | `read_orders`, `write_orders` |
  | Customers | `read_customers`, `write_customers` |
  | Online Store | `read_content`, `write_content` (blogs and articles). May also include theme scopes; the MCP does not use themes. |

- The admin screen shows areas, not raw scope names. To see the exact scope strings, open the active version in the Dev Dashboard (Versions > latest active version) and copy them into this table.
- Because the app can read customers and orders, Shopify flags it as accessing personal data (names, emails, phones, addresses). For a new store, drop Customers/Orders if you only need products and blog content.

### Changing scopes later
Dev Dashboard > app > **Versions > New version**, edit the scopes, release it, then approve the update in the store admin (Apps > Claude MCP). Restart Claude Code afterwards so the server fetches a fresh token with the new scopes.

---

## 4. Local side: install the server

```bash
npm install -g shopify-mcp
which shopify-mcp        # note this absolute path
```

Using the absolute path in `.mcp.json` avoids "command not found" when Claude Code is launched without the nvm
environment (recommended). `npx` also works: `"command": "npx", "args": ["-y", "shopify-mcp", ...]`.

---

## 5. Connect it to Claude Code

### Option A: project-scoped `.mcp.json` (simplest)
Create `.mcp.json` in the project folder (one folder per store keeps stores isolated):

```json
{
  "mcpServers": {
    "shopify": {
      "command": "/absolute/path/to/shopify-mcp",
      "args": [
        "--domain={store}.myshopify.com",
        "--clientId=YOUR_CLIENT_ID",
        "--clientSecret=YOUR_CLIENT_SECRET"
      ]
    }
  }
}
```

### Option B: CLI
```bash
claude mcp add shopify -- npx -y shopify-mcp \
  --clientId YOUR_CLIENT_ID --clientSecret YOUR_CLIENT_SECRET --domain {store}.myshopify.com
```

### Option C (recommended): keep the secret out of the file
`shopify-mcp` reads env vars and a `.env` in its working dir. Use `${VAR}` expansion in `.mcp.json`:

```json
{
  "mcpServers": {
    "shopify": {
      "command": "npx",
      "args": ["-y", "shopify-mcp"],
      "env": {
        "MYSHOPIFY_DOMAIN": "{store}.myshopify.com",
        "SHOPIFY_CLIENT_ID": "${SHOPIFY_CLIENT_ID}",
        "SHOPIFY_CLIENT_SECRET": "${SHOPIFY_CLIENT_SECRET}"
      }
    }
  }
}
```
and export the two variables in your shell profile (or per-store `.env` that you load).

### Then
1. Restart Claude Code in that folder. Approve the project MCP server when prompted.
2. Run `/mcp` and confirm `shopify` shows **connected**.
3. Test: *"List my 5 most recent products."*

### Multiple stores
Either (a) one project folder per store, each with its own `.mcp.json` (simplest, clear separation), or
(b) several entries in one `.mcp.json` named `shopify-storeA`, `shopify-storeB`, each with its own domain and credentials.
Prefer (a) so an instruction cannot accidentally hit the wrong store.

---

## 6. Suggested project layout

```
{store}-mcp/
  .mcp.json                      # MCP connection (secret!)
  .claude/settings.local.json    # permissions (WebFetch of own domain, WebSearch)
  blog-posts/                    # post-{n}-{slug}.html source files + WORKFLOW.md + CONTENT-ROADMAP.md
  theme-snippets/                # Liquid snippets to paste into the theme (blog-article-schema.liquid)
  theme-changes/                 # change logs: file, line range, what was added, how to revert
  seo-llmo-*.md                  # SEO / LLM-optimisation strategy notes
```

Conventions worth reusing:
- Content lives in local files, Claude publishes it through the MCP, so the repo is the source of truth.
- Each manual theme edit gets a note in `theme-changes/` with file, location, code and **how to revert**.
- A `WORKFLOW.md` with a ready prompt, e.g. *"Push the blog posts in blog-posts/ that haven't been published yet."*

### Blog publishing workflow (summary)
1. Write `blog-posts/post-{n}-{slug}.html` with Article + FAQPage JSON-LD in `<head>` and the body inside `<article>`.
2. Claude calls `get-blogs` to find the blog GID, then `create-blog-article` with `blogId`, `title`, `contentHtml`
   (the `<article>` contents **without the `<h1>`**, since the theme renders the title), `author`, `handle`, `summary`, `tags`, `published`.
3. Add a `{%- when 'handle' -%}` block to `snippets/blog-article-schema.liquid` for the structured data, then paste the snippet into the theme.
4. Manually add the featured image in Shopify Admin.
5. Validate with Google's Rich Results Test.

---

## 7. Setup procedure for Claude Code (checklist)

When the user asks to set up a new store with this guide:

1. Confirm Node is installed and `shopify-mcp` is installed globally (`which shopify-mcp`); install it if missing.
2. Ask for: store `*.myshopify.com` domain, Client ID, Client Secret. Confirm the app is installed and has the scopes in section 3.
3. Create `.mcp.json` in the target project folder (Option A or C). Prefer C, or A plus `.gitignore` entry for `.mcp.json`.
4. Tell the user to restart Claude Code and run `/mcp`.
5. Verify with a harmless read call (`get-products` limit 1). Do not run write tools as a test.
6. Create the folder layout from section 6 if the user wants content workflows.

---

## 8. Troubleshooting

| Symptom | Likely cause / fix |
|---|---|
| `Shopify token exchange failed (400/401)` | Wrong Client ID/Secret, app not installed on that store, or wrong domain (must be `*.myshopify.com`). |
| `Authentication credentials are required` | Args/env not reaching the process; check `.mcp.json` syntax and that the flags are `--clientId` / `--clientSecret` / `--domain`. |
| `Access denied` / missing scope in a GraphQL error | Add the scope in the Dev Dashboard, create a new app version, and re-approve the install. Restart the MCP. |
| `command not found: shopify-mcp` | Use the absolute path from `which shopify-mcp`, or `npx -y shopify-mcp`. |
| "SHOPIFY_ACCESS_TOKEN required" | You installed `shopify-mcp-server` (different package). Use `shopify-mcp`. |
| Server not listed in `/mcp` | Project MCP not approved or Claude Code not restarted in that folder. |
| Deprecated API version warnings | Pass `--apiVersion=YYYY-MM` (default `2026-01`). |

---

## 9. Security notes

- The **Client Secret grants admin API access** at the granted scopes. Treat it like a password.
- Do not commit `.mcp.json` with a literal secret. Add it to `.gitignore` or use Option C.
- If a project's `.mcp.json` holds the real Client Secret and that folder has ever been shared, synced to a public repo, or pasted into a chat, rotate the secret in the Dev Dashboard.
- Use least-privilege scopes, and a separate app per store.
