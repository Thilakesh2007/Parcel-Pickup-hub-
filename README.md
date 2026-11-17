# Parcel Pickup Hub System
**Smart Parcel Locker Management System**

An academic, production-style Smart Parcel Locker Management System built with a Node.js/Express REST backend, React (Vite & Tailwind CSS) frontend, and relational database architecture.

---

## 1. Project Objective & Core Capabilities

The **Parcel Pickup Hub System** solves the last-mile delivery challenge by automating parcel receipt, storage, and customer self-service retrieval:

1. **Automatic Locker Allocation Engine**: Calculates parcel dimensions (Length, Width, Height, Weight) with oriented 3D fit checks and automatically allocates the **smallest suitable available locker** (Small, Medium, Large) to optimize hub occupancy.
2. **OTP Security Verification**: Cryptographically generates unique 6-digit OTP codes per parcel with bcrypt hashing, expiration windows, and brute-force attempt limiting (auto-lockout after 4 failed attempts).
3. **Interactive Touchscreen Kiosk Simulator**: Visualizes the physical locker terminal interface where customers input tracking numbers or phone numbers, enter the OTP, trigger solenoid door unlocking, view illuminated compartments, and confirm door closure.
4. **Physical Expiry & Return Management**: Uncollected parcels past their pickup deadline transition to `EXPIRED` status. Crucially, the locker is held in `EXPIRED_HOLD` until Hub Staff physically removes the item and confirms clearance, preventing phantom locker re-allocation.
5. **Role-Based Access Control (RBAC)**: Distinct interfaces and permissions for **Admin**, **Vendor**, **Customer**, and **Hub Staff**, equipped with a 1-click Demo Persona Switcher for seamless demonstration.
6. **Analytics & Reports**: Real-time locker occupancy meters, turnover rates, average pickup turnaround duration, CSV data export, and printable summaries.
7. **Relational Database Design**: Complete normalized MySQL `schema.sql` script with foreign keys, constraints, indexes, and comprehensive seed data.

---

## 2. Technology Stack

- **Frontend**: React 19, TypeScript, Vite, Tailwind CSS, Lucide React, Motion.
- **Backend**: Node.js, Express 4, REST API architecture, JWT authentication (`jsonwebtoken`), Bcrypt password hashing (`bcryptjs`).
- **Database**:
  - `database/schema.sql`: Complete MySQL DDL script with 6 normalized tables, foreign keys, constraints, and seed data.
  - Runtime Database: SQLite relational engine (`better-sqlite3`) executing foreign keys (`PRAGMA foreign_keys = ON`), atomic transactions (`db.transaction`), and indices out-of-the-box.

---

## 3. User Roles & Seed Demo Personas

All demo accounts share the default password: **`Password123!`** (also accessible via 1-click buttons in the UI):

| Role | Name | Email | Purpose |
| :--- | :--- | :--- | :--- |
| **Admin** | System Administrator | `admin@hub.local` | Full system control, locker hardware management, analytics, user audit logs |
| **Vendor** | Apex Logistics | `apex@vendor.local` | Inward parcels, trigger auto-allocation, view OTP dispatches |
| **Vendor** | FastTrack Couriers | `fasttrack@vendor.local` | Courier manifest tracking & parcel statuses |
| **Customer** | Sarah Jenkins | `sarah@customer.local` | Self-service pickup portal, view assigned OTPs, collect at kiosk |
| **Customer** | David Chen | `david@customer.local` | Track shipments, inspect expired/return records |
| **Hub Staff**| Elena Rostova | `staff@hub.local` | Physical compartment inspections, clearing expired parcels, locker maintenance |

---

## 4. Locker Dimensions & Allocation Logic

Lockers are classified into standard hardware categories:

| Category | Max Dimensions (L × W × H) | Max Weight | Common Contents |
| :--- | :--- | :--- | :--- |
| **SMALL** | 15.0 × 20.0 × 30.0 cm | 10.0 kg | Earbuds, smartphones, watches, jewelry, books |
| **MEDIUM** | 30.0 × 35.0 × 45.0 cm | 20.0 kg | Shoeboxes, laptops, apparel, tablets, small kitchenware |
| **LARGE** | 50.0 × 50.0 × 60.0 cm | 35.0 kg | Microwaves, small appliances, backpacks, bulk parcels |

### Allocation Algorithm (`server/services/lockerAllocationService.ts`):
1. **3D Rotation Fit Check**: Sorts dimensions $[p_1, p_2, p_3] \le [l_1, l_2, l_3]$ to check if parcel fits in any orientation.
2. **Smallest-Fit Hierarchy**: Identifies the minimum required size tier (`SMALL` $\to$ `MEDIUM` $\to$ `LARGE`).
3. **Availability & Smart Upgrades**: If all lockers in the minimum required size are occupied, the engine automatically checks and upgrades to the next available larger size tier.
4. **Concurrency Safety**: Allocation executes within an atomic database transaction (`db.transaction`) to prevent double-booking.

---

## 5. Expiry & Physical Return Workflow

A key requirement in real-world locker hubs:
1. The background monitor (`server/jobs/expiryJob.ts`) scans every 60 seconds (or on-demand via the UI).
2. Parcels where `pickup_deadline < NOW()` and status is `READY_FOR_PICKUP` transition to `EXPIRED`.
3. The assigned locker is set to `EXPIRED_HOLD`.
4. **The locker is NOT made available immediately**.
5. Hub Staff or Admin inspects the physical locker, opens the compartment, retrieves the package, and clicks **"Clear & Release Locker"**.
6. Only after staff confirmation does the locker status return to `AVAILABLE` and the parcel status update to `RETURNED`.

---

## 6. Installation & Execution

### Prerequisites:
- Node.js (v18 or later)
- npm

### Quick Start:
```bash
# 1. Install dependencies
npm install

# 2. Run full-stack dev server (Vite + Express on Port 3000)
npm run dev
```

### Production Build:
```bash
npm run build
npm start
```

### MySQL Database Setup (Optional for standalone MySQL deployments):
To run on a standalone MySQL instance:
```bash
mysql -u root -p < database/schema.sql
```

---

## 7. REST API Endpoints Summary

- `POST /api/auth/login` - Authenticate user & return JWT
- `POST /api/auth/register` - Register customer/vendor
- `POST /api/auth/demo-switch` - 1-click demo role switcher
- `GET /api/auth/me` - Authenticated user session
- `POST /api/parcels` - Register parcel with dimension-based auto-allocation
- `GET /api/parcels` - Scoped parcel directory (role-based)
- `GET /api/lockers` - Visual wall hardware states & occupancy
- `POST /api/lockers` - Add new locker (Admin)
- `POST /api/pickup/verify-otp` - Kiosk OTP validation & door unlock
- `POST /api/pickup/confirm` - Door closure & compartment release
- `GET /api/returns` - List return requests
- `POST /api/returns/:id/clear` - Staff physical locker clearance & release
- `POST /api/returns/scan-expiry` - Trigger immediate expiry scan
- `GET /api/reports/statistics` - Hub KPIs, turnaround duration, capacity gauges
- `GET /api/reports/export` - Export full dataset for CSV download
