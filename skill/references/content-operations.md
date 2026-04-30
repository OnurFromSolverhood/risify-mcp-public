# Content Generation GraphQL Operations Reference

All operations use the `execute_graphql` MCP tool directly against the Risify API.

---

## Generate Content

```graphql
mutation {
  generateContent(input: {
    contentType: BLOG_POST
    parameters: "Topic or product information for the AI to write about"
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

### Content Types

| Enum Value | Description |
|---|---|
| `META` | SEO meta title and description (use `generateAIMeta` instead for structured output) |
| `FAQ` | FAQ Q&A pairs (use `generateAIFAQ` instead for structured output) |
| `PRODUCT_DESCRIPTION` | Product description copy |
| `COLLECTION_DESCRIPTION` | Collection description copy |
| `BLOG_POST` | Long-form blog post content |
| `ABOUT_US` | About-us or brand story content |

### Tone Options

`PROFESSIONAL`, `CASUAL`, `FRIENDLY`, `FORMAL`, `PERSUASIVE`

### Length Options

`SHORT`, `MEDIUM`, `LONG`

### Response

- `success: Boolean` — whether generation succeeded
- `content: String` — generated content as JSON string (parse before displaying)
- `contentType: String` — echoes the requested type
- `error: String` — error message if `success` is false
