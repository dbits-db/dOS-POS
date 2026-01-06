# dOS-POS

Mobile point-of-sale app for WisePad E, integrated with dOS (digiBandit Operating System).

Fork of [Stripe Terminal React Native SDK](https://github.com/stripe/stripe-terminal-react-native) example app.

## Features

- Connect to WisePad E via Bluetooth
- Search ITFlow clients
- Select unpaid invoices or enter custom amounts
- Accept Visa, Mastercard, Amex, Interac
- Log transactions to dOS
- Print receipts via n8n webhook

## Setup

### 1. Configure environment

```bash
cd example-app
cp .env.example .env
```

Edit `.env` and set your dOS API URL:
```
API_URL=https://script.google.com/macros/s/[DEPLOYMENT_ID]/exec
```

### 2. Install dependencies

```bash
cd example-app
yarn install
```

### 3. Run the app

**Android:**
```bash
yarn android
```

**iOS:**
```bash
cd ios && pod install && cd ..
yarn ios
```

## Architecture

```
dOS-POS App (React Native)
    │
    ├──▶ WisePad E (Bluetooth)
    │
    ├──▶ dOS Backend (Apps Script)
    │       ├── /connection_token
    │       ├── /create_payment_intent
    │       ├── /capture_payment_intent
    │       └── /log_transaction
    │
    └──▶ n8n Webhook (Print receipts)
```

## User Flow

```
1. Launch → Connect WisePad E
2. New Sale → Search/select client
3. Select invoice or enter amount
4. Charge → Present card
5. Success → Print receipt
```

## Development

See [PLAN.md](PLAN.md) for implementation details and roadmap.

## License

MIT - see [LICENSE](LICENSE)
