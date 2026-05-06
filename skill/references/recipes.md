# Cross-Flow Workflow Recipes

Multi-step workflows that chain features together. When a user's request spans multiple features, follow the matching recipe instead of improvising.

---

## Category 1: Audit-Driven Fixes

### Recipe 1: Audit → Fix Meta Issues

**Trigger:** "Find pages with missing meta titles and fix them", "Fix my meta issues from the audit", "Optimize meta tags for pages with SEO problems"

**Steps:**

1. Run or retrieve the latest audit (see `audit.md` Steps 1-5)
2. Drill into meta issues:
   ```graphql
   query { auditMetaIssuesConnection(args: { first: 50, query: "auditId:<audit-id>" }) { nodes { location issue url impact } pageInfo { hasNextPage endCursor totalCount } } }
   ```
3. Filter for issues mentioning "missing", "duplicate", or "too short/long" in the `issue` field
4. Match affected URLs to product/collection GIDs:
   - Extract handle from URL (e.g., `/products/blue-dress` → handle `blue-dress`)
   - Search products: `{ shopifyProductsConnection(args: { first: 5, query: "handle:blue-dress" }) { nodes { id title } } }`
   - Or collections: `{ shopifyCollectionsConnection(args: { first: 5, query: "handle:summer" }) { nodes { id title } } }`
5. Present findings: "{N} pages with meta issues found. Generate new meta tags for these?"
6. If user confirms → follow `meta.md` Steps 3-6 with the collected GIDs
7. After applying → offer: "Want me to re-run the audit to verify the fixes?"

**Flows involved:** Audit → Meta Tags → (optional) Audit

---

### Recipe 2: Audit → Fix Content Gaps

**Trigger:** "My audit shows thin content", "Improve content for low-scoring pages", "Generate descriptions for products missing them"

**Steps:**

1. Run or retrieve the latest audit (see `audit.md`)
2. Check on-page score — if low, content gaps likely
3. List products/collections and identify those with empty or short descriptions:
   ```graphql
   { shopifyProductsConnection(args: { first: 50 }) { nodes { id title handle description } pageInfo { hasNextPage endCursor } } }
   ```
4. Filter for products where `description` is empty or under 50 characters
5. Present: "{N} products have missing or thin descriptions. Generate content for them?"
6. If user confirms → follow `content.md` with `contentType: PRODUCT_DESCRIPTION` for each
7. After applying → offer to re-audit

**Flows involved:** Audit → Content Generation → (optional) Audit

---

### Recipe 3: Audit → Escalate to Support

**Trigger:** "I have too many SEO issues to fix myself", "I need help with these audit problems", "Can someone fix my broken links?"

**Steps:**

1. Run or retrieve the latest audit
2. Summarize critical findings (high-impact issues)
3. Format a support ticket with audit context:
   ```graphql
   mutation { createSupportTicket(input: { topic: "SEO Audit: {highImpactCount} high-impact issues found" message: "Latest audit (health score: {healthScore}/100) found {totalIssues} issues:\n- Broken Links: {count}\n- Meta Issues: {count}\n- Schema Errors: {count}\n\nTop issues:\n{list top 5 high-impact issues with URLs}" }) }
   ```
4. Confirm: "Support ticket created with your audit findings. The Risify team will review and respond."

**Flows involved:** Audit → Support

---

### Recipe 4: Audit → Page Speed → Page Design

**Trigger:** "My page speed is slow", "Improve my Lighthouse scores", "Optimize my page performance"

**Steps:**

1. Run or retrieve the latest audit — focus on `pageSpeedMetrics`
2. Present desktop/mobile performance scores and key metrics (LCP, CLS, FCP, TBT)
3. List PageSpeed opportunities from `pageSpeedMetrics.desktop.opportunities` and `mobile.opportunities`
4. Offer: "I can analyze your page design to generate optimized styles. Which pages should I check?"
5. Collect page URLs from user
6. Follow `pagedesign.md` Steps 2-5 with the affected page URLs
7. Present style configs and CSS optimizations

**Flows involved:** Audit → Page Design

---

## Category 2: Resource-Centric Optimization

### Recipe 5: Full Product SEO Optimization

**Trigger:** "Optimize this product for SEO", "Make this product rank better", "Full SEO for {product name}"

**Steps:**

1. Check AI credits (see `meta.md` Step 1)
2. Find the product: `{ shopifyProductsConnection(args: { first: 5, query: "title:*{name}*" }) { nodes { id title handle description } } }`
3. **Meta Tags:** Check current meta → if missing/weak, generate via `generateAIMeta` → review → apply (see `meta.md` Steps 3-6)
4. **FAQs:** Check existing FAQs on the product (via shopifyProxy metafield read) → if none, generate via `generateAIFAQ` → review → save via `bulkCreateAndAssignFaqs` (see `faq.md` Steps 3-5)
5. **Navigation:** Check breadcrumb assignment → if none, suggest via `suggestBreadcrumbPath` → apply (see `navigation.md` Breadcrumbs section)
6. Present summary:
   ```text
   Product SEO Optimization Complete:
     Meta Title: ✓ Applied
     Meta Description: ✓ Applied
     FAQs: ✓ {N} created and assigned
     Breadcrumbs: ✓ Set
   ```

**Flows involved:** Meta Tags → FAQ → Navigation

---

### Recipe 6: Full Collection SEO Optimization

**Trigger:** "Optimize this collection", "Full SEO for my {collection name} collection", "Make this collection rank better"

**Steps:**

1. Check AI credits
2. Find the collection: `{ shopifyCollectionsConnection(args: { first: 5, query: "title:*{name}*" }) { nodes { id title handle description productsCount } } }`
3. **Meta Tags:** Generate and apply (see `meta.md`)
4. **FAQs:** Generate and assign (see `faq.md`)
5. **Navigation:** Generate recommendations for Breadcrumbs + Similar collections + Discover via `generateBulkRecommendations` (see `navigation.md`) → review → accept
6. Present summary similar to Recipe 5

**Flows involved:** Meta Tags → FAQ → Navigation

---

### Recipe 7: Bulk SEO Sweep

**Trigger:** "Optimize all my products", "Bulk SEO for my store", "Generate meta tags and FAQs for everything"

**Steps:**

1. Check AI credits — estimate cost: ~1 credit per resource for meta, ~1 per resource for FAQs
2. Count resources:
   ```graphql
   { shopifyProductsConnection(args: { first: 1 }) { pageInfo { totalCount } } shopifyCollectionsConnection(args: { first: 1 }) { pageInfo { totalCount } } }
   ```
3. Present: "{X} products, {Y} collections. Estimated credits: {estimate}. Proceed?"
4. **Bulk Meta Tags:** Use `bulkGenerateAIMeta` in batches of 50 (see `meta.md`) → review sample → apply all
5. **Bulk FAQs:** Use `generateAIFAQ` per batch of resources → review → `bulkCreateAndAssignFaqs` (see `faq.md`)
6. Report coverage improvement:
   ```text
   Bulk SEO Sweep Complete:
     Meta Tags: {N} products + {M} collections updated
     FAQs: {F} FAQs created, assigned to {R} resources
     Credits Used: {total}
   ```

**Flows involved:** Meta Tags (bulk) → FAQ (bulk)

---

## Category 3: Content Pipeline

### Recipe 8: Content → Meta → FAQ Pipeline

**Trigger:** "Create all content for this product", "Full content package for {product}", "Write everything for this product"

**Steps:**

1. Check AI credits
2. Identify the product/collection
3. **Description:** Generate via `generateContent` with `PRODUCT_DESCRIPTION` → review → user saves to Shopify
4. **Meta Tags:** Generate via `generateAIMeta` for the same resource → review → apply via shopifyProxy
5. **FAQs:** Generate via `generateAIFAQ` for the same resource → review → save via `bulkCreateAndAssignFaqs`
6. Summary: "Content package complete: description, meta tags, and {N} FAQs created."

**Flows involved:** Content → Meta Tags → FAQ

---

### Recipe 9: FAQ Coverage Booster

**Trigger:** "Add FAQs to all products that don't have any", "Boost my FAQ coverage", "Which products need FAQs?"

**Steps:**

1. Check AI credits
2. List all products with their FAQ metafield:
   ```graphql
   { shopifyProxy(query: "query ($first: Int!, $after: String) { products(first: $first, after: $after) { nodes { id title metafield(key: \"$app:risify.faq\") { value } } pageInfo { hasNextPage endCursor } } }" variables: { "first": 50 }) { data errors } }
   ```
3. Filter products where metafield is null or empty → these have no FAQs
4. Present: "{N} of {total} products have no FAQs. Generate FAQs for the uncovered ones?"
5. Batch generate: `generateAIFAQ` for uncovered products (batches of 50)
6. Review a sample, then bulk save via `bulkCreateAndAssignFaqs`
7. Report: "FAQ coverage improved: {before}% → {after}%"

**Flows involved:** FAQ (discovery + bulk generation)

---

### Recipe 10: Content → Audit Validation

**Trigger:** "Will these changes improve my SEO?", "Generate content and check if my score improves", "Prove the ROI"

**Steps:**

1. Run audit → record current health score as baseline
2. Generate and apply content (meta tags, FAQs, descriptions — whichever the user requested)
3. Wait a few minutes for Shopify to index changes
4. Re-run audit
5. Compare:
   ```text
   SEO Score Comparison:
     Before: {baseline}/100
     After: {new}/100
     Change: +{delta} points

   Category Changes:
     Meta Issues: {before} → {after} ({delta})
     Broken Links: {before} → {after}
     Schema Errors: {before} → {after}
   ```

**Flows involved:** Audit → (any content flow) → Audit

---

## Category 4: Navigation Pipeline

### Recipe 11: Full Navigation Setup

**Trigger:** "Set up all navigation for my store", "I want breadcrumbs, similar collections, and discover suggestions", "Configure navigation from scratch"

**Steps:**

1. Check AI credits
2. Check semantic sync status (see `navigation.md` Semantic Sync section)
3. If not synced: preview cost → trigger sync → inform user to wait
4. Once synced: list collections
5. Generate bulk recommendations for all collections (covers Breadcrumbs, Similar collections, Discover — collection-side only):
   ```graphql
   mutation { generateBulkRecommendations(collectionIds: [...], types: [BREADCRUMBS, COLLECTION_MENU, RELATED_SEARCH]) { results { collectionId breadcrumbs { id title handle score } collectionMenu { id title handle score } relatedSearch { id title handle score } } errors { collectionId message } totalProcessed totalCreditsUsed } }
   ```
   > Enum→UI mapping (never expose enum names to user): `BREADCRUMBS`→Breadcrumbs, `COLLECTION_MENU`→Similar collections, `RELATED_SEARCH`→Discover.
6. Present recommendations grouped by collection → user accepts/dismisses
7. Apply accepted recommendations via shopifyProxy metafieldsSet (see `navigation-operations.md`)
8. **Note:** This recipe doesn't cover **Similar products** — that's a product-side feature with metafield key `related_products` and no `generateBulkRecommendations` enum. If the user wants product-side coverage, handle products separately via `BulkRelatedProductsEditModal` flow (manual selection or product-side AI suggestion if available).
9. Offer: "Want me to analyze the page design and generate styles for these navigation elements?"
10. If yes → follow Recipe 12

**Flows involved:** Navigation (sync → generate → apply) → (optional) Page Design

---

### Recipe 12: Navigation → Page Design

**Trigger:** "Style my breadcrumbs", "Make the navigation look better", "Generate CSS for my similar collections"

**Steps:**

1. Verify navigation is set up (check for existing breadcrumb/menu metafields)
2. Get store domain from `{ me { domain shopUrl } }`
3. Collect collection page URLs: `https://{domain}/collections/{handle}` for collections with navigation
4. Run `analyzePageDesign` with collection URLs and sections [BREADCRUMB, COLLECTION_MENU, RELATED_SEARCHES]
5. Present style configs and CSS
6. Guide user on applying the CSS to their theme

**Flows involved:** Navigation → Page Design

---

## Category 5: New Store Onboarding

### Recipe 13: Zero to Healthy (Complete Onboarding)

**Trigger:** "Set up SEO for my store from scratch", "I'm new, what should I do?", "Help me get started with Risify", "Complete SEO setup"

**Steps:**

1. **Account check:** `{ me { shopName isAppSubscriptionPlanActive } }` + `{ aiCreditInfo { limit usage } }` → verify subscription and credits
2. **Run first audit:** Follow `audit.md` → present health score as baseline
3. **Fix critical issues:** If broken links > 0 → list top 5 → suggest support ticket (Recipe 3)
4. **Fix meta issues:** If meta issues > 0 → follow Recipe 1 (audit → fix meta)
5. **Generate FAQs:** List top 20 products → generate 3 FAQs each → review → save (see `faq.md`)
6. **Set up navigation:** Follow Recipe 11 (full navigation setup)
7. **Final report:**
   ```text
   Store SEO Setup Complete!

   Baseline Health Score: {before}/100
   Actions Taken:
     ✓ SEO audit completed
     ✓ {N} meta tags optimized
     ✓ {F} FAQs created across {R} resources
     ✓ Navigation configured for {C} collections
     ✓ {issues} critical issues reported to support

   Next: Re-run your audit in a few days to see improvement.
   ```

**Flows involved:** Account → Audit → Meta Tags → FAQ → Navigation → (optional) Page Design

---

### Recipe 14: Quick Start (Minimum Viable SEO)

**Trigger:** "Quick SEO wins", "What's the fastest way to improve my SEO?", "I only have a few minutes"

**Steps:**

1. Check credits
2. **Meta tags for top products:** List first 20 products → `bulkGenerateAIMeta` → review → apply
3. **FAQs for top 10:** `generateAIFAQ` for first 10 products, 3 each → review → `bulkCreateAndAssignFaqs`
4. **Run audit:** Get baseline score
5. Summary: "Quick wins applied: {N} meta tags + {F} FAQs. Health score: {score}/100."

**Flows involved:** Meta Tags → FAQ → Audit

---

## Category 6: Monitoring & Maintenance

### Recipe 15: SEO Health Check

**Trigger:** "How's my SEO doing?", "Check my SEO health", "Any new issues?", "Compare with last audit"

**Steps:**

1. Get current audit: `{ currentAudit { id status createdAt } }`
2. Get audit list for trend comparison: `{ auditList { id status createdAt } }`
3. If latest audit is old (>7 days) → offer to run a new one
4. Fetch summary for latest + previous audit
5. Compare and present trends:
   ```text
   SEO Health Check:

   Current Score: {current}/100 (was {previous}/100)
   Trend: {direction} {delta} points

   Category Trends:
     Broken Links: {current} (was {previous}) {arrow}
     Meta Issues: {current} (was {previous}) {arrow}
     Schema Errors: {current} (was {previous}) {arrow}
     Page Speed: {current} (was {previous}) {arrow}

   Suggested Actions:
     {list top 3 improvement opportunities based on biggest issue categories}
   ```

**Flows involved:** Audit (current + historical comparison)

---

### Recipe 16: Credit-Aware Operation Planning

**Trigger:** "How many credits do I have left?", "What can I do with my remaining credits?", "Plan my credit usage"

**Steps:**

1. Check credits: `{ aiCreditInfo { limit usage resetAt } }`
2. Calculate remaining
3. Present options ranked by impact per credit:
   ```text
   AI Credits: {remaining} remaining (resets {resetAt})

   What you can do:
     Meta Tags: ~{remaining} products (1 credit each)
     FAQs: ~{remaining/count} batches of {count} FAQs (1 credit per resource)
     Navigation Sync: ~{remaining * collectionsPerCredit} collections
     Content: ~{remaining} pieces of content
     Page Design: ~{remaining} page analyses

   Recommendation: {highest-impact suggestion based on store state}
   ```

**Flows involved:** Account → (any feature based on budget)

---

## Category 7: Support Workflows

### Recipe 17: Feature Issue → Support Escalation

**Trigger:** "This isn't working", "I keep getting errors", "Something is broken", "I need help"

**Steps:**

1. Identify which feature is failing (from conversation context)
2. Collect error details from the last failed operation
3. Get account context: `{ me { shopName domain email } }`
4. Create support ticket with full context:
   ```graphql
   mutation { createSupportTicket(input: { topic: "Issue with {feature name}" message: "Store: {shopName} ({domain})\n\nIssue: {description of the problem}\n\nError: {error message from failed operation}\n\nSteps taken: {what was attempted}" }) }
   ```
5. Confirm: "Support ticket created. The Risify team will respond to {email}."

**Flows involved:** (any feature) → Support

---

## Category 8: Data Export

### Recipe 18: Collection Products Export

**Trigger:** "List all collections with their products", "Export collections and products", "Show me what's in each collection", "Collection product breakdown"

**Steps:**

1. List all collections with pagination:
   ```graphql
   { shopifyCollectionsConnection(args: { first: 250 }) { nodes { id title handle productsCount } pageInfo { hasNextPage endCursor } } }
   ```
   Keep paginating with `after` until `hasNextPage` is false. Collect all collection GIDs and titles.

2. For each collection, fetch its products via shopifyProxy:
   ```graphql
   { shopifyProxy(query: "query ($id: ID!, $first: Int!, $after: String) { collection(id: $id) { id title products(first: $first, after: $after) { nodes { id title handle status } pageInfo { hasNextPage endCursor } } } }" variables: { "id": "<collection-GID>", "first": 50 }) { data errors } }
   ```
   Paginate within each collection if it has more than 50 products.

3. Present results grouped by collection:
   ```text
   {Collection Title} ({productsCount} products):
     1. {product title} — {handle}
     2. {product title} — {handle}
     ...

   {Next Collection Title} ({productsCount} products):
     1. {product title} — {handle}
     ...
   ```

4. If the user wants a summary instead of full listing, present:
   ```text
   Collections Overview ({total} collections):

   1. {title} — {productsCount} products
   2. {title} — {productsCount} products
   ...

   Total: {sum} products across {total} collections
   ```

**Note:** For stores with many collections, this requires one API call per collection. Process in batches and show progress: "Fetched products for {N} of {total} collections..."

**Flows involved:** Shopify data export (collections + products)
