# LNTB Project Instructions

## Required reading

Before modifying code, read these files:

1. `docs/PROJECT_CONTEXT.md`
2. `docs/REQUIREMENTS.md`
3. `docs/PHASE_1_SCOPE.md`
4. `docs/DATABASE_DESIGN.md`
5. `docs/AUTH_DEVICE_FLOW.md`
6. `docs/API_CONTRACTS.md`
7. `docs/architecture/SYSTEM_ARCHITECTURE.md`

## Current development phase

Implement Phase 1 only:

- User registration and login
- Google authentication
- Phone number or email with password authentication
- Active user account immediately after successful registration
- User sessions using Laravel Sanctum
- IoT device registration by MAC address and generated claim code
- One primary owner per device
- Owner may share device access with at most five other registered users
- Owner and authorized users may control the IoT device
- Device-control history

Do not implement crop monitoring, telemetry, AI, irrigation automation, harvest, costs, reports, or later modules unless explicitly requested.

## Laravel rules

- Use the latest stable Laravel version.
- Use Form Requests for validation.
- Use API Resources for JSON output.
- Use Sanctum for mobile API authentication.
- Use Socialite for Google authentication.
- Use service classes for business operations.
- Use Policies for device authorization.
- Use database transactions for device claiming and device sharing.
- Use Pest for unit and feature tests.
- Use strict PHP typing where practical.
- Return consistent API error structures.
- Add indexes for frequently queried columns.

## Database rules

- Do not create foreign-key constraints.
- Use `unsignedBigInteger` for reference columns.
- Do not use database enums.
- Do not use boolean state columns.
- Represent state using lookup tables and numeric IDs.
- Resolve lookup records by immutable `code`, not hard-coded IDs.
- Hash passwords and device claim codes.
- Normalize MAC addresses as `AA:BB:CC:DD:EE:FF`.

## Device access rules

- One device has one main owner.
- The owner is stored in `devices.owner_user_id`.
- The owner is not counted in the five shared-user limit.
- Count both invited and active shared users.
- Only the owner may invite, revoke, or manage shared users.
- A shared user cannot invite another user.
- A user may control a device only when they are the owner or have active shared access.

## Completion rules

A feature is complete only when:

- Validation exists.
- Authorization exists.
- Error handling exists.
- Database transaction boundaries are correct.
- Pest tests cover success and failure paths.
- API contracts remain consistent.
- Documentation is updated when behavior changes.
