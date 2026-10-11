---
title: Launch-Ready SEO + LLMO Setup
summary: Make a new or migrated site findable by search engines and AI assistants - the on-site fixes plus the off-site setup (Search Console, Bing, GA4).
best_for: Small businesses and solo builders who just launched or migrated a site (often off Shopify or Wix) and want it indexed and cited by AI
difficulty: intermediate
setup_time: ~2-3 hours
version: 1.0
last_tested: 2026-10-11
tested_with: Claude Opus 4.8 in Claude Code (on osemdynamics.com)
author: OSEM Dynamics
license: MIT
---

# Launch-Ready: SEO + LLMO Setup

## What this builds

Takes a website that already exists - new, redesigned, or migrated from another
platform - and makes it findable by search engines **and** AI assistants
(ChatGPT search, Perplexity, Copilot, Google AI Overviews, Grok).

Your AI assistant will:

1. **Audit** the live site the way a crawler and an answer engine see it.
2. **Fix the code side** - metadata, canonicals, sitemap, structured data, llms.txt,
   crawler access, IndexNow.
3. **Walk you through the off-site setup** one screen at a time - Google Analytics,
   Search Console, Bing Webmaster Tools, Google Business Profile, and retiring any
   old platform that's still haunting search results.
4. **Score** the result and leave you a short list of what's left.

Most sites skip everything after "it's deployed." That gap is why AI assistants
still describe businesses using their old site, old platform, or nothing at all.

## Who it's for / not for

- **For:** small businesses, studios, and solo builders who just launched or
  migrated a site (especially off Shopify, Wix, Squarespace, or WordPress) and
  want it properly indexed and citable by AI.
- **Not for:** large sites with thousands of pages, international/multi-language
  SEO, or paid search campaigns. That's an SEO engagement, not a blueprint.

## Prerequisites

- A live site on a domain you control, and access to its **code and deploy**.
- Access to your **DNS provider** (GoDaddy, Cloudflare, Namecheap, etc.).
- A **Google account** you want to own the business's analytics - ideally a
  Workspace account on your domain, not a personal Gmail.
- A **Microsoft account** for Bing Webmaster Tools.
- 2-3 hours, mostly clicking through consoles with your assistant guiding you.

---

## Instructions for the AI assistant

> You are doing this **with** the user. Code changes you make directly. Console
> steps (Google, Bing, DNS, old platform) the user does - you guide them one screen
> at a time, ask for a screenshot or confirmation, and only then give the next step.
> Follow the hard rules at the bottom. Explain why each step matters in one line.

### 1. Ask before starting

Ask and wait for answers:

- What's the domain, and what framework/hosting runs it? (Next.js, Astro, WordPress, static, etc.)
- Was the site **previously on another platform**? Which one, and is that account still open?
- Where is DNS managed?
- Is this a **local / service-area business** (should appear on Google Maps) or online-only?
- Do you already have Google Analytics, Search Console, or Bing set up for this or any other project?
  (Existing Firebase projects silently create a GA account - see trap T1.)

### 2. Audit (read-only)

Fetch the live site as a crawler would. Do **not** change anything yet. Check and record:

**Indexing reality**
- Search `site:yourdomain.com` and the brand name. Is the site indexed at all? Is old-platform content showing?
- Do old-platform URLs (e.g. `/products/`, `/collections/`, `/pages/`) return 404/410 or redirect sensibly?
- Does `www` redirect to the apex (or vice versa) and `http` to `https`, in one hop?

**Per page** (home, every main section, and at least one page of each dynamic route)
- `<title>` unique and descriptive.
- Meta description unique, **120-155 characters**.
- **Self-referencing canonical.** Check dynamic routes specifically - a common bug is
  every child page canonicalizing to its parent index page (trap T6).
- `og:url`, `og:title`, `og:description` match the page, not the homepage.
- Exactly **one `<h1>`**. Watch for typewriter/cursor effects rendering a second empty one.
- Content present in the server HTML, not only after JavaScript runs.
- No splash/gate overlay text ("click to enter") in the crawlable text.

**Site-wide**
- `robots.txt`: allows crawling, lists the sitemap, doesn't block AI crawlers
  (GPTBot, ClaudeBot, PerplexityBot, Google-Extended) unless the user wants that.
  No nonstandard junk (`Host:` lines).
- `sitemap.xml`: valid, contains only indexable pages (no login/portal/admin),
  `lastmod` reflects real changes rather than one build timestamp for every URL.
- Structured data: `Organization` / `ProfessionalService` (or `LocalBusiness`) with
  name, logo, url, address/area, email, `sameAs`; `FAQPage` where there are Q&As.
- `llms.txt` at the root.
- Analytics installed?
- Copy consistency: the same promises (response time, team size, prices) everywhere.

Present the audit as a short prioritized list: **broken → missing → nice-to-have.**
Get the user's go-ahead before fixing.

### 3. Fix the code side

Adapt to the framework. For each fix, state what it fixes, then do it.

1. **Per-page metadata:** unique title, description (120-155 chars), self-referencing
   canonical, matching OG/Twitter tags on every route, including dynamic ones.
   *(Next.js: `generateMetadata` per route, `alternates.canonical`.)*
2. **Headings:** one `<h1>` per page; decorative effects must not create extra headings.
3. **Sitemap:** indexable pages only; real `lastmod` dates if available, otherwise omit
   `lastmod` rather than faking it.
4. **robots.txt:** allow crawling, disallow admin/api/portal, reference the sitemap.
5. **Noindex** private areas (login, portal, account pages) and remove them from the sitemap.
6. **Structured data:** Organization/LocalBusiness JSON-LD site-wide; `FAQPage` on Q&A
   content; leave `sameAs` as a placeholder array for profiles created later.
7. **llms.txt:** a plain-text summary of who the business is, what it does, its key
   pages, and its FAQ answers inline. Link raw `.md` versions of long-form pages if
   the site has them.
8. **Analytics:** install the GA4 tag in the root layout, loading after the page is
   interactive. Leave the Measurement ID as a config value - the user creates it in step 4.
9. **IndexNow:** generate a key, serve it at `/{key}.txt`, and ping `api.indexnow.org`
   with the sitemap URLs on every deploy.
10. **Deploy, then verify from outside** - fetch the live `sitemap.xml` and a few pages
    and confirm the changes are actually served (trap T7).

### 4. Guide the off-site setup

One step at a time. Ask for a screenshot or "done" before moving on.

**4a. Google Analytics 4**
- analytics.google.com → Admin → **Create → Account** (a new account, named for the
  business - not a property inside an existing account). → property → Web stream.
- Data-sharing checkboxes: defaults are fine; "Google products & services" can stay off.
- Objectives: *Generate leads* + *Understand web traffic* for most service businesses.
- Copy the **Measurement ID** (`G-…`) with the copy button, never retyped. Add it to the config and deploy.
- Verify in **Reports → Realtime**, not the "data collection isn't active" banner (trap T3).

**4b. Google Search Console**
- Add a **Domain** property (not URL-prefix). Verify via the DNS provider's
  one-click flow if offered, otherwise a TXT record at `@`. Domain properties cover
  every protocol and subdomain and survive site rebuilds.
- Sitemaps → submit `sitemap.xml`.
- URL Inspection → **Request indexing** for the homepage and 4-6 key pages, even
  if they already show "indexed" - the indexed copy may be from the old site.
- In GA4: Admin → Product links → **Search Console links** → link the property.

**4c. Bing Webmaster Tools**
- bing.com/webmasters → **Import from Google Search Console**.
- Confirm the sitemap was imported; submit it if not.
- URL Inspection on the homepage. Check the **Live URL** tab to confirm Bingbot can fetch it,
  then Request indexing. Submit key URLs via URL Submission.
- Run **Site Scan** and fix anything it flags.
- Bing matters disproportionately: its index feeds ChatGPT search, Copilot, and DuckDuckGo.

**4d. Retire the old platform** (if there was one)
- Confirm which store/site in the old platform's admin before changing anything -
  users often have several under one login.
- **Deactivate the old site directly. Do not remove the custom domain first** (trap T4).
- Export any data worth keeping (orders, customers) before deactivating.
- At the DNS provider, delete leftover platform records (e.g. Shopify's
  `23.227.38.65` A record, `shops.myshopify.com` CNAMEs) for every subdomain.

**4e. Google Business Profile** (local / service-area businesses only)
- business.google.com, signed in with the same Google account.
- **Name:** the plain legal/real-world name - no ™, ®, or keywords.
- No customer-facing location → **service-area business**; set realistic nearby areas, not a whole country.
- Phone is optional; a business line (Google Voice, OpenPhone) beats a personal cell.
- Description: up to 750 characters, services + location + areas served, no URLs, no "best/#1".
- **Skip the storefront photo** if there's no storefront - never upload a fake one.
- Verification is usually a single continuous video: street → workspace → proof of business.
  The name on any requested documents must match the business name.
- After verification: logo (square, 250px+), hours, services, real photos.

### 5. Verify and score

Re-fetch the site and confirm every audit item is resolved. Then give the user a
scorecard out of 10 for each:

| Area | What it measures |
|---|---|
| Crawlability | robots, sitemap, server-rendered content, no gates |
| Page signals | titles, descriptions, canonicals, headings, OG |
| Structured data | Organization/LocalBusiness, FAQPage, sameAs |
| AI readiness | llms.txt, answer-shaped content, concrete facts vs. vague claims |
| Indexing | GSC + Bing verified, sitemap accepted, key pages requested |
| Off-site authority | mentions and links from other sites, profiles, reviews |

Expect **off-site authority to be the lowest score** for any new site. End with the
user's remaining list (see Going further).

### 6. Hard rules

- **Read-only audit first.** No code or DNS changes until the user approves the fix list.
- **Never delete DNS records** except ones you've confirmed belong to a retired platform, and say which before the user deletes them.
- **Never remove the verification TXT record** once Search Console is verified.
- **Never fabricate** reviews, photos, addresses, business details, or metrics. Leave a placeholder and ask.
- **Never put credentials or account IDs other than the public GA Measurement ID in code.**
- **Verify from outside after every deploy** - don't trust the build log.
- **If a console step fails twice, stop and ask** rather than guessing at UI that may have changed.

---

## Traps we hit, so you don't

- **T1 - The wrong GA account.** If the user ever created a Firebase project, Google
  auto-created a "Default Account for Firebase" with a `G-` ID. GA opens to it.
  Don't use that ID - create a new account for the website.
- **T2 - "Couldn't fetch" on a fresh sitemap.** Search Console often shows this right
  after submission on a new property. Inspect the sitemap URL → Test live URL. If it's
  reachable, just wait; it flips to Success within hours to days.
- **T3 - GA says "no data received."** Banners and standard reports lag 24-48h.
  Realtime is the real test. Ad blockers and privacy browsers hide your own visit.
- **T4 - Removing the old platform's domain resurrects it.** On Shopify (and similar),
  while your domain is primary, the `*.myshopify.com` URL redirects to your real site.
  Remove the domain first and that URL starts serving the old storefront again. Deactivate instead.
- **T5 - "Indexed" doesn't mean current.** Google may be holding a copy from the old
  platform. Request indexing anyway.
- **T6 - Dynamic routes inheriting the parent canonical.** Child pages (blog posts,
  case studies, docs) pointing their canonical at the index page tell search engines
  they're duplicates. Your best content silently never gets indexed.
- **T7 - Committed isn't deployed.** Confirm changes on the live URL, from outside.
- **T8 - Bing "Discovered but not crawled."** Common for new, low-authority domains.
  Live URL test + Request indexing + IndexNow + a few real backlinks usually fixes it within a week or two.

## Troubleshooting

- **Search Console verification fails:** DNS can take 10-15 minutes. Retry. Confirm
  the TXT record is on the root (`@`), not a subdomain.
- **Realtime shows nothing:** View page source and search for the `G-` ID. If it's
  there, try a browser with no ad blocker. If it's not, the deploy didn't ship it.
- **Old pages still show in Google weeks later:** Search Console → **Removals** for
  stale URLs; make sure they return 404 or 410, not a soft 200.
- **Bing Site Scan flags meta description length:** keep every page at 120-155 characters.

## Going further

- **Backlinks you control:** "Built by…" links from your other projects and client
  sites (with permission), a LinkedIn company page, GitHub org, and directory
  profiles like Clutch. Add them all to `sameAs`.
- **Specifics over claims:** real price ranges, timelines, hardware requirements,
  and metrics. AI assistants quote facts and skip adjectives.
- **Give case studies and long-form content their own URLs** rather than anchors on one page.
- **Check back in a week:** sitemap status, Bing crawl status, first queries in
  Search Console → Performance, and Bing's AI Performance report.

