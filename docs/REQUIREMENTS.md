# LNTB Phase 1 Requirements

## Functional requirements

### Authentication

#### FR-AUTH-001 — Register with phone number and password

The user provides name, phone number, password, and password confirmation. The system validates uniqueness and creates an active user.

#### FR-AUTH-002 — Register with email and password

The user provides name, email, password, and password confirmation. The system validates uniqueness and creates an active user.

#### FR-AUTH-003 — Sign in with phone number or email and password

The system authenticates valid credentials and creates a Sanctum token.

#### FR-AUTH-004 — Sign in with Google

The system validates the Google identity and either links it to an allowed existing account or creates a new active user.

#### FR-AUTH-005 — Sign out

The current Sanctum token is deleted.

### Notifications

#### FR-NOTIFY-001 — Welcome notification

After a new account transaction commits, the system creates exactly one unread
welcome notification for that user. When the user has an FCM registration
token, push delivery is queued and retried independently from registration.
Push failure must not roll back account creation or remove the in-app
notification.

Each signed-in mobile installation has its own stable device key and FCM token.
Token rotation updates that installation, logout revokes only the current
installation, and per-device delivery tracking prevents duplicate retry sends.

### Device registration

#### FR-DEV-001 — Claim device

The main user submits the normalized MAC address and generated claim code.

The system verifies that:

- the device exists;
- the device status is available;
- the device has no owner;
- the claim code matches the stored hash;
- the claim code has not already been used.

On success:

- `owner_user_id` is assigned;
- device status changes to active;
- `claimed_at` is recorded;
- the claim code is invalidated.

#### FR-DEV-002 — List owned and shared devices

The authenticated user can view devices they own and devices shared with them using active access.

#### FR-DEV-003 — View device details

The user can view a device only when they own it or have active shared access.

### Device sharing

#### FR-SHARE-001 — Grant access to registered user

Only the owner may grant access to another registered user. The owner identifies the user by phone number or email. Access is active immediately — no invitation accept/reject flow.

#### FR-SHARE-002 — Maximum five shared users

The system must prevent the total of active shared users from exceeding five per device. The owner is not included in this count.

#### FR-SHARE-003 — Revoke access

Only the owner may revoke access. Access status becomes revoked.

### Device control

#### FR-CTRL-001 — Create control command

The owner or a user with active shared access may create a device-control command.

#### FR-CTRL-002 — Validate control authorization

The system must reject commands from users without permission.

#### FR-CTRL-003 — Track control status

A command has a lookup-table state: `pending`, `completed`, or `failed`.

#### FR-CTRL-004 — View control history

Authorized users may view device-control history.

## Non-functional requirements

### NFR-001 — No foreign-key constraints

Reference columns use `unsignedBigInteger` with indexes. Laravel validates referenced records.

### NFR-002 — No enums

All lifecycle states use lookup tables.

### NFR-003 — No boolean state fields

Use lookup state tables and timestamps instead of fields such as `is_active`.

### NFR-004 — Security

- Passwords use Laravel hashing.
- Claim codes are never stored in plaintext.
- Sanctum tokens are stored and revoked using Laravel mechanisms.
- Device operations require authorization.
- Sensitive operations use rate limiting.

### NFR-005 — Consistency

Device claiming and device-sharing capacity checks must run inside database transactions.

### NFR-006 — API responses

All API endpoints return a consistent JSON structure.

### NFR-007 — Testing

Pest tests must cover successful registration, duplicate identity, Google authentication, login failure, device claim success/failure, owner-only access grants, five-user limit, access revocation, and authorized/unauthorized control.

### NFR-008 — Notification delivery

- FCM delivery runs through a persistent Laravel queue worker.
- Firebase Admin credentials are read from environment configuration and are never committed.
- Welcome notification creation is idempotent per user.
- Invalid FCM registration tokens are cleared without logging their value.
