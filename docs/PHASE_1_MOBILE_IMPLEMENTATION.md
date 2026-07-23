# Phase 1 Mobile Experience

## Scope

The Phase 1 mobile application covers authentication, device ownership,
shared access, device commands, command history, notifications, and account
session management. OTP verification, telemetry, realtime device state,
automation, crop monitoring, invitations, reports, and later farming modules
remain out of scope.

## Experience

- The splash uses a project-owned smart-farm illustration with the official
  LNTB logo and localized text rendered by Flutter.
- Session validation runs while the splash remains visible for its minimum
  display duration.
- First-time installations see one welcome screen rather than a carousel.
- Main navigation contains Home, Devices, Shared Access, History, and Profile.
- Blue is used for primary actions and green for successful or online states.
- Khmer is the default language and every Phase 1 screen retains English
  translations.

## Authentication

Registration and login are separate modes. Users may authenticate with:

- Cambodia phone number (`+855`) and password
- normalized email and password
- Google

Phone and email registration do not use OTP. Registration creates an active
account immediately. A registration may contain an optional alternate phone or
email. Google authentication returns `is_new_account` so the mobile app can
show the account-created screen only for a newly created account.

The `users` table receives nullable, unique `email`, nullable
`email_verified_at`, and nullable `country_code` through an additive migration.
Existing phone clients keep their current request contract.

## Mobile API Contract

All routes are under `/api/v1` and authenticated routes use Sanctum.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| POST | `/auth/register` | Register with phone or email and password |
| POST | `/auth/login` | Login with phone or email and password |
| POST | `/auth/google` | Login or create an account with Google |
| GET | `/auth/me` | Load the current account |
| POST | `/auth/logout` | Revoke the current device installation and session |
| GET | `/devices` | List owned and actively shared devices |
| POST | `/devices/claim` | Claim using normalized MAC and claim code |
| GET | `/devices/{id}` | Load authorized device details |
| GET/POST | `/devices/{id}/controls` | Read or create device commands |
| GET | `/controls` | Paginated history across all authorized devices |
| GET/POST | `/devices/{id}/users` | List or grant shared access as owner |
| DELETE | `/devices/{id}/users/{access}` | Revoke a shared-access record |
| GET | `/notifications` | List in-app notifications and unread count |

Authentication responses add `is_new_account` without removing existing
fields. Device commands remain limited to `irrigation.start`,
`irrigation.stop`, `fan.start`, `fan.stop`, `roof.open`, `roof.close`, and
`camera.capture`.

## Architecture

Flutter screens consume typed `AppUser`, `DeviceModel`, `DeviceAccess`, and
`ControlRecord` objects. Account operations are isolated in
`AccountRepository`, while device claiming, access, controls, and history are
isolated in `DeviceRepository`. Controllers own loading, empty, error, and
command states; widgets render those states and do not create mock data. A
control toggle reflects the latest stored command and is not a claim of
realtime hardware state.

Laravel keeps validation in Form Requests, authentication behavior in
`AuthService`, access authorization in device policies, serialized output in
API Resources, and notification transport in queued notification classes.
The global history query includes only devices owned by the user or devices
with active shared access.

## Claim QR Payload

Supported QR labels contain JSON:

```json
{
  "mac_address": "AA:BB:CC:DD:EE:FF",
  "claim_code": "ABCD-EFGH-1234",
  "name": "Smart Farm #1"
}
```

`name` is optional. Invalid QR data leaves manual MAC/code entry available.

## Full Proposal Expansion

The mobile application now defines API-backed screen contracts for configured
farms, crop-cycle summaries, daily tasks, sensor readings, irrigation status,
water/electricity usage, ripeness results, digital logs, harvest records, and a
read-only farm assistant. The main navigation is Home, Farm, Devices, History,
and Profile.

Laravel persists these domains using lookup-driven tables without foreign-key
constraints or database enums. Farms remain backend-configured; the mobile app
does not create farms or crop cycles. Unsupported or unconfigured services
return explicit unavailable states rather than sample data.
