# LNTB Project Context

## Product summary

LNTB is a smart-agriculture IoT platform for Cambodian farmers.

The complete long-term product will monitor crop conditions, control farm equipment, detect tomato ripeness, provide alerts, and include an AI farming assistant. The current implementation is intentionally limited to Phase 1.

## Phase 1 product goal

Phase 1 establishes the identity, device ownership, device sharing, and device-control foundation.

A customer who purchases an LNTB IoT product must be able to:

1. Register or sign in.
2. Register the purchased IoT device using:
   - the device MAC address;
   - the generated claim code supplied with the product.
3. Become the main owner of that device.
4. Invite up to five other registered users.
5. Allow those users to control the device.
6. Review device-control history.
7. Revoke shared access.

## User types

### Main user / device owner

The purchaser of the physical IoT product.

The owner can:

- Claim an unowned device.
- Rename the device.
- View the device.
- Control the device.
- Invite up to five users.
- Revoke shared access.
- View control history.

### Shared user

A registered user granted access by the owner.

The shared user can:

- View assigned devices.
- Accept or reject invitations.
- Control devices after access becomes active.
- View control history for devices they can access.

The shared user cannot:

- Transfer ownership.
- Claim the device as owner.
- Invite other users.
- Revoke another user.
- Change ownership information.

## Authentication methods

Phase 1 supports:

- Phone number and password
- Email and password
- Google authentication

A successfully registered user becomes active immediately and proceeds into the application.

Phone OTP authentication is not part of the current migration baseline. It may be added later.

## Technology direction

- Backend: Laravel REST API
- Mobile application: Flutter
- Authentication: Laravel Sanctum
- Google authentication: Laravel Socialite
- Database: PostgreSQL or MySQL
- IoT command delivery: added after the Phase 1 API foundation, typically MQTT
- Realtime updates: added later through Laravel broadcasting/Reverb
