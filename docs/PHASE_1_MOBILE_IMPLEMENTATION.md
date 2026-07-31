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
- First-time installations see three onboarding pages before login.
- Skip is available before the final onboarding page and hidden on the final
  page.
- Main navigation contains Home, Devices, History, and Profile.
- Emerald is used for primary actions and green for successful or online states.
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
| POST | `/devices/claim` | Activate using an owner-bound one-time QR token |
| GET | `/devices/{id}` | Load authorized device details |
| GET/POST | `/devices/{id}/controls` | Read or create device commands |
| POST | `/devices/controls/batch` | Control 1–20 selected authorized devices |
| GET | `/controls` | Paginated history across all authorized devices |
| GET/POST | `/devices/{id}/users` | List or grant shared access as owner |
| DELETE | `/devices/{id}/users/{access}` | Revoke a shared-access record |
| GET | `/notifications` | List in-app notifications and unread count |
| GET | `/farms` | List the authenticated user's configured farm |
| GET | `/farms/{id}/dashboard` | Load farm metrics, devices, warnings, and activity |

Authentication responses add `is_new_account` without removing existing
fields. Device commands remain limited to `irrigation.start`,
`irrigation.stop`, `fan.start`, `fan.stop`, `roof.open`, `roof.close`, and
`camera.capture`.

## Architecture

Flutter screens consume typed `AppUser`, `DeviceModel`, `DeviceAccess`,
`FarmDashboard`, `DashboardMetric`, and `ControlRecord` objects, plus typed
batch-control results. Account operations are isolated in
`AccountRepository`, while device claiming, access, controls, and history are
isolated in `DeviceRepository`; the API-backed farm dashboard is isolated in
`FarmDashboardRepository`. Controllers own loading, empty, error, and command
states. Flutter contains no operational fixture or fallback data. A control
toggle reflects the latest stored API command and is not a claim of realtime
hardware state.

Laravel keeps validation in Form Requests, authentication behavior in
`AuthService`, access authorization in device policies, serialized output in
API Resources, and notification transport in queued notification classes.
The global history query includes only devices owned by the user or devices
with active shared access.

The Device Placement screen is a zone board. It groups case-equivalent,
trimmed placement names together and puts blank placements in Unassigned.
Online owned and active shared devices may be selected. A reviewed batch
command returns accepted or failed status per device; successful selections
clear and failed selections remain for retry. Owners may edit device name and
placement, while shared users have read-only placement access.

## Claim QR Payload

Supported QR labels contain JSON:

```json
{
  "v": 1,
  "device_ref": "123e4567-e89b-12d3-a456-426614174000",
  "activation_token": "base64url-encoded-256-bit-token",
  "device_name": "Greenhouse Controller"
}
```

`device_name` is optional. Legacy MAC-address and claim-code payloads are
rejected.

## Later-module code

The repository contains API-backed screen contracts for configured
farms, crop-cycle summaries, daily tasks, sensor readings, irrigation status,
water/electricity usage, ripeness results, digital logs, harvest records, and a
read-only farm assistant. These later modules are not part of Phase 1 and are
not included in Phase 1 navigation.

Laravel persists these domains using lookup-driven tables without foreign-key
constraints or database enums. Farms remain backend-configured; the mobile app
does not create farms or crop cycles. Unsupported or unconfigured services
return explicit unavailable states rather than sample data.
