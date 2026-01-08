# dOS-POS

**digiBandit Mobile POS** - React Native app for WisePad E Bluetooth terminal.

---

## Guidelines

- **No Co-Authored-By**: Do not include `Co-Authored-By: Claude...` in commit messages
- **ITFlow format**: Identifiers use `t`/`i`/`q` + YY + digits (e.g. `i260045`), no hyphens

---

## Tech Stack

- React Native + TypeScript
- Stripe Terminal SDK (WisePad E via Bluetooth)
- Backend: dOS Apps Script API
- Receipt printing via n8n webhook

## Structure

```
example-app/
├── src/
│   ├── api/api.ts       - Backend communication
│   ├── components/      - UI components
│   └── utils/           - Helpers
├── ios/                 - iOS native
├── android/             - Android native
└── PLAN.md              - Implementation plan
```

## Build

```bash
npm install
npx expo start           # Development
npx eas build            # Production builds
```

---

## digiBandit Ecosystem

| Repo | Purpose |
|------|---------|
| **dOS** | Employee portal, POS backend |
| **digiDocs** | Documentation library (225+ docs) |
| **xdm** | Tactical RMM scripts |
| **xdmu** | System tray utility |
| **itflow** | PSA/ticketing - invoice lookup |
| **n8n** | Receipt printing webhook |
| **bluebox** | Client network appliance |
| **freepbx** | VoIP phone system |
| **netmaker** | VPN infrastructure |
| **nextcloud** | File storage |
| **pxe** | Network boot server |
| **rocketchat** | Team communication |
| **rustdesk** | Remote desktop |
| **vaultwarden** | Password management |
| **wp** | WordPress sites |
| **zabbix** | Infrastructure monitoring |
| **cal** | Planning calendar |

All repos: `github.com/dbits-db/`
