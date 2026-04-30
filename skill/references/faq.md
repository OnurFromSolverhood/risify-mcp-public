# Flow: FAQ Generation & Assignment

Generate AI-powered FAQs and save them to Shopify as metaobjects linked to products, collections, or pages.

## Architecture

FAQs are **Shopify Metaobjects** (type `$app:risify_faq`) with fields `question`, `answer`, `tags`. They are linked to resources via **metafields** (namespace `$app:risify`, key `faq`, type `list.metaobject_reference`).

The domain tools handle all Shopify complexity internally — you do NOT need to interact with `shopifyProxy` or construct GraphQL for the standard FAQ workflow.

## Step-by-Step Flow

Follow these steps in order. Do not skip steps.

### Step 1: Check AI Credits

Use the `check_credits` tool (no parameters).

- `limit = -1` → unlimited. `limit = 0` → disabled.
- If zero credits remain, tell the user and stop. Direct them to Risify app > Support > AI Credits.

### Step 2: Identify Target Resources

Ask the user which products and/or collections they want FAQs for.

Use `list_products` or `list_collections` tool:
- `first`: number to return (default 20, max 250)
- `after`: cursor for next page
- `query`: search filter (e.g. "title:shoes")

Collect the GIDs from the results (e.g., `gid://shopify/Product/123`).

### Step 3: Generate FAQs

Use the `generate_faqs` tool:
- `resourceGIDs`: array of selected GIDs
- `count`: number of FAQs per resource (1-10)
- `language`: optional ISO code ("en", "fr", "de")
- `tone`: optional ("professional", "friendly", "casual")

The tool returns generated Q&A pairs for review — they are NOT saved yet.

### Step 4: Review with User

Present each generated FAQ. The user may accept, edit, or discard.

ALWAYS use this exact template:

```
**FAQ #N**
Q: {question}
A: {answer}
→ Accept / Edit / Discard?
```

### Step 5: Save and Assign

Use the `create_and_assign_faqs` tool with:
- `items`: array of approved FAQs, each with `question` and `answer`
- `resourceGIDs`: the same GIDs from Step 2

This single call creates FAQ metaobjects in Shopify AND assigns them to all specified resources. The backend handles merging with any existing FAQ assignments.

### Step 6: Confirm

Tell the user how many FAQs were created and which resources they were assigned to.

## Additional Operations

| Task | Method |
|------|--------|
| List existing FAQs | `list_faqs` tool |
| Update a FAQ | `execute_graphql` with shopifyProxy → `metaobjectUpdate` (see `faq-operations.md`) |
| Delete a FAQ | `execute_graphql` with shopifyProxy → `metaobjectDelete` (see `faq-operations.md`) |

## Constants

These are handled internally by domain tools. Only needed if using `execute_graphql` directly:

| Key | Value |
|-----|-------|
| Metaobject type | `$app:risify_faq` |
| Metafield namespace | `$app:risify` |
| Metafield key | `faq` |
| Metafield type | `list.metaobject_reference` |
| Max items per create_and_assign_faqs | 250 |
| FAQ count range | 1–10 |

## Error Handling

| Situation | Response |
|-----------|----------|
| No AI credits | Tell user. Direct to Risify > Support > AI Credits |
| Invalid resource GIDs | Verify selections exist. Re-fetch with list_products/list_collections |
| create_and_assign_faqs fails | Check error message — FAQ feature may not be activated in Risify |
| shopifyProxy errors (update/delete) | Check `errors` field. Common: access denied, rate limited |
