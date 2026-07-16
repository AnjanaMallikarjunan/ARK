# ARK — Adaptive Resilient Kommunication

> **"When the network disappeared, humanity became the network."**

ARK is a production-quality **Offline Human Coordination Platform** built with Flutter. Designed for post-disaster and grid-down scenarios, ARK enables survivors to discover each other, share resources, coordinate tasks, and synchronize critical information — all without any internet connection, cloud services, or mobile infrastructure.

---

## 📖 Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Architecture](#architecture)
4. [Folder Structure](#folder-structure)
5. [Design System](#design-system)
6. [Offline Synchronization](#offline-synchronization)
7. [Screens & Navigation](#screens--navigation)
8. [Installation](#installation)
9. [Dependencies](#dependencies)
10. [Environment Variables](#environment-variables)
11. [Build & Deployment](#build--deployment)
12. [Future Improvements](#future-improvements)
13. [Acknowledgements](#acknowledgements)

---

## Overview

ARK operates under a single constraint: **zero internet dependency**. Every feature — from survivor discovery to knowledge sharing — works entirely on-device and peer-to-peer. The app treats every smartphone as both a client and a relay node in a mesh network of humans.

**Core Principles:**
- 🔌 **Fully Offline** — No REST APIs, no cloud database, no Firebase, no SMS
- 📡 **Mesh Networking** — Bluetooth Low Energy + Wi-Fi Direct for device-to-device sync
- 🗄️ **Local-First Storage** — All data persisted on-device using Hive
- 🤝 **Trust-Based Coordination** — Reputation system incentivises verified, helpful actions
- 🗺️ **Offline Maps** — OpenStreetMap tiles cached locally via flutter_map

---

## Features

### 1. Survivor Registration
Multi-step onboarding form that collects and stores survivor identity locally.

| Field | Details |
|---|---|
| Name | Full display name |
| Blood Group | A+, A−, B+, B−, O+, O−, AB+, AB− |
| Age | Numeric input |
| Emergency Contact | Name + phone number |
| Profession | Doctor, Engineer, Electrician, Farmer, Mechanic, Cook, Teacher, Volunteer |
| Skills | Multi-select chip grid |

All data is stored in Hive and never leaves the device unless explicitly shared via QR or Bluetooth sync.

---

### 2. Dashboard — Command Center
The operational hub. Displays all critical information above the fold.

- **KPI Strip** — Nearby Survivors count, Active Alerts, Trust Score, Last Sync time
- **Emergency Action Cards** — Large tappable cards: Need Help, Offer Help, Community Board, Offline Map, Knowledge Library, Nearby Survivors, Resources, Emergency SOS
- **Nearby Survivors Strip** — Horizontal scroll of closest discovered devices with name, profession, distance, trust level, and battery
- **Metric Grid** — Open Tasks, Active Resources, Network Nodes, Posts Created
- **Sync Indicator** — Live BLE/Wi-Fi Direct sync status badge

---

### 3. Community Board
Users create **information cards** instead of chat messages. Cards are structured, categorised, and GPS-tagged.

**Categories:** Food Available · Water Source · Shelter · Medical Camp · Charging Station · Danger Zone · Bridge Broken · Need Blood · Need Doctor · Lost Person

Each card contains: Title, Description, GPS Coordinates, Timestamp, Priority (Low / Medium / High / Critical), Creator Callsign.

Cards automatically propagate to nearby devices during sync sessions.

---

### 4. Offline Map
Powered by `flutter_map` with OpenStreetMap tiles cached locally.

- **Pins by category:** Food 🍎 · Water 💧 · Shelter 🏠 · Danger ⚠️ · Medical ➕ · Charging ⚡
- Filter by resource category
- Search nearby resources within a configurable radius
- Tap pin to view full card details

---

### 5. Knowledge Library
A curated offline library of emergency guides, always available without internet.

**Categories:** First Aid · Water Purification · Fire Safety · Emergency Survival · Agriculture · Food Storage · Basic Medicine · Electrical Repair · CPR

- Documents stored as structured local data
- Share any guide via Bluetooth or QR code
- Full-text search across all guides

---

### 6. Nearby Survivors
Automatic peer discovery via BLE advertising and scanning.

Each discovered survivor card shows:
- Name & Callsign
- Distance (estimated via signal strength)
- Profession & Skills
- Trust Score & Level
- Battery Level
- Connection Status (Online / Nearby / Offline)

Send coordination requests directly to nearby survivors.

---

### 7. Tasks
Structured coordination requests replacing ad-hoc messaging.

**Task Types:** Need Blood Donor · Need Transport · Need Mechanic · Need Medicine · Need Volunteer · Need Food

**Lifecycle:** `Open` → `Accepted` → `Completed`

Completing a task increases the helper's Trust Score by +10.

---

### 8. Reputation System
A gamified trust framework that surfaces reliable survivors.

| Action | Points |
|---|---|
| Helpful task completed | +10 |
| Verified shelter reported | +5 |
| Verified medical help provided | +15 |
| False information reported | −20 |

**Trust Levels:**

| Level | Score Range | Colour |
|---|---|---|
| Bronze | 0 – 99 | 🟤 #CD7F32 |
| Silver | 100 – 299 | ⚪ #C0C0C0 |
| Gold | 300 – 599 | 🟡 #FFD700 |
| Guardian | 600 – 999 | 🔵 Electric Blue |
| Legend | 1000+ | 🟢 Neon Green |

---

### 9. QR Sharing
Generate and scan QR codes for any data object — completely offline.

**Shareable via QR:** Tasks · Shelter Locations · Map Pins · Knowledge Guides · Survivor Profiles

Uses `qr_flutter` for generation and `mobile_scanner` for scanning. No network required.

---

### 10. Emergency SOS
A full-screen, high-visibility emergency broadcast screen.

- **Pulsing Red Button** — Animated concentric rings, impossible to miss
- **Broadcast Payload** — Name, Blood Group, GPS Location, Emergency Type, Priority
- **Alert Feed** — Incoming SOS alerts from nearby devices displayed in real-time
- **Broadcast Config** — Select emergency type and priority before broadcasting

Nearby devices automatically receive SOS alerts via BLE advertisement packets.

---

### 11. Device Synchronisation
The core mesh networking layer. Triggered automatically when two ARK devices come within BLE range.

**Exchanged during sync:**
- Community Board posts
- Tasks (open and accepted)
- Knowledge Library entries
- Map pins and resource locations
- Survivor profiles
- Trust scores

**Conflict Resolution:** Last-write-wins based on `updatedAt` timestamp. Higher-priority items (SOS, Critical posts) are always propagated first.

---

### 12. Resources
Track physical survival resources by location.

**Categories:** Food · Water · Medicine · Fuel · Power

Each resource entry shows: Available Quantity, Expiry Date, Last Updated timestamp, Owner Callsign.

---

### 13. Profile
The survivor's identity card within the ARK network.

- Avatar with trust-level glow ring
- Callsign pill (e.g. ARK-0042) and Profession pill
- Blood Group, Age, Online Status
- 4-stat bento row: Completed Tasks · Resources Shared · People Helped · Posts Created
- Skills chip grid
- Trust/Reputation section with animated progress bar and level ladder
- Recent Activity feed

---

### 14. Settings
- Dark Mode toggle
- Language selection
- Export local database to file
- Import database from file
- Reset local database

---

## Architecture

ARK follows **Clean Architecture** principles with a clear separation between data, domain, and presentation layers.

```
┌─────────────────────────────────────────────────────┐
│                   PRESENTATION LAYER                │
│  Screens · Widgets · Riverpod UI State              │
├─────────────────────────────────────────────────────┤
│                    DOMAIN LAYER                     │
│  Models · Use Cases · Repository Interfaces         │
├─────────────────────────────────────────────────────┤
│                     DATA LAYER                      │
│  Hive Local DB · BLE Service · QR Service           │
│  flutter_map Tile Cache · SharedPreferences         │
└─────────────────────────────────────────────────────┘
```

### State Management
**Riverpod** is used throughout for reactive, testable state management. Each feature domain has its own provider file under `lib/providers/`.

### Navigation
**go_router** with `StatefulShellRoute.indexedStack` for bottom-nav tab persistence. Full-screen flows (Registration, Emergency SOS) are mounted outside the shell as standalone routes.

### Data Flow
```
User Action
    │
    ▼
Riverpod Provider (StateNotifier / AsyncNotifier)
    │
    ▼
Repository Interface (abstract)
    │
    ├── Hive Repository (local persistence)
    ├── BLE Repository (peer sync)
    └── QR Repository (encode / decode)
```

---

## Folder Structure

```
ark/
├── lib/
│   ├── main.dart                          # App entry point, GoRouter init
│   ├── core/
│   │   └── app_export.dart                # Barrel export for shared imports
│   ├── routes/
│   │   └── app_routes.dart                # GoRouter config, route constants
│   ├── theme/
│   │   └── app_theme.dart                 # Material 3 dark theme, brand colours
│   ├── models/                            # Data models (Hive TypeAdapters)
│   │   ├── survivor_model.dart
│   │   ├── community_post_model.dart
│   │   ├── task_model.dart
│   │   ├── resource_model.dart
│   │   └── knowledge_entry_model.dart
│   ├── services/                          # Business logic services
│   │   ├── hive_service.dart              # Hive box management
│   │   ├── ble_service.dart               # BLE advertising & scanning
│   │   ├── sync_service.dart              # Conflict-resolution sync logic
│   │   └── qr_service.dart                # QR encode / decode
│   ├── providers/                         # Riverpod providers
│   │   ├── survivor_provider.dart
│   │   ├── community_provider.dart
│   │   ├── task_provider.dart
│   │   ├── resource_provider.dart
│   │   └── sync_provider.dart
│   ├── database/
│   │   └── hive_boxes.dart                # Hive box names & adapter registration
│   ├── bluetooth/
│   │   ├── ble_advertiser.dart            # BLE peripheral role
│   │   └── ble_scanner.dart               # BLE central role
│   ├── qr/
│   │   ├── qr_generator_widget.dart
│   │   └── qr_scanner_widget.dart
│   ├── maps/
│   │   ├── offline_map_screen.dart
│   │   └── map_pin_widget.dart
│   ├── knowledge/
│   │   ├── knowledge_library_screen.dart
│   │   └── knowledge_detail_screen.dart
│   ├── utils/
│   │   ├── date_utils.dart
│   │   ├── distance_utils.dart
│   │   └── trust_utils.dart
│   ├── widgets/                           # Reusable UI components
│   │   ├── app_bar_widget.dart
│   │   ├── app_navigation.dart
│   │   ├── app_scaffold.dart
│   │   ├── custom_error_widget.dart
│   │   ├── custom_icon_widget.dart
│   │   ├── custom_image_widget.dart
│   │   ├── empty_state_widget.dart
│   │   ├── loading_skeleton_widget.dart
│   │   └── status_badge_widget.dart
│   └── presentation/
│       ├── survivor_registration_screen/
│       │   ├── survivor_registration_screen.dart
│       │   └── widgets/
│       │       ├── registration_hero_widget.dart
│       │       ├── registration_form_widget.dart
│       │       └── step_indicator_widget.dart
│       ├── dashboard_screen/
│       │   ├── dashboard_screen.dart
│       │   └── widgets/
│       │       ├── dashboard_action_cards_widget.dart
│       │       ├── dashboard_kpi_strip_widget.dart
│       │       ├── dashboard_metric_grid_widget.dart
│       │       ├── dashboard_nearby_strip_widget.dart
│       │       └── dashboard_sync_indicator_widget.dart
│       ├── emergency_sos_screen/
│       │   ├── emergency_sos_screen.dart
│       │   └── widgets/
│       │       ├── sos_pulse_button_widget.dart
│       │       ├── sos_broadcast_config_widget.dart
│       │       └── sos_alert_feed_widget.dart
│       └── profile_screen/
│           └── profile_screen.dart
├── assets/
│   └── images/
│       ├── img_app_logo.svg
│       ├── sad_face.svg
│       └── no-image.jpg
├── android/
├── ios/
├── pubspec.yaml
└── README.md
```

---

## Design System

ARK uses a **dark futuristic glassmorphism** design language built on Material 3.

### Colour Palette

| Token | Hex | Usage |
|---|---|---|
| `backgroundDark` | `#0B1220` | Scaffold background |
| `surfaceDark` | `#111927` | Card surfaces |
| `surfaceVariantDark` | `#1A2535` | Input fields, secondary surfaces |
| `electricBlue` | `#1E90FF` | Primary actions, links, Guardian level |
| `neonGreen` | `#AAFF00` | Accent, success, Legend level |
| `dangerRed` | `#FF3B5C` | SOS, danger zones, errors |
| `warningAmber` | `#F6AD3C` | Open tasks, warnings |
| `successGreen` | `#00D084` | Completed tasks, verified |
| `glassSurface` | `#0FFFFFFF` | Glassmorphism card fill |
| `glassBorder` | `#1AFFFFFF` | Glassmorphism card border |

### Typography
**Sora** (via `google_fonts`) — a geometric sans-serif that reads clearly on dark backgrounds.

| Style | Size | Weight |
|---|---|---|
| Display Large | 36sp | Bold 700 |
| Headline Large | 24sp | Bold 700 |
| Title Large | 16sp | Bold 700 |
| Body Large | 15sp | Regular 400 |
| Label Small | 11sp | Medium 500 |

### Glassmorphism Cards
```dart
BackdropFilter(
  filter: ImageFilter.blur(sigmaX: 12, sigmaY: 12),
  child: Container(
    decoration: BoxDecoration(
      color: AppTheme.glassSurface,          // 6% white
      borderRadius: BorderRadius.circular(18),
      border: Border.all(color: AppTheme.glassBorder, width: 1), // 10% white
    ),
  ),
)
```

---

## Offline Synchronisation

ARK's sync engine is the backbone of the mesh network. It operates in two phases:

### Phase 1 — Discovery
1. Each device continuously **advertises** a compact BLE beacon containing: Callsign, Trust Level, Battery %, Last Sync timestamp
2. The BLE scanner detects nearby ARK devices and adds them to the `NearbyDevicesProvider`
3. When signal strength exceeds the connection threshold (−70 dBm), a full sync session is initiated

### Phase 2 — Data Exchange
```
Device A                          Device B
   │                                  │
   │──── GATT Connect ───────────────▶│
   │                                  │
   │◀─── Exchange Manifest ──────────▶│
   │     (item IDs + timestamps)      │
   │                                  │
   │──── Send Delta (A→B) ───────────▶│
   │◀─── Send Delta (B→A) ────────────│
   │                                  │
   │──── GATT Disconnect ────────────▶│
```

### Conflict Resolution
- **Timestamp wins:** The record with the later `updatedAt` timestamp is kept
- **Priority override:** SOS alerts and Critical-priority posts always propagate regardless of timestamp
- **Trust score merge:** Scores are summed from unique verifiable events; duplicates are deduplicated by event ID

### Sync Scope
| Data Type | Sync Direction | Conflict Strategy |
|---|---|---|
| Community Posts | Bidirectional | Latest timestamp |
| Tasks | Bidirectional | Latest timestamp |
| Knowledge Entries | Bidirectional | Latest timestamp |
| Map Pins | Bidirectional | Latest timestamp |
| Survivor Profiles | Bidirectional | Latest timestamp |
| Trust Score Events | Bidirectional | Deduplicate by event ID |
| SOS Alerts | Broadcast (flood) | Always propagate |

---

## Screens & Navigation

```
/ (initial)
└── SurvivorRegistrationScreen        ← Standalone (no bottom nav)

/emergency-sos-screen
└── EmergencySosScreen                ← Standalone full-screen

StatefulShellRoute (bottom nav)
├── /dashboard-screen
│   └── DashboardScreen
└── /profile-screen
    └── ProfileScreen
```

### Route Transitions
- **Fade** (280ms, `easeOutCubic`) — Default for all routes
- **Slide + Fade** — Registration screen entry

---

## Installation

### Prerequisites

| Tool | Version |
|---|---|
| Flutter SDK | ≥ 3.10.0 |
| Dart SDK | ≥ 3.0.0 |
| Android Studio | Hedgehog or later |
| VS Code | Latest + Flutter extension |
| Android SDK | API 21+ (Android 5.0) |
| Xcode | 15+ (for iOS) |

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/your-org/ark.git
cd ark

# 2. Install Flutter dependencies
flutter pub get

# 3. Verify Flutter setup
flutter doctor

# 4. Run on a connected device or emulator
flutter run

# 5. Run on a specific platform
flutter run -d android
flutter run -d ios
```

### Android Permissions
Add the following to `android/app/src/main/AndroidManifest.xml`:

```xml
<!-- Bluetooth -->
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
<uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />

<!-- Location (required for BLE scanning on Android) -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

<!-- Wi-Fi Direct -->
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
```

### iOS Permissions
Add to `ios/Runner/Info.plist`:

```xml
<key>NSBluetoothAlwaysUsageDescription</key>
<string>ARK uses Bluetooth to discover and sync with nearby survivors.</string>
<key>NSBluetoothPeripheralUsageDescription</key>
<string>ARK uses Bluetooth to broadcast your survivor status.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>ARK uses your location to tag community posts and find nearby resources.</string>
```

---

## Dependencies

### Core

| Package | Version | Purpose |
|---|---|---|
| `flutter` | SDK | Core framework |
| `dart` | SDK ≥ 3.0 | Language runtime |

### UI & Design

| Package | Version | Purpose |
|---|---|---|
| `sizer` | ^2.0.15 | Responsive sizing (`.w`, `.h`, `.sp`) |
| `flutter_svg` | ^2.0.9 | SVG asset rendering |
| `google_fonts` | ^6.1.0 | Sora typeface |

### Navigation

| Package | Version | Purpose |
|---|---|---|
| `go_router` | ^14.6.1 | Declarative routing with shell routes |

### Storage

| Package | Version | Purpose |
|---|---|---|
| `shared_preferences` | ^2.2.2 | Key-value settings persistence |
| `hive` | ^2.2.3 | Fast local NoSQL database |
| `hive_flutter` | ^1.1.0 | Hive Flutter integration |

### Bluetooth & Networking

| Package | Version | Purpose |
|---|---|---|
| `flutter_blue_plus` | ^1.32.12 | BLE advertising, scanning, GATT |
| `nearby_connections` | ^4.1.0 | Wi-Fi Direct / Nearby API |

### Maps

| Package | Version | Purpose |
|---|---|---|
| `flutter_map` | ^6.1.0 | Offline OpenStreetMap rendering |
| `latlong2` | ^0.9.0 | Latitude/longitude data types |

### QR

| Package | Version | Purpose |
|---|---|---|
| `qr_flutter` | ^4.1.0 | QR code generation widget |
| `mobile_scanner` | ^5.2.3 | Camera-based QR scanning |

### Animations

| Package | Version | Purpose |
|---|---|---|
| `lottie` | ^3.1.2 | JSON-based vector animations |

### Utilities

| Package | Version | Purpose |
|---|---|---|
| `connectivity_plus` | ^6.1.4 | Network connectivity monitoring |
| `dio` | ^5.4.0 | HTTP client (future use) |
| `fluttertoast` | ^8.2.4 | Toast notifications |
| `fl_chart` | ^0.65.0 | Charts for resource/trust graphs |
| `cached_network_image` | ^3.3.1 | Image caching |

### Dev Dependencies

| Package | Version | Purpose |
|---|---|---|
| `flutter_lints` | ^5.0.0 | Lint rules |
| `hive_generator` | ^2.0.1 | Hive TypeAdapter code generation |
| `build_runner` | ^2.4.8 | Code generation runner |

---

## Environment Variables

ARK is designed to be fully offline. The `env.json` file at the project root is used only for optional cloud-connected features (future phases). No environment variables are required to run the core offline application.

```json
{
  "SUPABASE_URL": "",
  "SUPABASE_ANON_KEY": ""
}
```

---

## Build & Deployment

### Android APK (Debug)
```bash
flutter build apk --debug
# Output: build/app/outputs/flutter-apk/app-debug.apk
```

### Android APK (Release)
```bash
flutter build apk --release
# Output: build/app/outputs/flutter-apk/app-release.apk
```

### Android App Bundle (Play Store)
```bash
flutter build appbundle --release
# Output: build/app/outputs/bundle/release/app-release.aab
```

### iOS (Xcode Archive)
```bash
flutter build ios --release
# Then archive in Xcode for App Store / Ad Hoc distribution
```

### Hive Code Generation
If you add new Hive models, regenerate TypeAdapters:
```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

---

## Future Improvements

### Phase 2 — Enhanced Mesh Networking
- [ ] **Multi-hop relay** — Messages forwarded through intermediate ARK nodes to extend range beyond direct BLE reach
- [ ] **Store-and-forward** — Undelivered messages queued and forwarded when a relay node is encountered
- [ ] **Mesh topology visualisation** — Interactive graph showing the live relay network on the offline map

### Phase 3 — Advanced Features
- [ ] **Voice memos** — Short audio clips attached to community posts, shared via BLE
- [ ] **Photo attachments** — Compressed images (≤50 KB) attached to posts and tasks
- [ ] **Offline navigation** — Turn-by-turn routing using cached OSM data and `flutter_map`
- [ ] **Resource barcode scanning** — Scan product barcodes to log food/medicine resources
- [ ] **Medical triage forms** — Structured injury assessment forms for medical professionals

### Phase 4 — Resilience & Security
- [ ] **End-to-end encryption** — Curve25519 key exchange for private coordination requests
- [ ] **Tamper-evident trust scores** — Cryptographic signatures on trust score events
- [ ] **Sybil attack resistance** — Rate-limiting and peer vouching to prevent fake survivor spam
- [ ] **Data integrity checksums** — SHA-256 hashes on all synced records

### Phase 5 — Platform Expansion
- [ ] **Wear OS companion** — Wrist-based SOS trigger and nearby survivor count
- [ ] **Tablet layout** — Two-pane adaptive layout for larger screens
- [ ] **LoRa radio integration** — Ultra-long-range (10+ km) communication via LoRa hardware
- [ ] **Ham radio bridge** — Export/import data packets via amateur radio operators

### Developer Experience
- [ ] **Unit test suite** — Full coverage for sync conflict resolution and trust score logic
- [ ] **Widget test suite** — Golden tests for all glassmorphism card variants
- [ ] **Integration tests** — End-to-end BLE sync simulation with two emulated devices
- [ ] **CI/CD pipeline** — GitHub Actions for automated build, test, and APK artifact upload

---

## Acknowledgements

- Built with [Rocket.new](https://rocket.new) — AI-powered Flutter development platform
- Powered by [Flutter](https://flutter.dev) & [Dart](https://dart.dev)
- Maps by [OpenStreetMap](https://www.openstreetmap.org/) contributors
- Typography: [Sora](https://fonts.google.com/specimen/Sora) via Google Fonts
- Icons: Material Design 3 icon set

---

*ARK — Because when the network disappears, humanity becomes the network.*
