# IIIT NR Student App — Full Project Overview

This repository contains a cross-platform student application for IIIT Naya Raipur, built with a Flutter frontend and a Node.js/Express backend. It delivers attendance, gatepass, complaints, contacts, and buy/sell features, along with secure, biometric-based authentication. The app targets Android, iOS, web, Windows, Linux, and macOS using Flutter’s multi-platform support, with backend services designed for REST APIs and real-time updates.

## Contents
- Overview
- Architecture
- Frontend (Flutter)
- Backend (Node.js/Express)
- Key Features
- Biometric Flow & Security
- Attendance Module
- Leave Requests & Date Range UX
- Development Environment & Setup
- Running Locally
- Testing & Linting
- Project Structure
- Configuration & Secrets
- CI/CD & Deployment (Recommended)
- Troubleshooting & Common Issues
- Roadmap & Ideas

---

## Overview
The IIIT NR Student App provides a unified interface for campus-related tasks:
- Attendance tracking and session management
- Gatepass requests
- Complaint submission and tracking
- Buy/Sell marketplace
- Contacts directory
- Found/Lost items
- Real-time updates (notifications, approvals)
- Biometric device registration and approval for secure attendance

UX is tuned for a consistent dark theme with a purple accent (`#B39DDB`) and a dark blue app bar (`#0f1d3a`). The app prioritizes clarity, accessibility, and cross-platform predictability.

---

## Architecture
The project is a monorepo with two primary components:
- `frontend/`: Flutter application (multi-platform). Implements navigation, screens, repositories, services, and platform integrations.
- `backend/`: Node.js (Express) API server with modules for auth, biometrics, attendance, leave, sessions, and admin operations.

Data Flow:
- Frontend communicates with backend via REST (HTTP) and socket-based real-time channels.
- Authentication uses session/cookies or tokens (as provided by `ApiClient`).
- Biometric device enrollment uses platform channels (local_auth, native Android/iOS APIs) and backend validation/approval.

---

## Frontend (Flutter)
Primary technologies:
- Flutter SDK (Dart)
- Routing: `go_router`
- Networking: `http` and `dio`
- Security: `flutter_secure_storage`
- Device: `camera`, `mobile_scanner`, `local_auth`
- ML integrations: Google ML Kit (face detection and other modules)
- Utilities: `intl`, `cookie_jar`, `socket_io_client`, `file_picker`, `table_calendar`

Key Folders:
- `lib/main.dart`: App entry point and router setup
- `lib/screens/`: UI screens for student, professor, and admin flows
- `lib/services/`: Device services (e.g., biometrics)
- `lib/api/`: API client, realtime socket setup
- Platform folders: `android/`, `ios/`, `web/`, `windows/`, `linux/`, `macos/`

UI Theme:
- Dark mode as default (`#121212` background)
- App bar dark blue (`#0f1d3a`)
- Accent purple (`#B39DDB`)
- Consistent input decoration with visible labels, white hint text, and purple focus borders.

---

## Backend (Node.js/Express)
Primary technologies:
- Node.js
- Express framework
- MongoDB (via `db.js` config)
- Modules: `auth`, `biometrics`, `attendance`, `leave`, `sessions`, `admin`
- Real-time: `realtime/index.js` (Socket.io or similar)
- Scripts: `scripts/create_indexes.js`, `scripts/test_usedtoken_race.js`

Key Files:
- `src/app.js`: Server initialization, middleware, route mounting
- `src/router.js`: Route composition
- `src/models/`: Mongoose models for core entities (user, session, attendance, leave, biometricKey, etc.)
- `render.yaml`: Deployment blueprint (Render.com or similar)
- `test/`: Integration tests (e.g., biometrics)

---

## Key Features
- Student Dashboard with tiles for all modules
- Attendance flow with biometric verification
- Leave request with date range picker and manual entry support
- Gatepass request flow
- Complaint submission and tracking
- Buy/Sell listings
- Contacts directory
- Found/Lost item reporting
- Admin approval workflows for biometric keys

---

## Biometric Flow & Security
1. **Status Check**: Frontend calls `ApiClient.I.biometricCheck()` to determine device status (`none`, `pending`, `approved`, `revoked`).
2. **Registration**: If `none` or `revoked`, guide the user to enroll device biometrics and generate a public key PEM via `BiometricService.generateAndGetPublicKeyPem()`.
3. **Challenge Signing**: Test signing a random challenge using platform biometrics (`BiometricService.signChallenge(...)`).
4. **Backend Registration**: Submit public key to backend (`registerBiometricKey`) for admin approval.
5. **Approval & Usage**: Once approved, attendance actions rely on biometric signatures to prevent impersonation.

Security Considerations:
- Keys stored securely on device; signing requires biometric prompt.
- Backend validates signatures against the registered public key.
- Admins can revoke keys; client handles re-registration gracefully.

---

## Attendance Module
- Professor creates sessions; students mark attendance.
- Students require an approved biometric key; otherwise flows prompt registration or show status.
- Real-time updates propagate session status changes.

---

## Leave Requests & Date Range UX
- Leave page uses `showDateRangePicker` with a dark theme and accented purple.
- All day labels are forced white for readability; selected text honors `onPrimary` (white).
- Manual date entry is supported via two fields (Start/End):
  - Auto-insert slashes (`_DateSlashFormatter`) to enforce `dd/MM/yyyy`.
  - Inline validation for format, bounds (today → +30 days), ordering, and length (≤30 days inclusive).
  - Picker selection syncs with manual fields and vice versa.
- Submission validates final range and reason, then posts via `ApiClient.I.requestLeaveRange(...)`.

---

## Development Environment & Setup
Requirements:
- Flutter SDK (3.6.x compatible)
- Dart SDK (ships with Flutter)
- Android SDK & emulator (for Android dev)
- Xcode (for iOS dev, macOS)
- Node.js (LTS)
- MongoDB instance (local or hosted)

Install Frontend Dependencies:
```powershell
cd frontend
flutter pub get
```

Install Backend Dependencies:
```powershell
cd backend
npm install
```

Environment Variables:
- Frontend uses `flutter_dotenv` for environment (e.g., API base URL).
- Backend uses environment variables (e.g., `MONGODB_URI`, `SESSION_SECRET`).

---

## Running Locally
Backend:
```powershell
cd backend
npm start
```
(Ensure `db.js` points to a reachable MongoDB)

Frontend:
```powershell
cd frontend
flutter run
```
To launch an emulator:
```powershell
flutter emulators --launch Pixel_5_API_33
```
To run for web:
```powershell
flutter run -d chrome
```

---

## Testing & Linting
Frontend Analyze:
```powershell
cd frontend
flutter analyze
```

Backend Tests:
```powershell
cd backend
npm test
```

Linting Notes:
- Some warnings (deprecated `withOpacity`, naming style, unused imports) are known and non-blocking.
- Prefer `Color.withValues()` over deprecated `withOpacity` in modern Flutter.
- Follow `lower_case_with_underscores` for file names when convenient.

---

## Project Structure
```
backend/
  package.json
  src/
    app.js
    router.js
    config/db.js
    models/*.js
    modules/*
  realtime/index.js
  scripts/*.js
  test/*.spec.js

frontend/
  pubspec.yaml
  lib/
    main.dart
    api/*
    navigation/*
    repositories/*
    screens/*
    services/*
    utils/*
  android/ ios/ web/ windows/ linux/ macos/
  assets/
    images/
    fonts/
```

---

## Configuration & Secrets
Frontend:
- `.env` (loaded by `flutter_dotenv`) for `API_BASE_URL`, feature flags, etc.
- Secure storage for tokens/secrets via `flutter_secure_storage`.

Backend:
- Environment variables stored in deployment platform (Render/Azure/etc.).
- MongoDB connection string kept secret; never committed.
- Consider `dotenv` for local dev.

---

## CI/CD & Deployment (Recommended)
- Use GitHub Actions for:
  - Frontend: `flutter analyze`, `flutter build apk`, web builds
  - Backend: `npm test`, deploy to Render/Azure
- Backend `render.yaml` provides Render deploy configuration.
- Containerization: add Dockerfiles for backend and optional frontend web hosting.

---

## Troubleshooting & Common Issues
- Date picker text unreadable → Ensure `onPrimary` and `dayForegroundColor` set to white; apply `textTheme.apply(...)`.
- Manual date entry not parsing → Confirm `dd/MM/yyyy` format; the formatter inserts slashes automatically.
- Build fails due to deprecated APIs → Replace `withOpacity` with `withValues()`.
- Real-time updates not received → Check socket connection (`api/realtime.dart`) and backend `realtime/index.js`.
- Biometric flows not prompting → Verify device supports biometrics and enrollment via OS settings.

---

## Roadmap & Ideas
- Add comprehensive unit/integration tests for Flutter screens and services.
- Introduce state management (e.g., Riverpod/Bloc) for clearer separation.
- Improve admin dashboards and analytics.
- Add accessibility passes (text scaling, high contrast options).
- Internationalization with `intl` and language packs.
- Offline caching for core modules.

---

## Contributing
- Open issues and PRs are welcome.
- Please avoid committing secrets or credentials.
- Follow the existing dark theme and coding style.

---

## License
Internal project. If licensing is needed, add a suitable license file.
