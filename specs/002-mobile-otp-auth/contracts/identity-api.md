# Identity API Contract

Identity routes are versioned under `/api/v1/identity`. Identity service owns account, challenge, rate-limit, and session data in MySQL. It is the only service permitted to read or write those tables.

## `POST /api/v1/identity/otp-requests`

Request: `{ "mobile": "string" }`

Behavior:
- Normalize and validate an Indian mobile number.
- Apply per-number and per-source request limits and the 60-second resend cooldown.
- Generate a six-digit, single-use code that expires after five minutes; persist only its hash.
- Deliver through the server-configured SMS adapter.
- Return a generic acknowledgment that does not reveal whether an account already exists.

Success: `{ "requestId": "opaque", "resendAvailableAt": "ISO-8601" }`.
Errors include invalid mobile, cooldown/rate-limited with retry time, and delivery/service failure. Delivery failure must not create an authenticated session or claim successful delivery.

## `POST /api/v1/identity/otp-verifications`

Request: `{ "requestId": "opaque", "mobile": "string", "code": "string" }`

Behavior:
- Confirm request ID and normalized mobile match.
- Reject expired, consumed, invalid, or blocked challenges.
- On five incorrect attempts, block verification for 15 minutes.
- On valid code, consume it atomically, create the account if absent, and create a session.
- Set the session in a secure, HTTP-only, SameSite cookie; return no session secret in JSON.

Success: `{ "authenticated": true, "redirect": "/safe-relative-path" }`.
Failure responses use generic verification messaging and never reveal account existence.

## `POST /api/v1/identity/logout`

Revokes the active session and clears the cookie. Subsequent protected requests require a new valid session.

## Error semantics

Use explicit validation/rate-limit/unauthorized/service errors. Never represent provider delivery or verification failure as successful registration or sign-in.
