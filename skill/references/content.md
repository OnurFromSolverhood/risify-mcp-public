# Flow: Content Generation

Generate AI-powered content for various ecommerce use cases: blog posts, product descriptions, collection descriptions, and about-us pages.

## Step-by-Step Flow

### Step 1: Check AI Credits

```graphql
query { aiCreditInfo { limit usage resetAt } }
```

### Step 2: Determine Content Type

Ask user what kind of content they need:

| Content Type | Value | Use Case |
|---|---|---|
| Blog post | `BLOG_POST` | Long-form blog articles |
| Product description | `PRODUCT_DESCRIPTION` | Product page copy |
| Collection description | `COLLECTION_DESCRIPTION` | Collection page copy |
| About us | `ABOUT_US` | Brand story / about page |

### Step 3: Gather Parameters

Ask user for the topic/context. The `parameters` field is a free-text description of what to write about.

Examples:
- Blog: "Write about the benefits of organic cotton t-shirts for our sustainable fashion brand"
- Product: "Premium running shoes with breathable mesh upper, EVA midsole, brand: SportsPro"
- Collection: "Summer 2026 collection featuring lightweight linen clothing"
- About us: "Family-owned jewelry brand since 1985, handcrafted in Portland, Oregon"

Ask for tone and length preferences:
- **Tone:** `PROFESSIONAL`, `CASUAL`, `FRIENDLY`, `FORMAL`, `PERSUASIVE` (default: PROFESSIONAL)
- **Length:** `SHORT`, `MEDIUM`, `LONG` (default: MEDIUM)

### Step 4: Generate Content

```graphql
mutation {
  generateContent(input: {
    contentType: BLOG_POST
    parameters: "Write about the benefits of organic cotton t-shirts"
    toneOfVoice: PROFESSIONAL
    contentLength: MEDIUM
  }) {
    success
    content
    contentType
    error
  }
}
```

The `content` field returns the generated text as a JSON string.

### Step 5: Present to User

Parse the `content` JSON and present in a clean, readable format. Ask user:
- **Accept** → provide as final copy
- **Edit** → ask what to change, regenerate with updated parameters
- **Regenerate** → run again with same or different tone/length

## Error Handling

| Situation | Response |
|-----------|----------|
| No AI credits | Tell user. Suggest plan upgrade |
| `success = false` | Show error message, suggest adjusting parameters |
| Content too short/long | Suggest regenerating with different `contentLength` |
