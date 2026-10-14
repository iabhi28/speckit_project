# Research: Mobile OTP Authentication

## Decision: Implement one unified mobile-number flow; after successful verification, sign in an existing account or create a new account.

**Rationale**: This is the specified user journey and avoids leaking account existence before proof of number control.

**Alternatives considered**: Separate sign-in and registration entry points were rejected because they duplicate UX and can disclose account existence.

## Decision: Normalize India-only mobile numbers to a single canonical E.164 representation before account lookup or challenge creation.

**Rationale**: Local and `+91` input forms must map to one account key and cannot create duplicates.

**Alternatives considered**: Preserving the original input was rejected because formatting differences would fragment identity records.

## Decision: Store only a hash of each short-lived code in the Identity service, with one-time consumption, expiry, resend cooldown, failed-attempt lockout, and per-number/per-source request limits enforced server-side.

**Rationale**: The cleartext code is needed only at delivery; server enforcement cannot be bypassed by browser refresh or concurrent requests.

**Alternatives considered**: Client-side timers/attempt counters were rejected because they are not security boundaries. Persisting the clear OTP was rejected because it unnecessarily exposes authentication secrets.

## Decision: Use a replaceable SMS delivery adapter configured only in the Identity service runtime.

**Rationale**: SMS delivery is required but the specification does not mandate a vendor. The adapter isolates vendor protocol/credentials while keeping product-facing behavior and versioned Identity contracts stable.

**Alternatives considered**: Embedding a provider SDK into the frontend or hard-coding a vendor in domain logic was rejected because it exposes secrets and tightly couples the service.

## Decision: Create a session only after successful OTP verification and set it in a secure, HTTP-only, SameSite cookie.

**Rationale**: This keeps session credentials inaccessible to browser scripts and prevents an unverified user from being treated as authenticated.

**Alternatives considered**: Local storage or a client-created authenticated state was rejected because it is not a trusted authentication boundary.

## Decision: Validate and constrain the post-authentication destination to a same-origin relative path.

**Rationale**: The required return-to-origin flow must not become an open redirect.

**Alternatives considered**: Accepting arbitrary redirect URLs was rejected because it can redirect users to attacker-controlled sites.
