# Metafields setup — Rough Bobber theme

The custom sections read a few **metafields** so content can vary per product /
collection. All of them are **optional** — without them, sections fall back to their
global section settings (the breadcrumb simply won't show nested parents).

## Where to create them

**Shopify admin → Settings → Custom data → Products** (or **Collections**) **→ Add definition.**

- The **Namespace and key** must match exactly (e.g. `specs.size_guide`).
- Whatever you type in the definition's **Description** shows up under the field on
  every product/collection admin page — that's where merchants read the rules.

---

## Product metafields — used by the **Product specs** section (`rb-product-specs`)

### 1. `specs.product_details` — Rich text
Overrides the Details column body (intro + bullet list) for that product.

> Product details for this product. Use a bulleted list for the spec lines.
> Overrides the section's default details.

### 2. `specs.size_guide` — Multi-line text
Per-product size table. **Highest priority** for the size guide.

> Size chart. One row per line; separate columns with commas. The FIRST line is the
> header. Keep the same number of commas on every line. Don't use commas inside a
> value (use 1000, not 1,000). Put units in the header.
>
> Example:
> Size, Chest (in), Length (in)
> S, 18, 28
> M, 20, 29
> L, 22, 30

### 3. `specs.size_guide_image` — File (limit accepted types to **Images**)
Per-product size chart image. Shown only if **no** text size guide is set.

> Optional size chart image. If the text size guide above is also filled in, the
> text version is shown instead.

### 4. `specs.size_note` — Single line text
Small note shown under the size table/image.

> Optional note shown under the size guide (e.g. fit advice).

**Size guide precedence (first match wins):**
`specs.size_guide` (text) → `specs.size_guide_image` (image) → section's global Size table.

---

## Collection metafields — used by the **Breadcrumb** section (`rb-breadcrumb`)

### 5. `custom.parent_collection` — Collection (reference)
Points a collection to its parent so the breadcrumb can build nested trails
(`Home / Shop / T-Shirts / Product`). Set it on each collection that has a parent.

> The parent collection of this collection. Used to build the breadcrumb trail.
> Leave empty for top-level collections.

The metafield key is configurable in the breadcrumb section settings
("Parent collection metafield"); `custom.parent_collection` is the default.

---

## Quick reference

| # | Owner | Namespace.key | Type | Section |
|---|---|---|---|---|
| 1 | Product | `specs.product_details` | Rich text | Product specs |
| 2 | Product | `specs.size_guide` | Multi-line text | Product specs |
| 3 | Product | `specs.size_guide_image` | File (images) | Product specs |
| 4 | Product | `specs.size_note` | Single line text | Product specs |
| 5 | Collection | `custom.parent_collection` | Collection reference | Breadcrumb |

## Notes

- The theme also reads `reviews.rating` / `reviews.rating_count` (star ratings). These
  are standard Dawn / product-review-app metafields, created automatically by a reviews
  app — not something you set up manually here.
- The **Product specs** size table (global setting and the `specs.size_guide` metafield)
  use the same format: one row per line, cells separated by commas, first line = header.