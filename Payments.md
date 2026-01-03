# Tommy the Tapir: Payments Architecture

## Overview

Tommy is designed as a **payment-aware but payment-optional** framework. This approach ensures maximum adoptability while providing flexibility for retailers who want to integrate payment processing.

## 1. Tommy Core (Always Payment-Neutral)

Tommy Core includes:
- **Menu / catalog**
- **Selection / cart**
- **Session handling**
- **Receipt claiming**
- **ORF generation**

### Core Requirements

Tommy Core **MUST** function with:
- Cash payments
- External POS payments
- Offline environments

This keeps Tommy adoptable by small retailers who may not have or want payment processing integration.

### Core Characteristics

- **Payment Independence**: Core functionality works without any payment integration
- **Universal Compatibility**: Supports any payment method through external systems
- **Offline Resilience**: Operates in environments without network connectivity
- **Minimal Dependencies**: No payment processing libraries required in core

## 2. Tommy Payment Adapters (Optional Modules)

Tommy may provide adapters for major payment processors:

### Available Adapters

```javascript
// Optional payment adapters
@tommy/payments-stripe     // Stripe integration
@tommy/payments-adyen      // Adyen integration
@tommy/payments-square     // Square integration
// ... additional adapters as needed
```

### Adapter Characteristics

- **Pluggable**: Easy to add/remove based on retailer needs
- **Replaceable**: Can switch between providers without core changes
- **Optional**: Not required for basic Tommy functionality
- **Isolated**: Separate from ORF logic and core receipt generation

### Adapter Responsibilities

Payment adapters:
- **Emit payment events** for Tommy to process
- **Never define receipt structure**
- **Never mutate ORF schema**
- Handle provider-specific payment flows
- Manage payment method UI/UX

## 3. Event Boundary (This Is Critical)

Tommy should treat payment as an **event**, not a dependency.

### Payment Event Structure

```json
{
  "event_type": "payment_completed",
  "source": "stripe",
  "external_reference": "pi_3N...",
  "amount": 42.50,
  "currency": "USD",
  "timestamp": "2026-01-02T14:10:00Z"
}
```

### Event Processing

Tommy may:
- **Attach this as metadata** to the session
- **Reference it from ORF** receipts
- **Reconcile later** when POS data becomes available

### Critical Principle

**ORF never validates or executes payment events.**

Payment events are **descriptive, not prescriptive**. They inform the receipt but don't control its validity.

## 4. ORF Mapping for Payment-Integrated Tommy

### When Payment Exists

ORF reflects payment information descriptively:

```json
{
  "payments": [
    {
      "method": "card",
      "provider": "stripe",
      "reference": "pi_3N...",
      "amount": 42.50,
      "currency": "USD"
    }
  ],
  "receipt_assertion": {
    "asserted_by": "merchant",
    "assertion_method": "cashier_confirmation",
    "assertion_time": "2026-01-02T14:05:00Z"
  }
}
```

### When Payment Does Not Exist

```json
{
  "payments": [],
  "receipt_assertion": {
    "asserted_by": "merchant",
    "assertion_method": "cashier_confirmation",
    "assertion_time": "2026-01-02T14:05:00Z"
  }
}
```

### Important

**Both receipts are equally valid ORF.**

The presence or absence of payment information doesn't affect the fundamental validity of the receipt.

## 5. Tommy's Responsibility Boundary

### What Tommy May Do

Tommy **may** integrate with payment systems for **convenience**, but:

- Does **not replace** merchant POS systems
- Does **not replace** payment processors
- Does **not replace** fiscal devices
- Operates as a **complementary layer**

### Payment Adapter Independence

- Payment adapters operate **independently** of receipt generation
- Payment adapters are **not required** for ORF compliance
- Payment processing is **isolated** from core Tommy functionality

### Integration Philosophy

```
Tommy Core ←→ Payment Adapters ←→ External Payment Processors
                ↓
            ORF Receipt Generation (Payment-Optional)
```

## 6. Implementation Architecture

### Core Layer
```javascript
// Always available, payment-independent
class TommyCore {
  createSession()
  manageCart()
  generateORF()
  claimReceipt()
}
```

### Payment Adapter Layer
```javascript
// Optional, pluggable
interface PaymentAdapter {
  processPayment(amount, currency)
  emitPaymentEvent()
  handleCallback()
}
```

### Event Processing
```javascript
// Tommy processes payment events as metadata
class PaymentEventProcessor {
  attachPaymentMetadata(session, paymentEvent)
  includeInORF(session, paymentEvent)
  reconcileLater(session, posData)
}
```

## 7. Deployment Scenarios

### Scenario A: Cash-Only Retailer
- **Tommy Core only**
- No payment adapters
- Manual cashier confirmation
- ORF generation without payment data

### Scenario B: Integrated Payment Retailer
- **Tommy Core + Payment Adapter**
- Automated payment processing
- Payment data included in ORF
- Enhanced customer experience

### Scenario C: Hybrid Environment
- **Tommy Core + Multiple Payment Adapters**
- Support for different payment methods
- Flexible for various transaction types
- Unified receipt generation

## 8. Benefits of Payment-Optional Architecture

### For Retailers
- **Lower barrier to entry**: Start with basic functionality
- **Gradual adoption**: Add payment integration when ready
- **Risk mitigation**: No vendor lock-in for payment processing
- **Flexibility**: Support multiple payment providers

### For Developers
- **Simplified testing**: Core functionality doesn't require payment setup
- **Modular development**: Build payment features independently
- **Easier maintenance**: Payment issues don't affect core functionality
- **Broader adoption**: More retailers can use Tommy

## 9. Error Handling

### Payment Adapter Failures
- Core Tommy functionality remains intact
- Payment failures don't prevent receipt generation
- Graceful degradation to cash-only mode

### Network Connectivity Issues
- Offline operation supported
- Payment events queued for later processing
- Receipt generation continues uninterrupted

### Payment Reconciliation
- POS data can upgrade receipt status later
- Payment events can be matched with POS transactions
- Maintains data consistency across systems

## 10. Security Considerations

### Payment Data Isolation
- Payment information stored separately from core cart data
- PCI compliance handled by payment adapters
- Core Tommy doesn't process sensitive payment data

### Event Validation
- Payment events validated before inclusion in ORF
- External payment references verified
- Amount reconciliation between Tommy and payment systems

## Future Enhancements

### Planned Payment Adapters
- Additional regional payment providers
- Cryptocurrency payment support
- Buy-now-pay-later (BNPL) integration
- Contactless payment methods

### Advanced Features
- Payment analytics and reporting
- Multi-currency support
- Automatic tax calculation
- Discount and promotion integration