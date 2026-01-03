# Shopify / WooCommerce Integration — Thought Experiment

## 1. Ground Rules (Very Important)

**Assumptions:**

- Tommy does not replace checkout
- Tommy does not handle payments
- Tommy does not require customer accounts
- Tommy consumes only what platforms already expose
- ORF remains the canonical receipt artifact

> If any of these fail, the model is wrong.

---

## 2. What Shopify / WooCommerce Already Provide

Both platforms expose:

- Order lifecycle events
- Line items
- Prices
- Taxes
- Discounts
- Order IDs
- Payment status
- Webhooks

They do **not** provide:

- A standardized receipt object
- Claim-based receipt retrieval
- Portable receipt formats

---

## 3. Minimal Integration: "Receipt Companion" Pattern

Tommy is added as a **post-checkout companion**, not a checkout plugin.

### Entry Point

After checkout completion:

- **Shopify:** "Thank you" page
- **WooCommerce:** Order received page

Tommy is linked via:

- Button: "Claim digital receipt"
- URL with signed order reference

**Example:**

```
https://tommy.example.com/claim?order=123&sig=abc
```

No customer data required.

---

## 4. Data Flow (Step by Step)

### Step 1: Order Completes

**Shopify / WooCommerce:**

- Finalizes order
- Fires webhook:
  - `order.created`
  - `order.paid`

### Step 2: Tommy Receives Event

**Tommy:**

- Stores minimal order snapshot
- Generates:
  - `receipt_claim_id`
  - Expiry
- Prepares ORF document

### Step 3: Customer Claims Receipt

**Customer:**

- Clicks link
- Sees receipt
- Downloads ORF JSON / PDF
- Saves to wallet (optional)

No email. No login.

---

## 5. The Receipt Assertion (Online-Specific)

```json
"receipt_assertion": {
  "asserted_by": "system",
  "assertion_method": "platform_order_finalized",
  "source_platform": "shopify",
  "source_reference": "order_123"
}
```

This is materially different from in-store — and cleanly modeled.

---

## 6. Payment Reference (Descriptive Only)

```json
"payments": [
  {
    "method": "card",
    "provider": "shopify_payments",
    "reference": "txn_456",
    "amount": 79.00,
    "currency": "USD"
  }
]
```

ORF records **what happened**, not **how it happens**.

---

## 7. Platforms Impacts

**Important realism check.**

Tommy:

- Does not intercept checkout
- Does not compete with payments
- Does not capture customer emails
- Does not bypass platform fees

It adds:

- Better receipt UX
- Fewer support requests
- No platform risk

From a platform perspective, **this is harmless**.

---

## 8. WooCommerce-Specific Nuance (Interesting Case)

WooCommerce is self-hosted.

This allows a deeper integration:

Tommy plugin can:

- Generate ORF directly
- Host claim endpoint locally
- Avoid third-party services

This could become:

- A reference ORF implementation
- A community-driven adoption path

**Shopify cannot do this.**

---

## 9. Failure Mode Analysis

**What if:**

- **Webhook fails?**
  → Customer can still claim later via order ID + amount

- **Payment is delayed?**
  → Receipt marked as provisional

- **Partial refunds?**
  → ORF receipt updated with adjustment events

**Nothing breaks.**

---

## 10. Why This Is Not "Just Another Receipt"

Shopify already emails receipts.

But:

- Emails are not standardized
- Emails are not portable
- Emails are not privacy-preserving
- Emails decay over time

Tommy + ORF:

- Standardizes the artifact
- Decouples delivery
- Enables reuse (accounting, warranty, expense)

**This is the real differentiation.**

---

## 11. The Thought Experiment Verdict

| Question | Answer |
|----------|--------|
| Does this require special Shopify / WooCommerce features? | No |
| Does it break if payments change? | No |
| Does it require customer identity? | No |
| Does it generalize back to physical stores? | Yes |

**That symmetry is the key result.**

---

## 12. The One-Sentence Outcome

> If Tommy can sit next to Shopify without touching checkout, it can sit next to any POS without touching payments.

**That is the litmus test — and it passes.**
