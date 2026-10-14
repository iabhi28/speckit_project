# Data Model: Mobile OTP Authentication

All records below are owned by the independently deployable Identity service and stored in its MySQL schema.

## CustomerAccount

- `id`: primary key.
- `mobile_e164`: canonical, unique Indian mobile number.
- `created_at`, `updated_at`: timestamps.
- Validation: account creation occurs only after successful OTP verification; registration stores no required profile fields beyond verified mobile.

## AuthenticationRequest

- `id`: opaque request/challenge identifier.
- `mobile_e164`: normalized target number.
- `code_hash`: one-way hash of the short-lived code; never return or log the code.
- `created_at`, `expires_at`: code valid for five minutes.
- `resend_available_at`: at least 60 seconds after code request.
- `failed_attempts`: nonnegative counter.
- `blocked_until`: set for 15 minutes after five incorrect submissions.
- `consumed_at`: set once on successful verification.
- `delivery_state`: pending, sent, or failed.
- Validation: a challenge can verify only its associated canonical number; successful challenge is single-use; older active challenges are invalidated or superseded deterministically.

## AuthenticatedSession

- `id`: session identifier stored only in a secure cookie.
- `customer_id`: CustomerAccount foreign key.
- `created_at`, `expires_at`, `revoked_at`.
- Validation: created only after successful code verification; invalidated on sign-out/expiry; session policy is configured by Identity service.

## Relationships and transitions

- CustomerAccount has many AuthenticatedSessions.
- Mobile number has many historical AuthenticationRequests.
- AuthenticationRequest: pending -> sent -> consumed; pending/sent -> expired; delivery failure stays unauthenticated; five invalid submissions -> blocked for 15 minutes.
- CustomerAccount uniqueness is enforced on `mobile_e164`.
