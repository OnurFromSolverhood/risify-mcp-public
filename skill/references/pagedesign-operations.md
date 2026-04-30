# Page Design GraphQL Operations Reference

All operations use the `execute_graphql` MCP tool directly against the Risify API.

---

## Analyze Page Design

```graphql
mutation {
  analyzePageDesign(input: {
    pages: [
      { url: "https://store.myshopify.com/collections/summer", pageType: COLLECTION },
      { url: "https://store.myshopify.com/products/blue-dress", pageType: PRODUCT }
    ]
    sections: [BREADCRUMB, FAQ, COLLECTION_MENU, RELATED_SEARCHES, FAQ_TAGS]
  }) {
    styleConfigs
    customCss
    creditsUsed
  }
}
```

### Input

- `pages` (required): Array of 1-9 pages to analyze
  - `url: String!` — full page URL
  - `pageType: PageDesignPageType!` — `COLLECTION`, `PRODUCT`, or `BLOG`
- `sections` (optional): Which sections to analyze. Default: all.
  - `BREADCRUMB`, `FAQ`, `COLLECTION_MENU`, `RELATED_SEARCHES`, `FAQ_TAGS`

### Response

- `styleConfigs: String!` — JSON string keyed by section name, containing style configuration objects
- `customCss: String!` — JSON string keyed by section name, containing CSS strings
- `creditsUsed: Int!` — AI credits consumed
