# Quickstart: Mobile OTP Authentication Validation

## Prerequisites

- Node.js/npm and project dependencies.
- Identity service with MySQL configured and a test SMS adapter/transport.
- Test mobile number and code delivery available in the isolated test environment.

## Run

```bash
npm run dev
npm test -- --run mobile-otp
npm run lint
```

Coverage must exceed 90% for statements, branches, functions, and lines.

## End-to-end checks

1. Submit local-format and `+91` forms of the same Indian number; verify they normalize to the same identity.
2. Request a code; verify generic response, SMS adapter delivery, five-minute expiry, and one-minute resend cooldown.
3. Verify an existing-account code; confirm session creation and safe return to the requested same-origin page.
4. Verify a new-account code; confirm account creation happens only after correct verification.
5. Submit wrong codes five times; verify a 15-minute block. Test expired/reused/mismatched-number codes.
6. Simulate SMS and Identity service failures; confirm no account/session success state is shown.
7. Log out and confirm the session is revoked.

Use the [Identity API contract](./contracts/identity-api.md) and [data model](./data-model.md) as the source for request/state expectations.
