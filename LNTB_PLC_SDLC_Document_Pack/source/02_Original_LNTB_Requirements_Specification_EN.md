# LNTB — Smart Tomato Farming System
## Requirements Specification Document

**Team:** LNTB (Leading Next-Gen Tomato Business)
**Stack:** Laravel (backend) · Flutter (mobile) · Arduino/C++ on ESP32 (firmware)

---

## 1. Introduction

### 1.1 Purpose
This document defines what each part of the LNTB system must do and the conditions it must operate under. It covers functional requirements, non-functional requirements, hardware requirements, software requirements, and data requirements across all three components: IoT firmware, Laravel backend, and Flutter mobile app.

### 1.2 Scope
The system helps Cambodian net-house tomato farmers automate irrigation, track water/electricity cost in real time, and detect fruit ripeness using a sensor + AI camera setup, all managed through a Khmer-language mobile app.

### 1.3 Definitions
| Term | Meaning |
|---|---|
| Net house | Netted greenhouse structure for tomato cultivation |
| Sensor node | ESP32 board + moisture/temp/light sensors + relay/pump |
| Camera node | ESP32-CAM for ripeness detection |
| Farmer | Primary end user of the mobile app |

---

## 2. Overall System Requirements

- The system shall consist of three cooperating components: firmware (Arduino/ESP32), backend (Laravel), and mobile app (Flutter).
- The system shall function with intermittent/weak WiFi typical of rural net-house locations.
- The system shall operate on a total hardware budget of approximately $103.50 per net house (per original BOM).
- The system shall support at least one net house per farmer account at MVP stage.

---

## 3. Functional Requirements

### 3.1 Firmware (Arduino / ESP32) — FR-HW

| ID | Requirement |
|---|---|
| FR-HW-01 | The sensor node shall read soil moisture at a configurable interval (default: every 60s). |
| FR-HW-02 | The sensor node shall read temperature and humidity via DHT22. |
| FR-HW-03 | The sensor node shall read ambient light level via LDR. |
| FR-HW-04 | The sensor node shall send readings to the Laravel API as JSON over HTTPS. |
| FR-HW-05 | The sensor node shall poll the Laravel command endpoint at a configurable interval (default: every 10s) for pending pump/servo commands. |
| FR-HW-06 | The sensor node shall trigger the relay (water pump) when it receives a `pump_on` command, and stop on `pump_off`. |
| FR-HW-07 | The sensor node shall trigger the servo (net roof) when it receives `servo_open` / `servo_close`. |
| FR-HW-08 | The sensor node shall acknowledge executed commands back to the backend. |
| FR-HW-09 | The sensor node shall fall back to a locally-stored safe moisture threshold and auto-irrigate if the backend is unreachable for longer than a configurable timeout. |
| FR-HW-10 | The camera node shall capture an image of tomato fruit at a configurable interval or on trigger. |
| FR-HW-11 | The camera node shall send the captured image (or locally-inferred ripeness result, depending on final inference-location decision) to the Laravel API. |
| FR-HW-12 | The firmware shall store WiFi credentials and API endpoint in a separate config file, not hardcoded inline. |
| FR-HW-13 | The firmware shall reconnect automatically after WiFi drop without requiring a manual reset. |

### 3.2 Backend (Laravel) — FR-BE

| ID | Requirement |
|---|---|
| FR-BE-01 | The backend shall expose an authenticated endpoint for ESP32 devices to submit sensor readings. |
| FR-BE-02 | The backend shall expose an endpoint for ESP32 to poll pending commands. |
| FR-BE-03 | The backend shall expose an endpoint for ESP32 to acknowledge command execution. |
| FR-BE-04 | The backend shall expose an endpoint to receive ripeness detection results/images. |
| FR-BE-05 | The backend shall calculate real-time water (m³) and electricity (kWh) usage per irrigation event. |
| FR-BE-06 | The backend shall run a scheduled job to roll up daily usage into `usage_records` with calculated USD cost. |
| FR-BE-07 | The backend shall broadcast live sensor updates to the Flutter app via WebSocket (Laravel Reverb) per farm channel. |
| FR-BE-08 | The backend shall send push notifications (FCM) when a fruit is marked ready-to-harvest. |
| FR-BE-09 | The backend shall send push notifications when soil moisture crosses a critical threshold. |
| FR-BE-10 | The backend shall provide Sanctum token-based authentication for the Flutter app. |
| FR-BE-11 | The backend shall provide paginated, filterable logbook query endpoints (by month/year/date range). |
| FR-BE-12 | The backend shall allow a farmer to manually trigger/stop irrigation via API, subject to authentication. |
| FR-BE-13 | The backend shall record who/what triggered each irrigation event (`auto` vs `manual`, and by which user if manual). |
| FR-BE-14 | The backend shall support configurable per-farm cost rates ($/m³, $/kWh) used in cost calculation. |

### 3.3 Mobile App (Flutter) — FR-APP

| ID | Requirement |
|---|---|
| FR-APP-01 | The app shall display a login screen and authenticate against the Laravel Sanctum endpoint. |
| FR-APP-02 | The app shall display a real-time dashboard showing soil moisture, temperature, humidity, and light level. |
| FR-APP-03 | The app shall visually indicate status using color coding (green = normal, red = needs attention). |
| FR-APP-04 | The app shall allow the farmer to manually trigger or stop irrigation. |
| FR-APP-05 | The app shall display current auto-irrigation status (on/off, last triggered). |
| FR-APP-06 | The app shall display the latest ripeness detection results with images and confidence level. |
| FR-APP-07 | The app shall display real-time and historical water/electricity cost. |
| FR-APP-08 | The app shall provide a digital logbook screen, filterable by month and year. |
| FR-APP-09 | The app shall receive and display push notifications for ripeness-ready and threshold alerts. |
| FR-APP-10 | The app UI shall be rendered 100% in Khmer language. |
| FR-APP-11 | The app shall function acceptably on low-end/budget Android devices. |
| FR-APP-12 | The app shall handle intermittent connectivity gracefully (e.g., cached last-known dashboard state, retry logic). |

---

## 4. Non-Functional Requirements

| ID | Category | Requirement |
|---|---|---|
| NFR-01 | Usability | A farmer with no technical background shall be able to read the dashboard status within 5 seconds using color/icon cues alone. |
| NFR-02 | Performance | Sensor readings shall appear on the dashboard within 5 seconds of being sent by the ESP32 (via WebSocket broadcast). |
| NFR-03 | Reliability | The system shall continue basic auto-irrigation even if the Laravel backend is temporarily unreachable (firmware fallback threshold). |
| NFR-04 | Availability | The Laravel backend should target uptime suitable for a single-farm pilot (formal SLA not required at MVP). |
| NFR-05 | Security | All API traffic shall use HTTPS. Device endpoints shall require an API key/device token; user endpoints shall require Sanctum auth. |
| NFR-06 | Scalability | The database schema shall support multiple farms/devices per user without redesign (even if MVP UI restricts to one farm). |
| NFR-07 | Localization | All user-facing text in the Flutter app shall be sourced from Khmer `.arb` files, not hardcoded strings. |
| NFR-08 | Cost | Total hardware bill-of-materials per net house shall not exceed approximately $105 USD. |
| NFR-09 | Maintainability | Firmware, backend, and app shall each be structured in clearly separated modules (see Project Structure Document) to allow independent updates. |
| NFR-10 | Data retention | Sensor readings and logs shall be retrievable for at least one full growing season for cost comparison purposes. |

---

## 5. Hardware Requirements

| Component | Purpose | Qty |
|---|---|---|
| ESP32 Microcontroller Board | Main sensor node controller | 1 |
| ESP32-CAM | Ripeness detection camera node | 1 |
| Soil Moisture Sensor | Soil moisture reading | 2 |
| DHT22 | Temperature & humidity | 1 |
| LDR Sensor + Servo Motor | Light reading + auto net roof control | 1 set |
| 2-Channel Relay + Water Pump + Fan | Irrigation + ventilation control | 1 set |
| Wiring / waterproof enclosure / circuit board | Assembly | — |

**Environmental requirement:** all outdoor/net-house hardware (sensor node, wiring, camera) must be housed in a waterproof enclosure suitable for humid greenhouse conditions.

---

## 6. Software Requirements

### 6.1 Firmware
| Item | Choice |
|---|---|
| Language | C++ (Arduino framework) |
| Toolchain | PlatformIO (recommended) or Arduino IDE |
| Key libraries | `DHT sensor library`, `ESP32Servo`, `HTTPClient`, `ArduinoJson`, `esp32-camera` |

### 6.2 Backend
| Item | Choice |
|---|---|
| Framework | Laravel (latest LTS-compatible version) |
| Auth | Laravel Sanctum |
| Realtime | Laravel Reverb |
| Database | MySQL |
| Notifications | Firebase Cloud Messaging (via Laravel notification channel) |
| Task scheduling | Laravel Scheduler (for daily usage rollup job) |

### 6.3 Mobile App
| Item | Choice |
|---|---|
| Framework | Flutter |
| State management | Riverpod |
| Routing | GoRouter |
| Realtime client | `laravel_echo` + `pusher_client` (Reverb-compatible) |
| HTTP client | Dio |
| Push notifications | `firebase_messaging` |
| Localization | Flutter `intl` / `.arb` files (Khmer) |

---

## 7. User Requirements

- Farmers using the app may have limited literacy with technology — UI must rely on color/icons over dense text.
- Farmers need to check farm status remotely (not physically present at net house at all times).
- Farmers need historical cost data to compare month-to-month or season-to-season.
- Farmers need confidence that manual override is always possible, even when auto-mode is active.

---

## 8. Data Requirements

- Sensor readings must be timestamped and traceable to a specific device and farm.
- Irrigation events must record enough detail (duration, volume, trigger source) to support accurate cost calculation.
- Ripeness results must retain the source image for farmer verification (AI is a decision aid, not fully autonomous).
- Daily usage rollups must persist independently of raw readings so historical cost comparisons remain fast even as raw data grows.

*(Full schema is defined in the Project Structure Document — Section 4: Database Design.)*

---

## 9. Interface Requirements

- Firmware ↔ Backend: HTTPS/JSON over REST (defined in Project Structure Document — Section 5: API Endpoint Map).
- Backend ↔ Mobile App: HTTPS/JSON REST for requests, WebSocket (Reverb) for live push, FCM for notifications.
- No direct communication between firmware and Flutter app — Laravel is the sole intermediary, by design.

---

## 10. Constraints & Assumptions

**Constraints**
- Budget ceiling (~$105/net house) restricts sensor precision and camera quality.
- Rural WiFi reliability cannot be assumed to be constant.
- Team is early-stage in Laravel/Flutter/Arduino integration experience — architecture favors simplicity over advanced patterns.

**Assumptions**
- Each net house has at least intermittent WiFi coverage.
- Farmers have access to a smartphone capable of running a modern Flutter app.
- One ESP32-CAM per net house is sufficient for ripeness sampling (not full 100% fruit coverage).

---

## 11. Open Items Requiring Decision

1. AI ripeness inference location: on ESP32-CAM vs. Laravel-side/external inference service (affects FR-HW-11 and FR-BE-04 precisely).
2. Whether moisture threshold values are firmware-hardcoded-only or backend-configurable and pushed to firmware.
3. Confirm single-farm vs multi-farm MVP scope (affects FR-APP screens and auth design).
