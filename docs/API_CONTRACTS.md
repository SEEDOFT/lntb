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

### Register with password

```http
POST /api/v1/auth/register
```

Request:

```json
{
  "name": "Dara",
  "phone_number": "+85512345678",
  "email": null,
  "password": "StrongPassword123!",
  "password_confirmation": "StrongPassword123!",
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
  "login": "+85512345678",
  "password": "StrongPassword123!",
  "device_name": "Dara Android",
  "platform": "android",
  "app_version": "1.0.0"
}
```

### Google authentication

```http
POST /api/v1/auth/google
```

Request:

```json
{
  "access_token": "google-oauth-access-token",
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

## Device endpoints

### List accessible devices

```http
GET /api/v1/devices
```

### Claim device

```http
POST /api/v1/devices/claim
```

Request:

```json
{
  "mac_address": "AA:BB:CC:DD:EE:FF",
  "claim_code": "ABCD-EFGH-1234",
  "name": "Smart Farm Controller"
}
```

Business error codes:

- `DEVICE_NOT_FOUND`
- `DEVICE_NOT_AVAILABLE`
- `DEVICE_ALREADY_CLAIMED`
- `INVALID_CLAIM_CODE`
- `CLAIM_CODE_ALREADY_USED`

### View device

```http
GET /api/v1/devices/{device}
```

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

### View control command

```http
GET /api/v1/devices/{device}/controls/{control}
```

## Operational commands

Provision inventory before claim:

```text
php artisan device:provision {serial} {mac} --type=smart_farm_controller --name="..." --firmware="..."
```

The generated claim code is displayed once and only its hash is stored. Sanctum tokens expire after 30 days.

## Required Form Requests

- `RegisterRequest`
- `LoginRequest`
- `GoogleLoginRequest`
- `ClaimDeviceRequest`
- `GrantDeviceUserRequest`
- `CreateDeviceControlRequest`

## Required API Resources

- `UserResource`
- `DeviceResource`
- `DeviceAccessResource`
- `DeviceControlResource`
