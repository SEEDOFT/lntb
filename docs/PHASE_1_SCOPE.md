# LNTB Phase 1 Scope

## Included

### Authentication

- Register with phone number and password
- Register with email and password
- Sign in with phone/email and password
- Google sign-in
- Sanctum token creation
- Current-user profile
- Logout (revoke current token)
- Persist one welcome notification when a user account is first created
- Synchronize FCM tokens for multiple signed-in mobile installations
- Queue idempotent per-installation welcome delivery with native foreground alerts
- List and update the authenticated user's in-app notifications

### Device ownership

- Device-type lookup
- Device-status lookup
- Device inventory record
- MAC-address normalization
- Generated claim-code verification
- Claim device
- Assign owner
- List owned devices
- View device details

### Shared device access

- Grant access to an existing registered user (immediate active status)
- Maximum five active shared users per device
- Revoke shared access
- List users with access

### Device control

- Create device-control record
- Validate owner/shared-user access
- Store control type and control data
- Update command status
- View control history

## Excluded

- Sensor telemetry
- Soil moisture monitoring
- Temperature and humidity monitoring
- Water and electricity measurement
- Irrigation automation
- Camera image upload
- Tomato ripeness AI
- Farmer AI assistant
- Farm and crop-cycle management
- Notifications other than the welcome notification and existing in-app notification API
- Realtime WebSocket updates
- MQTT broker integration
- Offline synchronization
- Reports and analytics
- Billing and subscriptions
- Ownership transfer
- Granular per-command permissions
- User roles beyond owner and shared user
- Phone OTP login
- User sessions (separate table)
- Invitation accept/reject flow
- Device rename endpoint
- Idempotency-key header on controls
- Correlation ID middleware

## Phase 1 completion criteria

1. A user can register and enter the application.
2. A user can sign in with password or Google.
3. A main user can claim an available device.
4. The system prevents a second owner from claiming the same device.
5. The owner can grant access to up to five registered users.
6. The sixth active shared user is rejected.
7. The owner can revoke access.
8. The owner and active shared users can create device-control commands.
9. Unauthorized users cannot view or control the device.
10. Control history is stored.
11. Pest tests pass.
