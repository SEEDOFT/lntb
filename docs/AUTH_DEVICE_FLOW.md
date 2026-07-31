# Authentication and Device Flow

## Registration with phone number

```text
User submits registration data (name, country_code, phone_number, password)
        ↓
Form Request validates data
        ↓
Service resolves active user status
        ↓
Password is hashed
        ↓
User is created
        ↓
Sanctum token is created
        ↓
User enters the application
```

The user is not created in a pending state.

After the account transaction commits, Laravel creates one unread welcome
notification. Each active mobile installation gets an idempotent push-delivery
record and dedicated queue job. If the registration token becomes available
after authentication, the authenticated token-sync endpoint queues the still
unsent welcome notification. Registration remains successful when FCM is
unavailable.

Flutter keeps a stable installation key in secure storage and synchronizes its
FCM token during authentication, authenticated startup, and token rotation.
Logout revokes only the current installation, so other signed-in phones remain
eligible for notifications.

## Google authentication

```text
Flutter receives a Google OAuth access token
        ↓
Flutter sends the access token to Laravel
        ↓
Laravel retrieves and validates the Google identity through Socialite
        ↓
Find user by google_id or approved email link
        ↓
Create or update active user
        ↓
Create Sanctum token
        ↓
Return authenticated user
```

Do not trust a Google email without validating the Google access token. Automatic account linking requires Google to report the email as verified.

## Logout

```text
Authenticated user requests logout
        ↓
Current Sanctum token is deleted
        ↓
Return success
```

## Device claim

Seller preparation:

```text
Resolve available device and active registered customer
        ↓
Lock device and revoke prior unused activation
        ↓
Generate 256-bit token and store only its hash
        ↓
Bind activation to intended_user_id with expiry
        ↓
Generate one-time versioned QR
```

Customer input:

- `device_ref`
- `activation_token`
- optional `name`

Transaction:

```text
Lock activation and device rows
        ↓
Confirm device exists
        ↓
Confirm device status = available
        ↓
Confirm owner_user_id is null
        ↓
Confirm authenticated user matches intended_user_id
        ↓
Verify token hash, expiry, revocation, consumption, and attempt limit
        ↓
Resolve active device status
        ↓
Set owner_user_id
        ↓
Set device_status_id to active
        ↓
Set claimed_at and consume the activation
        ↓
Commit
```

## Grant shared access

Input:

- `device_id`
- `phone_number` or `email`

Transaction:

```text
Lock device row
        ↓
Confirm requester is device owner
        ↓
Find registered user
        ↓
Reject granting to owner
        ↓
Reject if user already has active access
        ↓
Count active shared users
        ↓
Reject when count >= 5
        ↓
Create access record with active status
        ↓
Commit
```

Access is active immediately — there is no invitation accept/reject flow.

## Revoke access

Only the owner may revoke. Status changes to `revoked` and `revoked_at` is recorded.

## Device-control authorization

A user may create a device command when they are the device owner or have active shared access.

```text
Validate request
        ↓
Load device
        ↓
Authorize with DevicePolicy
        ↓
Validate supported control_type
        ↓
Resolve pending control status
        ↓
Create device_controls record
        ↓
Publish later to IoT transport
        ↓
Update control status as processing continues
```

The Phase 1 API may initially create and track commands before MQTT integration is added.

## Batch device control

Input:

- 1–20 unique `device_ids`
- one supported `control_type`
- optional `control_data`

```text
Validate the complete request
        ↓
Load requested device records without exposing missing devices
        ↓
For each ID, authorize control with DevicePolicy
        ↓
Create a normal pending control record for each authorized device
        ↓
Return accepted or failed result for every requested ID
```

Batch processing uses partial success. Missing and inaccessible IDs both return
`DEVICE_ACCESS_DENIED`. A failure for one device does not roll back successful
commands for other authorized devices.

## User session model

Sanctum tokens represent logged-in mobile sessions directly. There is no separate `user_sessions` table. The token is created during authentication and deleted on logout.
