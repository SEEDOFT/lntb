# LNTB Phase 1 Progress

Last updated: 2026-07-31

## Overall status

Phase 1 functional implementation is complete across the Laravel API and
Flutter application. Focused backend verification passes. Final mobile release
validation on physical Android and iOS devices remains pending.

## Completed

### Authentication and application entry

- Phone or email registration and password login
- Google authentication
- Active account creation and Laravel Sanctum sessions
- Three-page onboarding before login on first installation
- Skip hidden on the final onboarding page
- Authenticated splash routing and FCM installation synchronization

### Offline-ready mobile foundation

- Bundled Noto Sans Khmer, Noto Sans, and Kantumruy Pro fonts
- Locale-aware typography with Khmer line-height and Latin numeric styles
- Native Android font resources for first-launch availability
- Khmer and English translations across current Phase 1 screens
- Phase 1 navigation: Home, Owned Devices, Shared Access, History, Profile

### Device claim and demo environment

- Seller-prepared, owner-bound, one-time QR device activation
- Full-screen QR scanner with camera, scan target, flashlight, and gallery import
- Immediate return of valid LNTB QR data to the claim form
- Idempotent local/testing `app:seed-test-data` command with a complete mobile dataset
- Fixed owner candidate and reset-safe test dataset

### Ownership and shared access

- One primary owner per device
- Up to five active shared users, excluding the owner
- Active registered-user lookup by normalized phone or email
- Owner-only grant, revoke, and active-user management
- Re-grant after revoke with audit records retained
- Immediate authorization loss after revoke
- Owned and actively shared devices separated in the mobile experience

### Device control

- Owner and active shared-user single-device controls and history
- Zone board grouped by normalized device placement
- Unassigned-device group and owner-only placement editing
- Online-device selection, per-zone selection, clear, and cancel actions
- Review and confirmation before controlling selected devices
- Partial-success batch endpoint for 1–20 unique device IDs
- Per-device accepted/failed results and ordinary pending control records
- Successful selections clear while failures remain available for retry

### Device power rating

- `rated_power_watts` column on devices (owner-set power draw per hour)
- Owner-only `PATCH /devices/{id}` support and validation for the rating
- `rated_power_watts` returned in the device resource
- `device:provision --power=` option for provisioning-time rating
- Test dataset seeds per-device ratings (fan, roof, camera, water meter/pump)
- Mobile device-detail tile shows the rating; owners edit it inline with a
  watts-per-hour dialog, shared users read-only
- Rating is the basis for runtime energy estimation (kWh = watts × hours / 1000)

## Verification completed

- Full backend Pest suite: 61 passed, 195 assertions
- Full Flutter test suite: 29 passed (including new `rated_power_watts` parsing)
- `dart analyze lib` clean
- PHP formatting, PHP syntax checks, and repository diff checks completed
  successfully

## Remaining release checks

- Run the focused Flutter zone-grouping/widget tests when the FVM runner is
  responsive; previous short-timeout runs did not complete.
- Run focused Dart analysis with FVM; the latest short-timeout run did not
  complete and reported no result.
- Smoke-test QR camera permission, photo import, and flashlight on physical
  Android and iOS devices.
- Smoke-test Khmer text scaling and first-launch bundled-font rendering.
- Verify twelve-device zone selection and partial batch failures against a
  deployed API environment.

MQTT delivery, realtime state, telemetry, automation, crop monitoring, and
later farm modules remain outside Phase 1.
