# dOS-POS Implementation Plan

## Overview
Fork of Stripe Terminal React Native example app, customized to integrate with dOS (digiBandit Operating System) for mobile POS payments via WisePad E.

**Repo:** https://github.com/dbits-db/dOS-POS

---

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   dOS-POS App   │────▶│   dOS Backend   │────▶│   Stripe API    │
│  (React Native) │     │ (Apps Script)   │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │
        │                       ▼
        │               ┌─────────────────┐
        │               │    ITFlow API   │
        │               └─────────────────┘
        │
        ▼
┌─────────────────┐     ┌─────────────────┐
│   WisePad E     │     │   n8n Webhook   │──▶ Print Receipt
│   (Bluetooth)   │     │                 │
└─────────────────┘     └─────────────────┘
```

---

## Backend Endpoints (Add to dOS Main.js)

### Required for Stripe Terminal SDK

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/connection_token` | POST | Returns Stripe Terminal connection token |
| `/create_payment_intent` | POST | Creates payment intent for Terminal |
| `/capture_payment_intent` | POST | Captures manual-capture payments |

### dOS Integration Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/search_clients` | GET | Search ITFlow clients |
| `/get_invoices` | GET | Get unpaid invoices for client |
| `/log_transaction` | POST | Save transaction to dOS sheet |
| `/print_receipt` | POST | Trigger n8n print webhook |

### Implementation in Main.js

```javascript
// Web app entry point for dOS-POS API
function doPost(e) {
  var path = e.parameter.action || e.pathInfo;

  switch(path) {
    case 'connection_token':
      return createConnectionToken();
    case 'create_payment_intent':
      return createTerminalPaymentIntent(e.parameter);
    case 'capture_payment_intent':
      return captureTerminalPaymentIntent(e.parameter);
    case 'log_transaction':
      return logPOSTransaction(e.parameter);
    default:
      return ContentService.createTextOutput(JSON.stringify({error: 'Unknown action'}));
  }
}

function createConnectionToken() {
  var response = UrlFetchApp.fetch('https://api.stripe.com/v1/terminal/connection_tokens', {
    method: 'POST',
    headers: { 'Authorization': 'Basic ' + Utilities.base64Encode(getStripeKey() + ':') }
  });
  return ContentService.createTextOutput(response.getContentText())
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

## Mobile App Changes

### Files to Modify

| File | Changes |
|------|---------|
| `example-app/.env` | Set API_URL to dOS webapp URL |
| `example-app/app.json` | Rename to "dOS-POS", update bundle ID |
| `src/App.tsx` | Update navigation, add dOS screens |
| `src/api/api.ts` | Add ITFlow/dOS API methods |
| `src/AppContext.ts` | Add client/invoice state |
| `src/colors.ts` | Match dOS color scheme |

### New Screens to Create

#### 1. `src/screens/ClientSelectScreen.tsx`
- Search input for ITFlow client lookup
- List of matching clients
- Select client → store in context
- "Walk-in" option for no client

#### 2. `src/screens/InvoiceSelectScreen.tsx`
- Show unpaid invoices for selected client
- Select invoice to pay (pre-fills amount)
- "Custom Amount" option for non-invoice payments

#### 3. `src/screens/QuickPayScreen.tsx`
- Simplified payment screen (replaces CollectCardPaymentScreen)
- Shows: Client name, amount, description
- CAD currency, Interac enabled by default
- Big "Charge" button
- Auto-capture (not manual)

#### 4. `src/screens/PaymentSuccessScreen.tsx`
- Shows success confirmation
- Receipt details
- "Print Receipt" button → n8n webhook
- "New Transaction" button

### Screens to Remove/Hide

- SetupIntentScreen (not needed)
- RegisterInternetReaderScreen (not needed for Bluetooth)
- Most advanced options from CollectCardPaymentScreen

---

## Simplified User Flow

```
1. Launch App
   └─▶ HomeScreen (connect WisePad E via Bluetooth)

2. Connected
   └─▶ "New Sale" button

3. ClientSelectScreen
   ├─▶ Search ITFlow clients
   ├─▶ Select client (or "Walk-in")
   └─▶ Next

4. InvoiceSelectScreen (if client selected)
   ├─▶ Show unpaid invoices
   ├─▶ Select invoice OR "Custom Amount"
   └─▶ Next

5. QuickPayScreen
   ├─▶ Shows amount, client, description
   ├─▶ Tap "Charge $XX.XX"
   └─▶ Present card to WisePad E

6. PaymentSuccessScreen
   ├─▶ Success!
   ├─▶ [Print Receipt]
   └─▶ [New Transaction]
```

---

## Configuration

### .env File
```
API_URL=https://script.google.com/macros/s/[DEPLOYMENT_ID]/exec
```

### app.json Updates
```json
{
  "expo": {
    "name": "dOS-POS",
    "slug": "dos-pos",
    "version": "1.0.0",
    "ios": {
      "bundleIdentifier": "ca.dbits.dospos"
    },
    "android": {
      "package": "ca.dbits.dospos"
    }
  }
}
```

### Script Properties (dOS)
```
STRIPE_KEY=sk_live_...           (existing)
STRIPE_LOCATION=tml_...          (existing)
```

---

## Development Steps

### Phase 1: Backend (1 day)
- [ ] Add `doPost()` handler for API routing in Main.js
- [ ] Implement `createConnectionToken()`
- [ ] Implement `createTerminalPaymentIntent()`
- [ ] Implement `captureTerminalPaymentIntent()`
- [ ] Implement `logPOSTransaction()`
- [ ] Deploy new dOS version
- [ ] Test endpoints with curl

### Phase 2: App Setup (0.5 day)
- [ ] Update .env with dOS API URL
- [ ] Update app.json (name, bundle IDs)
- [ ] Update colors.ts to match dOS theme
- [ ] Run `yarn install` in example-app
- [ ] Test basic app launches

### Phase 3: New Screens (1.5 days)
- [ ] Create ClientSelectScreen
- [ ] Create InvoiceSelectScreen
- [ ] Create QuickPayScreen
- [ ] Create PaymentSuccessScreen
- [ ] Update navigation in App.tsx
- [ ] Add API methods to api.ts

### Phase 4: Integration (1 day)
- [ ] Wire up ITFlow client search
- [ ] Wire up invoice lookup
- [ ] Test full payment flow with WisePad E
- [ ] Add transaction logging to dOS
- [ ] Add print receipt via n8n

### Phase 5: Polish (0.5 day)
- [ ] Error handling and loading states
- [ ] Offline handling
- [ ] App icon and splash screen
- [ ] Build release APK/IPA

---

## API Reference

### ITFlow Clients (existing in dOS)
```javascript
searchITFlowClients(query)
// Returns: [{id, name, email, phone}, ...]
```

### ITFlow Invoices (existing in dOS)
```javascript
getClientInvoices(clientId)
// Returns: [{id, number, amount, due_date, status}, ...]
```

### Create Terminal Payment Intent
```javascript
// POST /create_payment_intent
{
  amount: 11300,        // cents
  currency: 'cad',
  description: 'Invoice #1234',
  metadata: {
    client_id: 456,
    client_name: 'Acme Corp',
    invoice_id: 1234,
    source: 'dOS-POS'
  }
}
// Returns: { secret: 'pi_xxx_secret_xxx', intent: 'pi_xxx' }
```

### Log Transaction
```javascript
// POST /log_transaction
{
  payment_intent_id: 'pi_xxx',
  charge_id: 'ch_xxx',
  amount: 113.00,
  tax: 13.00,
  client_id: 456,
  client_name: 'Acme Corp',
  description: 'Invoice #1234',
  payment_method: 'Visa ****4242'
}
```

---

## Estimated Timeline

| Phase | Time |
|-------|------|
| Backend endpoints | 1 day |
| App setup | 0.5 day |
| New screens | 1.5 days |
| Integration | 1 day |
| Polish | 0.5 day |
| **Total** | **4.5 days** |

---

## Files Summary

### dOS (Main.js additions)
```
+100 lines: doPost handler + Terminal API functions
```

### dOS-POS (example-app modifications)
```
Modified:
- .env
- app.json
- src/App.tsx
- src/api/api.ts
- src/AppContext.ts
- src/colors.ts
- src/screens/HomeScreen.tsx

New:
- src/screens/ClientSelectScreen.tsx
- src/screens/InvoiceSelectScreen.tsx
- src/screens/QuickPayScreen.tsx
- src/screens/PaymentSuccessScreen.tsx
- src/api/dosApi.ts (dOS-specific API calls)
```
