# LNTB Phase 1 API Contracts

## API conventions

Base path:

```text
/api/v1
```

Headers:

```http
Accept: application/json
Content-Type: application/json
Authorization: Bearer {token}
```

Success response:

```json
{
  "message": "Operation completed successfully.",
  "data": {}
}
```

Business error:

```json
{
  "message": "Unable to complete the operation.",
  "code": "DEVICE_ALREADY_CLAIMED"
}
```

Validation errors include an `errors` object keyed by request field.

## Authentication endpoints

### Register with phone and password

```http
POST /api/v1/auth/register
```

Request:

```json
{
  "name": "Dara",
  "country_code": "+855",
  "phone_number": "12345678",
  "password": "StrongPassword123!",
  "password_confirmation": "StrongPassword123!",
  "fcm_token": "firebase-registration-token",
  "fcm_device_key": "stable-random-installation-key",
  "device_name": "Dara Android",
  "platform": "android",
  "app_version": "1.0.0"
}
```

### Login with password

```http
POST /api/v1/auth/login
```

Request:

```json
{
  "country_code": "+855",
  "phone_number": "12345678",
  "password": "StrongPassword123!",
  "fcm_token": "firebase-registration-token",
  "fcm_device_key": "stable-random-installation-key",
  "device_name": "Dara Android",
  "platform": "android",
  "app_version": "1.0.0"
}
```

A compound `login` field (e.g. `+85512345678`) may be used instead of `country_code` + `phone_number`.

### Google authentication

```http
POST /api/v1/auth/google
```

Request:

```json
{
  "token": "google-id-token",
  "fcm_token": "firebase-registration-token",
  "fcm_device_key": "stable-random-installation-key",
  "device_name": "Dara Android",
  "platform": "android",
  "app_version": "1.0.0"
}
```

### Current user

```http
GET /api/v1/auth/me
```

### Logout

```http
POST /api/v1/auth/logout
```

Optional request:

```json
{
  "fcm_device_key": "stable-random-installation-key"
}
```

The supplied installation is revoked without affecting other signed-in phones.

### Synchronize an FCM installation

```http
POST /api/v1/auth/fcm-token
```

```json
{
  "fcm_token": "firebase-registration-token",
  "fcm_device_key": "stable-random-installation-key",
  "device_name": "Dara Android",
  "platform": "android",
  "app_version": "1.0.0+1"
}
```

The same request is used after authentication, app startup, and FCM token
rotation. Repeating it updates the same installation.

### Revoke an FCM installation

```http
DELETE /api/v1/auth/fcm-token
```

```json
{
  "fcm_device_key": "stable-random-installation-key"
}
```

## Device endpoints

### List accessible devices

```http
GET /api/v1/devices
```

### Activate seller-prepared device

```http
POST /api/v1/devices/claim
```

Request:

```json
{
  "device_ref": "123e4567-e89b-12d3-a456-426614174000",
  "activation_token": "base64url-encoded-256-bit-token",
  "name": "Smart Farm Controller"
}
```

Business error codes:

- `INVALID_DEVICE_ACTIVATION`

### View device

```http
GET /api/v1/devices/{device}
```

### Update device name or placement

```http
PATCH /api/v1/devices/{device}
```

Owner-only request:

```json
{
  "name": "Greenhouse Controller 1",
  "placement": "Greenhouse A"
}
```

`name` and `placement` are trimmed. Placement may be null or blank to return
the device to the mobile Unassigned group. Active shared users may see these
values but cannot update them.

## Notification endpoints

### List notifications

```http
GET /api/v1/notifications
```

Returns unread/read notifications for the authenticated user and excludes
notifications marked as deleted. Pagination metadata includes `unread_count`,
which is the authoritative unread total across all pages.

### Update notification status

```http
PATCH /api/v1/notifications/{notification}
```

Request:

```json
{
  "status": "read"
}
```

Allowed status codes: `unread`, `read`, `deleted`.
The response metadata includes the updated `unread_count`.

## Shared-access endpoints

### List device users

```http
GET /api/v1/devices/{device}/users
```

### Grant access to registered user

```http
POST /api/v1/devices/{device}/users
```

Request:

```json
{
  "login": "+85598765432"
}
```

Business error codes:

- `DEVICE_OWNER_REQUIRED`
- `USER_NOT_FOUND`
- `OWNER_CANNOT_BE_GRANTED`
- `ACCESS_ALREADY_EXISTS`
- `DEVICE_ACCESS_LIMIT_REACHED`

### Revoke shared access

```http
DELETE /api/v1/devices/{device}/users/{access}
```

## Device-control endpoints

### List control history

```http
GET /api/v1/devices/{device}/controls
```

### Create control command

```http
POST /api/v1/devices/{device}/controls
```

Request:

```json
{
  "control_type": "irrigation.start",
  "control_data": {
    "duration_seconds": 180
  }
}
```

Supported Phase 1 control types: `irrigation.start`, `irrigation.stop`, `fan.start`, `fan.stop`, `roof.open`, `roof.close`, `camera.capture`. `control_data` must be null or a JSON object no larger than 8 KB.

### Create control commands for selected devices

```http
POST /api/v1/devices/controls/batch
```

Request:

```json
{
  "device_ids": [12, 14, 18],
  "control_type": "irrigation.start",
  "control_data": {}
}
```

`device_ids` must contain 1–20 unique positive IDs. The endpoint authorizes
each device independently and creates the same ordinary pending control record
used by the single-device endpoint. Results are partial: authorized devices may
succeed when another ID is missing or inaccessible. Missing and inaccessible
devices both return `DEVICE_ACCESS_DENIED` without exposing device details.

```json
{
  "data": {
    "accepted_count": 2,
    "failed_count": 1,
    "results": [
      {"device_id": 12, "accepted": true, "control": {}},
      {"device_id": 14, "accepted": true, "control": {}},
      {"device_id": 18, "accepted": false, "error_code": "DEVICE_ACCESS_DENIED"}
    ]
  }
}
```

### View control command

```http
GET /api/v1/devices/{device}/controls/{control}
```

## Operational commands

Create or refresh the complete authenticated mobile test dataset in a local or
testing environment:

```text
php artisan app:seed-test-data
php artisan app:seed-test-data --reset
```

The dataset is owned by the phone `+855 010000099` (password `LntbTest123!`) and
contains Sokha Tomato Farm, four active devices, current sensor readings,
usage, notifications, and completed, pending, and failed control records. The
command is idempotent, refuses non-local environments, and reset affects only
this dedicated dataset. Local `DatabaseSeeder` runs include it when
`LNTB_SEED_TEST_DATA=true`; automated tests opt in explicitly so lookup seeding
does not alter unrelated table-count assertions.

`GET /api/v1/farms/{farm}/dashboard` additively returns `devices`, `activity`,
and `warnings`. Metrics include their source device and recorded time. Mobile
clients must not substitute embedded operational values when this request
fails.

Provision inventory before claim:

```text
php artisan device:provision {serial} {mac} --type=smart_farm_controller --name="..." --firmware="..."
```

Prepare an owner-bound activation after the customer registers:

```text
php artisan device:prepare-activation {serial} {customer_login} --operator=seller-id
```

The command writes a one-time QR under `storage/app/activations`. The raw token
exists only inside that QR; the database stores its hash. Sanctum tokens expire
after 30 days.

Process notification delivery with a persistent worker:

```text
php artisan queue:work database --queue=notifications,default --tries=3
```

Required backend environment configuration:

```text
FIREBASE_PROJECT=app
FIREBASE_CREDENTIALS=storage/app/firebase/service-account.json
FIREBASE_HTTP_CLIENT_TIMEOUT=10
```

The service-account JSON must be downloaded from the Firebase project used by
the mobile application and must not be committed.

## Required Form Requests

- `RegisterRequest`
- `LoginRequest`
- `GoogleLoginRequest`
- `ClaimDeviceRequest`
- `GrantDeviceUserRequest`
- `CreateDeviceControlRequest`
- `CreateBatchDeviceControlRequest`

## Required API Resources

- `UserResource`
- `DeviceResource`
- `DeviceAccessResource`
- `DeviceControlResource`
