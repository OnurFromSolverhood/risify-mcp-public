# Flow: Page Design Analysis

Analyze store pages and generate style configurations and custom CSS for Risify theme sections (breadcrumbs, FAQs, collection menus, related searches).

## Step-by-Step Flow

### Step 1: Check AI Credits

```graphql
query { aiCreditInfo { limit usage resetAt } }
```

### Step 2: Gather Page URLs

Ask user which store pages to analyze. Get 1-9 page URLs with their types.

Page types: `COLLECTION`, `PRODUCT`, `BLOG`

Example: "Which pages should I analyze? I need the URL and type (collection, product, or blog page)."

### Step 3: Choose Sections (optional)

Ask if user wants all sections analyzed or specific ones:

| UI label | GraphQL enum value |
|---|---|
| Breadcrumbs | `BREADCRUMB` |
| FAQ | `FAQ` |
| Similar collections | `COLLECTION_MENU` |
| Discover (suggestions) | `RELATED_SEARCHES` |
| FAQ Tags | `FAQ_TAGS` |

Map user phrasings: "discover" / "discover sections" → `RELATED_SEARCHES`; "similar" / "similar collections" → `COLLECTION_MENU`. Old labels "Related Searches" and "Collection Menu" must NOT appear in user-facing text.

Default: all sections.

### Step 4: Analyze Pages

```graphql
mutation {
  analyzePageDesign(input: {
    pages: [
      { url: "https://store.myshopify.com/collections/summer", pageType: COLLECTION },
      { url: "https://store.myshopify.com/products/blue-dress", pageType: PRODUCT }
    ]
    sections: [BREADCRUMB, FAQ, COLLECTION_MENU]
  }) {
    styleConfigs
    customCss
    creditsUsed
  }
}
```

- Max 9 pages per call
- `styleConfigs`: JSON string with style configurations per section
- `customCss`: JSON string with custom CSS per section

### Step 5: Present Results

Parse the JSON responses and present:

```text
Page Design Analysis Complete ({creditsUsed} credits used):

Style Configurations:
{formatted styleConfigs per section}

Custom CSS:
{formatted CSS per section}

Would you like to apply these styles, or make adjustments?
```

## Constraints

| Key | Value |
|-----|-------|
| Max pages per call | 9 |
| Page types | COLLECTION, PRODUCT, BLOG |
| Sections | BREADCRUMB, FAQ, COLLECTION_MENU, RELATED_SEARCHES, FAQ_TAGS |

## Error Handling

| Situation | Response |
|-----------|----------|
| No AI credits | Tell user. Suggest plan upgrade |
| Invalid URL | Ask user to verify the page URL is correct and accessible |
| Page not crawlable | The store page may be behind a password or unavailable |
