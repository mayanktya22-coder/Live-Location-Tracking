# HIT-Tracker: 3D Indoor Tactical Tracking Dashboard

[![Network](https://img.shields.io/badge/Network-100%25%20Offline-blue)]()
[![Stack](https://img.shields.io/badge/Stack-Node%20%7C%20React%20%7C%20Three.js%20%7C%20SQLite-indigo)]()
[![Tests](https://img.shields.io/badge/Tests-21%20Passed-emerald)]()

**HIT-Tracker** is an offline-first live 3D indoor tracking dashboard designed for a Task Force Commander at an Integrated Command Post (ICP). It tracks small combat units (**House Intervention Teams, HITs**) operating inside multi-floor structures and subterranean environments, rendering their positions on a 3D building model in real time.

---

## Visual Overview

<!-- Screenshot / GIF Placeholder -->
```text
+-----------------------------------------------------------------------------------------------+
|  HIT-TRACKER [ICP 3D]    [ONLINE]            Sector 4 Compound          [TF Commander]        |
|-----------------------------------------------------------------------------------------------|
|  [ELEVATION] |                                                     | [HIT COMBAT ROSTER]      |
|  [ALL FLOORS]|                  3D TACTICAL VIEWPORT               |   HIT-1 "Alpha"          |
|  [L5]        |                                                     |     ALPHA-1 (L1, 91%)    |
|  [L4]        |            [Floor L2]                               |     ALPHA-2 (G, 96%)     |
|  [L3]        |           /          \                              |     ALPHA-3 (G, 92%)     |
|  [L2]   [2]  |      (o) ALPHA-1    (o) BRAVO-2                     |     ALPHA-4 (G, 21%)     |
|  [L1]   [4]  |           \          /                              |   HIT-2 "Bravo"          |
|  [G]    [4]  |            [Floor G]                                |   HIT-3 "Charlie"        |
|  [B1]   [4]  |                                                     |   HIT-4 "Delta"          |
|  [B2]   [2]  |                                                     |                          |
|--------------+-----------------------------------------------------+--------------------------|
|  [PLAY] [||]  00:14:32 =========O============================ 00:30:00   SPEED: [1x] [2x] [5x] |
+-----------------------------------------------------------------------------------------------+
```
*(Drop UI recording / walkthrough GIF at `docs/images/hit-tracker-demo.gif`)*

---

## Key Capabilities

- **Barometric Floor Estimation**: Translates measured pressure into relative altitude against entry-point reference pressure ($P_0$) using the hypsometric formula. Employs a 1D Kalman filter and hysteresis state machine to eliminate floor bouncing.
- **3D Building Visualization**: Renders GLB/GLTF and OBJ building models with OrbitControls, tactical camera presets, and coordinate alignment gizmo.
- **Floor Isolation & Slicing**:
  - **Ghost Mode**: Non-selected floors are rendered with holographic low opacity.
  - **Cutaway / Slice Mode**: Slices upper levels to reveal interior room clearing operations.
  - **Isolated Mode**: Shows only the active floor and units currently on it.
- **3D Movement Trails**: Color-coded tactical path trails tracing each unit's patrol history.
- **Tactical Alerts Engine**:
  - Critical audio-visual alarm when a soldier has not transmitted for $> 10\text{ seconds}$ ("SIGNAL LOST").
  - Warning alarm for low battery ($< 20\%$).
  - Informational floor transition announcements.
- **Model Alignment Calibration Tool**: Allows the Commander to adjust model translation ($X, Y, Z$), rotation, and scale to calibrate against tracker coordinates and save to SQLite.
- **Historical Mission Replay**: Scrub through past operations with play/pause, time scrubber, and speed multipliers ($1\times, 2\times, 5\times, 10\times$).
- **Offline-First Security**: Fully self-contained local network operation (zero runtime CDNs, no Google Maps, no cloud services), JWT authentication with bcrypt hashing, and Commander vs Viewer roles.

---

## Project Structure

```text
hit-tracker/
├── shared/       # Shared Zod schemas, types, barometric estimation & unit tests
├── server/       # Node.js Express server, SQLite database, dual WebSocket gateway
│   ├── samples/  # Committed sample 3D building models (.glb, .obj)
│   └── uploads/  # Runtime uploaded models (gitignored)
├── client/       # React + Vite + Three.js (@react-three/fiber) + Tailwind CSS
├── simulator/    # Realistic 16-soldier multi-team combat simulator
├── docs/         # Architecture, hardware protocol, and deployment specifications
└── README.md
```

---

## Quickstart Guide

### Prerequisites
- **Node.js**: `v20.0.0` or later (tested on Node v20/v22/v25) & `npm`
- **Python 3**: Only required if running the 3D sample model procedural generators (`server/generate-sample-*.py`)

### 1. Installation
```bash
# Clone repository
git clone <repo-url> hit-tracker
cd hit-tracker

# Install dependencies across all workspaces
npm install

# (Optional) For clean production installs matching lockfile:
# npm ci
```

### 2. Environment Setup
```bash
cp .env.example .env
```
*Configure `JWT_SECRET` and initial passwords in `.env` before running in production.*

### 3. Build & Run Tests
```bash
# Build shared library and run unit tests (21 Vitest tests)
npm test

# Typecheck all workspaces (shared, server, client, simulator)
npm run typecheck

# Build all workspaces
npm run build
```

### 4. Launch Application (Development Mode)
Run backend server, frontend dev server, and combat simulator concurrently:
```bash
npm start
```
*The simulator automatically waits for the backend API health check before streaming telemetry.*

Open your browser to:
- **Command Dashboard**: [http://localhost:5173](http://localhost:5173)

### 5. Production Single-Port Deployment
To build all assets and serve the single-page application directly from the Node.js server on port 3001:
```bash
npm run start:prod
```
Access the application at [http://localhost:3001](http://localhost:3001).

---

## Access Roles & Authentication

User accounts are initialized during first-time database setup from environment variables:

| Role | Username | Permissions |
|---|---|---|
| **TF Commander** | `commander` | Full access: 3D model upload, model alignment calibration, building specs, alert acknowledgment |
| **Intel Analyst** | `viewer` | Read-only command post monitor: 3D telemetry, floor slicing, mission replay |

> Passwords are configured via `COMMANDER_PASSWORD` and `VIEWER_PASSWORD` in `.env`. Demo buttons on the login modal provide one-click credential prefill in development.

---

## Sample 3D Building Models Included

The repository includes pre-generated procedural tactical models in `server/samples/`:
- `sample-building.glb` (**8 levels**: Basements B2, B1, Ground G, and Floors 1, 2, 3, 4, 5 with stairwells, rooms, columns, and structural walls)
- `sample-building.obj` (Matching Wavefront OBJ tactical compound)

To regenerate sample models:
```bash
npm run generate:models
```

---

## Limitations & Assumptions

1. **Hardware Assumptions**: Tracker packs are assumed to integrate a calibrated barometric sensor ($\pm 0.05\text{ hPa}$ resolution) and a 2D indoor positioning source (such as Ultra-Wideband, SLAM-assisted PDR, or local radio ranging).
2. **Barometric Weather Drift**: Atmospheric pressure naturally drifts over hours due to weather. Absolute elevation readings require calibration against entry-point reference pressure ($P_0$) or differential barometry.
3. **RF Signal Penetration**: Deep basements and reinforced concrete shafts experience RF attenuation, triggering the $>10\text{s}$ Signal Lost alert until line-of-sight or mesh relay is re-established.
4. **Local Network Scope**: Designed for air-gapped field deployments on private LANs, tactical Wi-Fi access points, or private LTE/5G cells.

---

## Future Work

- **Differential Barometry Subsystem**: Automatic continuous recalibration using a fixed reference barometer at the ICP field station to cancel ambient weather fronts.
- **Pedestrian Dead-Reckoning (PDR) Drift Correction**: Extended Kalman filter fusing IMU step detection, gyro heading drift compensation, and known anchor checkpoints.
- **Direct IFC / BIM Ingestion**: Native parsing of Industry Foundation Classes (.ifc) building information models directly in the browser via WebAssembly.
- **Tactical Radio Hardware Bridge**: Native C/Rust daemon forwarding NMEA/protobuf packets from serial/USB tactical radios (Harris, Silvus, TrellisWare) into the WebSocket ingestion gateway.

---

## Documentation
- [System Architecture](docs/ARCHITECTURE.md)
- [Sensor Protocol Specification](docs/PROTOCOL.md)
- [Air-Gapped Field Deployment & TLS Guide](docs/OFFLINE_DEPLOYMENT.md)
# Live-Location-Tracking
