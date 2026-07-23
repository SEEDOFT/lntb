# LNTB Phase 1 Database Design

## Database conventions

- Primary keys: `bigIncrements`
- Reference columns: `unsignedBigInteger`
- Reference columns are indexed
- No foreign-key constraints
- No enums
- No boolean lifecycle columns
- Lookup values use immutable `code`
- Timestamps use Laravel `timestamps()`

## `user_statuses`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| code | VARCHAR(50) | Unique |
| name | VARCHAR(100) | Required |
| description | VARCHAR(255) | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

Initial codes: `active`, `suspended`, `closed`.

## `users`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| name | VARCHAR(120) | Required |
| phone_number | VARCHAR(20) | Nullable, unique |
| email | VARCHAR(255) | Nullable, unique |
| google_id | VARCHAR(255) | Nullable, unique |
| password | VARCHAR(255) | Nullable |
| user_status_id | BIGINT UNSIGNED | Indexed |
| phone_verified_at | TIMESTAMP | Nullable |
| email_verified_at | TIMESTAMP | Nullable |
| last_login_at | TIMESTAMP | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

At least one authentication path must exist: phone number plus password, email plus password, or Google ID.

## `user_fcm_tokens`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| user_id | BIGINT UNSIGNED | Indexed |
| device_key | VARCHAR(128) | Unique |
| fcm_token | VARCHAR(255) | Nullable, unique |
| platform | VARCHAR(50) | Nullable |
| device_name | VARCHAR(120) | Nullable |
| app_version | VARCHAR(30) | Nullable |
| last_used_at | TIMESTAMP | Nullable |
| revoked_at | TIMESTAMP | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

One user may have multiple active installations. Token rotation updates the row
identified by `device_key`; revocation clears the token without affecting other
installations.

## `device_types`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| code | VARCHAR(50) | Unique |
| name | VARCHAR(120) | Required |
| description | VARCHAR(255) | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

Initial codes: `smart_farm_controller`, `camera`, `water_energy_meter`.

## `device_statuses`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| code | VARCHAR(50) | Unique |
| name | VARCHAR(120) | Required |
| description | VARCHAR(255) | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

Initial codes: `available`, `active`, `suspended`, `maintenance`, `retired`.

## `devices`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| device_type_id | BIGINT UNSIGNED | Indexed |
| device_status_id | BIGINT UNSIGNED | Indexed |
| owner_user_id | BIGINT UNSIGNED | Nullable, indexed |
| name | VARCHAR(120) | Nullable |
| serial_number | VARCHAR(100) | Unique |
| mac_address | VARCHAR(17) | Unique |
| claim_code_hash | VARCHAR(255) | Required |
| firmware_version | VARCHAR(50) | Nullable |
| claim_code_used_at | TIMESTAMP | Nullable |
| claimed_at | TIMESTAMP | Nullable |
| last_seen_at | TIMESTAMP | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

Rules:

- Store normalized MAC addresses only.
- Store claim-code hash only.
- `owner_user_id` is null before claim.
- `claim_code_used_at` prevents code reuse.

## `device_access_statuses`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| code | VARCHAR(50) | Unique |
| name | VARCHAR(120) | Required |
| description | VARCHAR(255) | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

Initial codes: `active`, `revoked`.

## `device_user_access`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| device_id | BIGINT UNSIGNED | Indexed |
| user_id | BIGINT UNSIGNED | Indexed |
| granted_by_user_id | BIGINT UNSIGNED | Indexed |
| device_access_status_id | BIGINT UNSIGNED | Indexed |
| granted_at | TIMESTAMP | Required |
| revoked_at | TIMESTAMP | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

Unique index: `(device_id, user_id)`.

Rules:

- Owner is not inserted into this table.
- Only `active` records count toward the five-user limit.
- Only the device owner may grant or revoke access.

## `device_control_statuses`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| code | VARCHAR(50) | Unique |
| name | VARCHAR(120) | Required |
| description | VARCHAR(255) | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

Initial codes: `pending`, `completed`, `failed`.

## `notifications`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| user_id | BIGINT UNSIGNED | Indexed |
| deduplication_key | VARCHAR(255) | Nullable, unique |
| notification_type_id | BIGINT UNSIGNED | Indexed |
| notification_status_id | BIGINT UNSIGNED | Indexed |
| title | VARCHAR(255) | Required |
| body | TEXT | Required |
| data | JSON | Nullable |
| push_sent_at | TIMESTAMP | Nullable |
| push_failed_at | TIMESTAMP | Nullable |
| push_failure_message | VARCHAR(500) | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

Welcome notifications use `welcome:user:{user_id}` as their deduplication key.
Read state and push delivery state are intentionally tracked separately.

## `notification_push_deliveries`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| notification_id | BIGINT UNSIGNED | Indexed |
| user_fcm_token_id | BIGINT UNSIGNED | Indexed |
| queued_at | TIMESTAMP | Nullable |
| sent_at | TIMESTAMP | Nullable |
| failed_at | TIMESTAMP | Nullable |
| failure_message | VARCHAR(500) | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

The notification and FCM-token pair is unique. Per-installation tracking avoids
resending successful deliveries when another phone is retried.

## `device_controls`

| Column | Type | Rules |
|---|---|---|
| id | BIGINT | Primary key |
| device_id | BIGINT UNSIGNED | Indexed |
| user_id | BIGINT UNSIGNED | Indexed |
| device_control_status_id | BIGINT UNSIGNED | Indexed |
| control_type | VARCHAR(100) | Required |
| control_data | JSON | Nullable |
| requested_at | TIMESTAMP | Required |
| completed_at | TIMESTAMP | Nullable |
| failure_message | VARCHAR(500) | Nullable |
| created_at | TIMESTAMP | Required |
| updated_at | TIMESTAMP | Required |

Example control types:

- `irrigation.start`
- `irrigation.stop`
- `fan.start`
- `fan.stop`
- `roof.open`
- `roof.close`
- `camera.capture`

## Relationships

```text
user_statuses
  └── users
       ├── devices as owner
       ├── device_user_access
       └── device_controls

device_types
  └── devices

device_statuses
  └── devices

device_access_statuses
  └── device_user_access

device_control_statuses
  └── device_controls
```

## Required service-layer validation

Because there are no foreign-key constraints, services must validate every referenced record before insert or update.
