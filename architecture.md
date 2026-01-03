# Tommy the Tapir: Architecture Documentation

## Overview

Tommy the Tapir is a lightweight framework for enabling menu browsing, item selection, and receipt retrieval in physical retail environments — without replacing the existing POS.

**Tommy is not "another POS."** It is a receipt-first interaction layer.

## Core Principles

- **POS remains system of record**
- **Customer interaction is web-based**
- **Receipts are claim-based, not pushed**
- **Cashier confirmation is a valid receipt event**
- **Trust is explicit, not implicit**

## Tommy Architecture (High Level)

```
[NFC / QR]
   ↓
[Tommy Web App]
   ↓
[Selection / Cart]
   ↓
[Cashier / POS]
   ↓
[Receipt Confirmation]
   ↓
[ORF Receipt Claim]
```

Tommy lives beside the POS, not inside it.

## System Components

### 1. Menu + Selection (Physical Store)

#### How It Works

**NFC / QR tag placement:**
- Shelf
- Table
- Counter

**Customer interaction flow:**
1. Customer taps/scans NFC/QR tag
2. Customer sees:
   - Menu / catalog
   - Prices
   - Availability
3. Customer builds a cart

**Important:** At this point:
- No payment
- No reservation
- No commitment

This is essentially "in-store browsing with memory."

### 2. Checkout & Cashier Confirmation (Key Insight)

This is where Tommy becomes powerful without POS integration.

#### Flow

1. Customer approaches cashier
2. Customer shows cart summary (Tommy UI)
3. Cashier rings items into POS
4. Cashier confirms:
   - Items match
   - Total matches

#### Confirmation Methods (choose one)

- **Cashier taps "Confirm" button**
- **Cashier scans a QR shown on customer's phone**
- **Cashier enters a short confirmation code**

This creates a shared acknowledgment event.

### 3. Receipt Retrieval Without POS APIs

After confirmation, there are two receipt options:

#### Option A: Confirmed Selection Receipt

ORF receipt marked as:
```json
{
  "receipt_type": "confirmation",
  "payment_status": "external"
}
```

**Indicates:**
- Items confirmed by merchant
- Payment handled separately

**This is valid for:**
- Expense tracking
- Warranty
- Proof of purchase (non-fiscal)

#### Option B: Matched POS Receipt (Later)

If POS receipt ID becomes available:
- Tommy reconciles
- Upgrades receipt status
- Both map cleanly to ORF

### 4. NFC / QR Receipt Claim (Tommy Style)

After checkout, customer can:
- Tap the same NFC tag
- Scan a receipt QR at the counter

**Payload:**
```
https://tommy.example.com/r/8XK3
```

**Result:**
- ORF receipt displayed
- Downloadable
- Shareable

**No email.**
**No app.**
**No loyalty trap.**

### 5. Why "Cashier Confirmation" Is Legitimate

This is important philosophically and legally.

#### A receipt is:
- A statement of goods exchanged
- A merchant assertion

#### It does not require:
- Payment processor involvement
- Fiscal device access
- Network connectivity

Tommy formalizes something that already happens verbally.

### 6. ORF Mapping (Tommy-Friendly)

Tommy should consistently populate:

```json
{
  "receipt_assertion": {
    "asserted_by": "merchant",
    "assertion_method": "cashier_confirmation",
    "assertion_time": "2026-01-02T14:05:00Z"
  }
}
```

This makes the receipt honest about its provenance.

## Benefits for Retailers

- **Zero POS replacement**
- **No payment integration**
- **No customer data capture**
- **Faster lines**
- **Fewer "can I get a receipt?" moments**

This is especially attractive to:
- Cafés
- Food courts
- Markets
- Small retailers
- Emerging markets

## How ORF + Tommy Work Together (Positioning)

- **ORF defines the receipt**
- **Tommy defines the moment of trust**

## Technical Implementation Notes

### Trust Model
The framework operates on explicit trust between customer, cashier, and the Tommy system. Trust is established through:
- Physical presence at the point of sale
- Direct interaction between customer and cashier
- Verifiable confirmation events

### Data Flow
1. **Customer** interacts with NFC/QR tag
2. **Tommy Web App** creates session and cart
3. **Cashier** verifies and confirms selections
4. **Confirmation event** creates trust anchor
5. **ORF Receipt** is generated based on confirmation

### Scalability Considerations
- Framework is designed for lightweight deployment
- No central database requirement for cart storage
- Stateless architecture for horizontal scaling
- Minimal infrastructure requirements

## Deployment Architecture

### Physical Components
- NFC tags or QR codes at strategic locations
- Customer devices (web-based, no app required)
- Cashier confirmation interface

### Digital Components
- Tommy Web App (responsive, mobile-first)
- ORF integration layer
- Receipt claim system

### Integration Points
- **Optional:** POS system integration for receipt matching
- **Required:** ORF API for receipt generation
- **Optional:** Payment processor integration (not required for basic functionality)

## Security Considerations

- No sensitive customer data storage
- Session-based cart management
- Time-limited confirmation codes
- Physical presence requirements for trust establishment

## Future Enhancements

- POS receipt reconciliation when APIs available
- Enhanced analytics for retailers
- Inventory integration for real-time availability
- Loyalty program integration (optional)