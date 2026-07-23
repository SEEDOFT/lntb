# Authentication and Device Flow

## Registration with phone number or email

```text
User submits registration data
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

Input:

- `mac_address`
- `claim_code`
- optional `name`

Transaction:

```text
Normalize MAC address
        ↓
Lock device row
        ↓
Confirm device exists
        ↓
Confirm device status = available
        ↓
Confirm owner_user_id is null
        ↓
Confirm claim_code_used_at is null
        ↓
Verify submitted code against claim_code_hash
        ↓
Resolve active device status
        ↓
Set owner_user_id
        ↓
Set device_status_id to active
        ↓
Set claimed_at and claim_code_used_at
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

## User session model

Sanctum tokens represent logged-in mobile sessions directly. There is no separate `user_sessions` table. The token is created during authentication and deleted on logout.
