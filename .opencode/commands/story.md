---
description: Generate user stories from a feature description
---

Generate user stories from the following description:

$ARGUMENTS

Use @ears-reference.adoc as the reference for writing acceptance criteria using the EARS standard.

## Step 1: Analyze the Input

Read the description carefully. Identify:
- The main functionality requested
- Who the users are (roles)
- What problems they want solved
- Any specific requirements mentioned

## Step 2: Decompose into Stories

Break the functionality into atomic user stories. Each story should:
- Be implementable in 1-3 days
- Deliver clear user value
- Have testable acceptance criteria

### Common Decomposition Patterns

| Feature Type | Typical Stories |
|--------------|-----------------|
| Authentication | Login, Logout, Registration, Password Reset, Session Management |
| CRUD Operations | Create, Read, Update, Delete, List/Search |
| User Settings | View Settings, Update Settings, Preferences |
| Notifications | Subscribe, Receive, View History, Manage Preferences |

## Step 3: Write Each Story

For each story, include:

### Title
Concise, descriptive title (e.g., "User Registration")

### User Story Statement
```
As a [specific user role],
I want [specific action/feature],
so that [benefit/value].
```

### Acceptance Criteria (EARS Standard)

Write each acceptance criterion using the appropriate EARS sentence type:

- **Ubiquitous** (always-on behavior): `The <system> shall <response>.`
- **Event-driven** (triggered by action): `When <trigger>, the <system> shall <response>.`
- **State-driven** (during a state): `While <state>, the <system> shall <response>.`
- **Unwanted behavior** (error/edge cases): `If <trigger>, then the <system> shall <response>.`
- **Optional** (feature-dependent): `Where <feature>, the <system> shall <response>.`

Each criterion must be:
- Written using one of the EARS sentence types above
- Specific, unambiguous, and verifiable
- Covering happy path, edge cases, and error scenarios

## Step 4: Structure the Output

Organize stories under feature headings. Use this AsciiDoc template:

```asciidoc
= User Stories

:toc:

== [Feature Name]

=== Story N: [Title]
As a [role],
I want [goal],
so that [benefit].

Acceptance Criteria:
* [ ] The system shall [ubiquitous requirement].
* [ ] When [trigger], the system shall [response].
* [ ] If [error condition], then the system shall [response].
```

## Step 5: File Handling

- Target file: `user-stories.adoc` in project root
- If file doesn't exist: Create with the `= User Stories` header and `:toc:` directive
- If file exists: Read current content, append new stories under appropriate feature section or create new feature section
- Preserve existing content and TOC
- Always write the result to the file

## Guidelines for Good User Stories (INVEST)

1. **Independent** - Each story can be implemented and tested separately
2. **Negotiable** - Details can be discussed and refined
3. **Valuable** - Delivers value to the end user
4. **Estimable** - Can be reasonably estimated
5. **Small** - Can be completed in one sprint/iteration
6. **Testable** - Clear acceptance criteria written using EARS sentence types

Break complex features into multiple smaller stories. Group related stories under feature headings.

## Example Transformation

**Input**: "I want an e-commerce site where users can browse products, add them to cart, and checkout"

**Output**:
```asciidoc
== Product Browsing

=== Story 1: View Product Catalog
As a shopper,
I want to browse available products,
so that I can find items I want to purchase.

Acceptance Criteria:
* [ ] The system shall display products in a grid or list format.
* [ ] The system shall show the name, price, and image for each product.
* [ ] When the user selects a category filter, the system shall display only products in that category.
* [ ] When the user selects a sort option, the system shall order products by the selected attribute.

=== Story 2: Search Products
As a shopper,
I want to search for products by keyword,
so that I can quickly find specific items.

Acceptance Criteria:
* [ ] The system shall provide a search input that accepts text queries.
* [ ] When the user submits a search query, the system shall display matching products.
* [ ] When the user submits a search query that matches no products, the system shall display a helpful empty-state message.
* [ ] When search results are displayed, the system shall highlight the matching search terms in the results.

== Shopping Cart

=== Story 3: Add to Cart
As a shopper,
I want to add products to my cart,
so that I can purchase multiple items at once.

Acceptance Criteria:
* [ ] The system shall display an add-to-cart button on each product.
* [ ] When the user adds a product to the cart, the system shall update the cart count in the header.
* [ ] When the user adds a product to the cart, the system shall display a success confirmation.
* [ ] When the user adds a product that is already in the cart, the system shall increment the quantity for that product.

=== Story 4: View Cart
As a shopper,
I want to view my cart contents,
so that I can review items before checkout.

Acceptance Criteria:
* [ ] The system shall display all cart items with their quantities.
* [ ] The system shall display correct item prices and a cart total.
* [ ] When the user changes an item quantity, the system shall update the item subtotal and cart total.
* [ ] When the user removes an item from the cart, the system shall remove it from the display and update the cart total.

== Checkout

=== Story 5: Checkout Process
As a shopper,
I want to complete my purchase,
so that I can receive my ordered items.

Acceptance Criteria:
* [ ] The system shall provide a form for entering a shipping address.
* [ ] The system shall allow the user to select a payment method.
* [ ] The system shall display an order summary before the user confirms the order.
* [ ] When the user confirms the order, the system shall display a confirmation page.
* [ ] When the order is confirmed, the system shall send an order confirmation email.
* [ ] If payment processing fails, then the system shall display an error message and allow the user to retry.
```

## Quality Checklist

Before outputting, verify:
- [ ] Each story has clear user value
- [ ] Acceptance criteria use the correct EARS sentence type
- [ ] Acceptance criteria are specific, unambiguous, and verifiable
- [ ] Stories are independent and can be implemented in any order
- [ ] No story is too large (break down if needed)
- [ ] Feature groupings make sense
- [ ] Error and edge case scenarios are covered using the "If ... then" (unwanted behavior) EARS type
