# FAQ Operations Reference

Most FAQ operations use **domain tools** directly. Only FAQ update/delete still require `execute_graphql` with `shopifyProxy`.

---

## Domain Tools (preferred)

### Check AI Credits
Tool: `check_credits` (no parameters)

### List Products
Tool: `list_products` with optional `first`, `after`, `query`

### List Collections
Tool: `list_collections` with optional `first`, `after`, `query`

### Generate FAQs
Tool: `generate_faqs`
- `resourceGIDs`: ["gid://shopify/Product/123", "gid://shopify/Collection/456"]
- `count`: 3
- `language`: "en" (optional)
- `tone`: "professional" (optional)

### Save and Assign FAQs
Tool: `create_and_assign_faqs`
- `items`: [{"question": "What is your return policy?", "answer": "We offer 30-day returns."}]
- `resourceGIDs`: ["gid://shopify/Product/123"]

### List Existing FAQs
Tool: `list_faqs` with optional `first`, `after`

---

## Shopify Admin API via execute_graphql (for update/delete only)

These operations have no domain tool and must use `execute_graphql` with `shopifyProxy`.

### Update FAQ Metaobject

```graphql
{
  shopifyProxy(
    query: "mutation metaobjectUpdate($id: ID!, $metaobject: MetaobjectUpdateInput!) { metaobjectUpdate(id: $id, metaobject: $metaobject) { metaobject { id handle type displayName updatedAt fields { key value } } userErrors { field message code } } }"
    variables: {
      "id": "gid://shopify/Metaobject/123",
      "metaobject": {
        "fields": [
          { "key": "question", "value": "Updated question?" },
          { "key": "answer", "value": "Updated answer." }
        ]
      }
    }
  ) {
    data
    errors
  }
}
```

### Delete FAQ Metaobject

```graphql
{
  shopifyProxy(
    query: "mutation metaobjectDelete($id: ID!) { metaobjectDelete(id: $id) { deletedId userErrors { field message code } } }"
    variables: {
      "id": "gid://shopify/Metaobject/123"
    }
  ) {
    data
    errors
  }
}
```

### Get FAQ Metrics Count

```graphql
{
  shopifyProxy(
    query: "query ($type: String!) { metaobjectDefinitionByType(type: $type) { metaobjectsCount } }"
    variables: {
      "type": "$app:risify_faq"
    }
  ) {
    data
    errors
  }
}
```
