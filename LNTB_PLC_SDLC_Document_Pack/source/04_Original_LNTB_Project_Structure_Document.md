# LNTB — Smart Tomato Farming System
## Project Structure Document

**Team:** LNTB (Leading Next-Gen Tomato Business)
**System:** IoT + AI-enabled tomato farming with Flutter mobile app and Laravel backend

---

## 1. System Overview

Three layers working together:

| Layer | Role |
|---|---|
| **IoT (ESP32 + sensors)** | Collects soil moisture, temperature, humidity, light, and ripeness images. Executes pump/servo commands. |
| **Laravel (backend core)** | Central source of truth. Ingests sensor data, stores history, calculates cost, runs AI ripeness check (or receives result from ESP32-CAM), broadcasts live updates, sends push notifications. |
| **Flutter (mobile app)** | Farmer-facing app in Khmer. Displays dashboard, controls irrigation, shows ripeness status, cost tracking, and digital logbook. |

```
[ESP32 + Sensors] --HTTP POST--> [Laravel API] --MySQL--> [Database]
                                        |
                                        |--Reverb WebSocket--> [Flutter: live dashboard]
                                        |--FCM Push-----------> [Flutter: notifications]
                                        |
[Flutter] --HTTP (Sanctum token)------> [Laravel API] --queues command--> [ESP32 polls & executes]
```

---

## 2. Project Action Plan (Development Phases)

Mapped to your existing 5-step plan, expanded into actionable engineering tasks.

### Phase 1 — Research & Data Collection (May–early June)
- Confirm target moisture %, light lux, and ripeness color thresholds with farmers
- Collect tomato images across ripeness stages (unripe, semi-ripe, ripe) for AI training set
- Finalize sensor list and wiring plan
- Define Laravel data model based on what needs to be measured/stored

### Phase 2 — Backend Foundation (early–mid June)
- Set up Laravel project, Sanctum auth, Reverb broadcasting
- Build database schema (see Section 4)
- Build device ingestion endpoint + command queue endpoint
- Seed test data to unblock Flutter development in parallel

### Phase 3 — Hardware Assembly (mid June)
- Wire ESP32 + soil moisture + DHT22 + LDR + servo + relay + pump + fan
- Wire ESP32-CAM separately (own power/network considerations)
- Flash firmware: read sensors → POST to Laravel every N seconds
- Flash firmware: poll `/commands` endpoint → trigger relay/servo

### Phase 4 — Flutter App Development (June–July, parallel with Phase 3)
- Auth (Sanctum login)
- Dashboard (live sensor values + status colors)
- Irrigation control screen (manual trigger + auto status)
- Ripeness screen (latest detection results)
- Cost tracking screen (daily/monthly water + electricity cost)
- Digital logbook (searchable history by month/year)
- Notifications (FCM integration)
- Khmer localization throughout

### Phase 5 — Integration & Testing (July)
- Connect real ESP32 output to real Laravel endpoints (replace seed data)
- Validate AI ripeness accuracy against manual inspection
- Field test in actual net house — check WiFi range/reliability
- Tune moisture/ripeness thresholds based on real readings

### Phase 6 — Calibration & Final Presentation (August)
- Fix issues found in field testing
- Calculate actual cost savings (20–30% target)
- Prepare final report and demo

---

## 3. Features Needed (By Module)

### 3.1 Device & Sensor Management
- Register/pair ESP32 devices to a farm
- Receive and store sensor readings (moisture, temp, humidity, light)
- Device online/offline status (`last_seen_at`)

### 3.2 Irrigation Control
- Auto-trigger pump based on moisture threshold (logic can live in Laravel or ESP32 firmware — recommend Laravel owns the threshold *decision*, ESP32 just executes)
- Manual override from Flutter app
- Log every irrigation event: trigger source (auto/manual), duration, water volume (m³)

### 3.3 Ripeness Detection
- Receive image/result from ESP32-CAM
- Store per-fruit or per-batch ripeness status
- Flag "ready to harvest" (e.g., 80–90% red) and push notification to farmer

### 3.4 Cost Tracking (Real-time)
- Record water (m³) and electricity (kWh) usage per irrigation/event
- Daily rollup job (scheduled) → `usage_records`
- Convert usage to cost (configurable $/m³, $/kWh rates)
- Monthly comparison view

### 3.5 Digital Logbook
- Searchable/filterable history (by date range, month, year)
- Replaces paper logbook — must be easy to browse on mobile

### 3.6 Notifications
- Ripeness ready alert
- Moisture too low/high alert
- Device offline alert (nice-to-have)

### 3.7 Farmer-Friendly UI Requirements
- 100% Khmer language
- Color status system: green = good, red = needs attention
- Clear icons, minimal text-heavy screens
- Works acceptably on low-end Android phones (common in target market)

### 3.8 Auth & Multi-Farm Support
- Farmer login (Sanctum token)
- Optionally support multiple net houses per farmer account

---

## 4. Database Design

### `farms`
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| user_id | FK → users | owner |
| name | string | |
| location | string | nullable |
| created_at / updated_at | timestamps | |

### `devices`
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| farm_id | FK → farms | |
| esp32_id | string, unique | hardware identifier |
| type | enum | `sensor_node`, `camera_node` |
| last_seen_at | timestamp | nullable |
| created_at / updated_at | timestamps | |

### `sensor_readings`
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| device_id | FK → devices | |
| soil_moisture | decimal | % |
| temperature | decimal | °C |
| humidity | decimal | % |
| light_level | decimal | lux or raw LDR value |
| recorded_at | timestamp | |

*(Index on `device_id, recorded_at` — this table grows fast.)*

### `irrigation_events`
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| device_id | FK → devices | |
| triggered_by | enum | `auto`, `manual` |
| triggered_by_user_id | FK → users | nullable, only if manual |
| duration_seconds | integer | |
| water_m3 | decimal | |
| power_kwh | decimal | |
| started_at | timestamp | |
| ended_at | timestamp | nullable |

### `ripeness_results`
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| device_id | FK → devices | camera node |
| image_url | string | |
| confidence | decimal | AI confidence % |
| status | enum | `unripe`, `semi_ripe`, `ready_to_harvest` |
| detected_at | timestamp | |

### `usage_records` (daily rollup)
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| farm_id | FK → farms | |
| date | date | |
| water_m3 | decimal | sum for the day |
| power_kwh | decimal | sum for the day |
| cost_usd | decimal | calculated |
| created_at / updated_at | timestamps | |

### `pending_commands` (ESP32 command queue)
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| device_id | FK → devices | |
| command | enum | `pump_on`, `pump_off`, `servo_open`, `servo_close` |
| status | enum | `pending`, `acknowledged`, `executed` |
| created_at / updated_at | timestamps | |

### `users`
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| name | string | |
| phone or email | string | Khmer farmers may prefer phone login |
| password | hashed | |
| created_at / updated_at | timestamps | |

**Relationships summary:**
`users` 1—* `farms` 1—* `devices` 1—* `sensor_readings` / `irrigation_events` / `ripeness_results`
`farms` 1—* `usage_records` (daily rollups, farm-level not device-level)

---

## 5. API Endpoint Map

| Method | Endpoint | Consumer | Purpose |
|---|---|---|---|
| POST | `/api/devices/{esp32_id}/readings` | ESP32 | Push sensor data |
| GET | `/api/devices/{esp32_id}/commands` | ESP32 | Poll for pending command |
| POST | `/api/devices/{esp32_id}/commands/{id}/ack` | ESP32 | Confirm command executed |
| POST | `/api/devices/{esp32_id}/ripeness` | ESP32-CAM | Push ripeness result |
| POST | `/api/login` | Flutter | Sanctum auth |
| GET | `/api/dashboard` | Flutter | Current status summary |
| POST | `/api/irrigation/trigger` | Flutter | Manual pump on/off |
| GET | `/api/irrigation/history` | Flutter | Irrigation log |
| GET | `/api/ripeness/latest` | Flutter | Latest ripeness results |
| GET | `/api/cost?period=month` | Flutter | Cost summary |
| GET | `/api/logbook?month=&year=` | Flutter | Paginated history |

---

## 6. Tech Stack Summary

| Layer | Choice |
|---|---|
| Mobile | Flutter, Riverpod, GoRouter |
| Backend | Laravel, Sanctum (auth), Reverb (WebSocket broadcast) |
| Database | MySQL |
| Realtime | Laravel Reverb + `laravel_echo`/`pusher_client` on Flutter |
| Push notifications | Firebase Cloud Messaging (FCM) — used only for push, not as the data store |
| Hardware | ESP32, ESP32-CAM, DHT22, Soil Moisture Sensor, LDR, Servo, 2-Channel Relay, Water Pump, Fan |

---

## 7. Open Decisions To Confirm

1. **Where does AI ripeness inference run?** On-device on ESP32-CAM (limited compute) vs. ESP32-CAM sends raw image to Laravel, which calls an external inference service. This affects both firmware and backend design — worth locking down before Phase 3.
2. **Threshold ownership** — should moisture-triggered auto-irrigation logic live in Laravel (central, easy to tune) or in ESP32 firmware (works even if backend is down)? Recommend: firmware has a hardcoded safe fallback threshold, Laravel can push an updated threshold value to override it.
3. **Multi-farm vs single-farm MVP** — simplifies both DB design and UI if you scope to one farm per account for now.
