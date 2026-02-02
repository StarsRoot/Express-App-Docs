# Easy Express – User Guide

This guide explains how to use the Easy Express logistics and delivery management system. It covers all user roles, main workflows, and common tasks.

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Public Features](#public-features)
3. [Customer Dashboard](#customer-dashboard)
4. [Driver](#driver)
5. [Front Desk & Dispatcher](#front-desk--dispatcher)
6. [Branch Manager](#branch-manager)
7. [HR](#hr)
8. [Editor](#editor)
9. [Super Admin](#super-admin)
10. [Quick Reference](#quick-reference)

---

## Getting Started

### Sign Up & Sign In

- **Sign up**: Go to `/sign-up` or use the "Sign Up" link in the navigation.
- **Sign in**: Go to `/sign-in` or use the "Sign In" link.
- Authentication is handled by Clerk. After signing in, you are redirected to the appropriate dashboard based on your role.

### User Roles

| Role | Description | Main Dashboard |
|------|-------------|----------------|
| **Customer** | Sends and receives packages | `/dashboard` |
| **Driver** | Delivers packages, updates status | `/driver/dashboard` |
| **Front Desk** | Receives packages, ships, assigns drivers | `/frontdesk` |
| **Dispatcher** | Same as Front Desk | `/frontdesk` |
| **Branch Manager** | Manages branch, staff, reports | `/branch-manager` |
| **HR** | Staff, attendance, reports | `/hr` |
| **Editor** | Blog and gallery content | `/editor` |
| **Super Admin** | Full system control | `/admin` |
| **Accountant** | Financial reports | `/accountant` |
| **Store Manager** | Inventory and store operations | `/store-manager` |

---

## Public Features

These features are available without signing in.

### Track a Package

1. Go to **Track** in the navigation or visit `/tracking`.
2. Enter the **tracking number**.
3. Click **Search**.
4. View status, addresses, and history.
5. Click **View Details** to open the full tracking page.

**Direct tracking URL**: `/tracking/[trackingNumber]` (e.g. `/tracking/EE-123456`)

### QR Code Delivery Confirmation

When a driver scans a QR code at delivery:

1. The recipient (or driver) opens `/deliver/[trackingNumber]`.
2. The shipment is marked as **delivered**.
3. A confirmation message is shown.

### Request a Quote / Book a Shipment

1. Go to **Booking** or visit `/booking`.
2. Select a **branch** (pickup location).
3. Enter **pickup** and **delivery** addresses.
4. Add a **description** (optional).
5. Submit the request.
6. You receive a quote; after approval, a shipment is created.

### Browse Content

- **Home** (`/`): Landing page.
- **About Us** (`/about`): Company information.
- **Services** (`/services`): Service offerings.
- **Blog** (`/blog`): Articles and updates.
- **Contact** (`/contact`): Contact form and details.

---

## Customer Dashboard

**URL**: `/dashboard` (or `/dashboard/shipments`)

### My Shipments & Quotes

1. Sign in and open **Dashboard** → **My Shipments**.
2. View **quotes** (pending approval) and **shipments** (active/delivered).
3. Filter by status: All, Pending, In Transit, Delivered, Failed.
4. Use **Track** to open the tracking page.
5. Use **Invoice** to view or download the invoice.

### Book a Shipment

- Use **Book Shipment** or go to `/booking` to create a new shipment or quote.

### Profile & Settings

- **Profile** (`/dashboard/profile`): Update name, email, etc.
- **Settings** (`/settings`): Notification preferences.

---

## Driver

**URL**: `/driver/dashboard`

### View Assigned Shipments

1. Sign in and open **Driver Dashboard**.
2. See counts by status: Pending, In Transit, Delivered.
3. Click a shipment to view details.

### Update Shipment Status

1. Open a shipment card.
2. Click **Update Status**.
3. Choose the next status (e.g. Picked Up → In Transit → Delivered).
4. Add notes if needed and confirm.

### Record Failed Delivery

1. Open a shipment that is **Ready for Delivery** or **In Transit**.
2. Click **Record Failed Attempt**.
3. Select a reason and add notes.
4. Submit. The shipment returns for retry.

### Upload Proof of Delivery

1. Open a **delivered** shipment.
2. Click **Upload Photo**.
3. Upload an image as proof of delivery.

### My Packages

- **Packages** (`/driver/packages`): View front-desk packages assigned to you.

### Driver Registration (Quote Flow)

- When a quote is assigned to you, go to `/driver/quotes/[id]/register` to register the shipment and start delivery.

---

## Front Desk & Dispatcher

**URL**: `/frontdesk`

### Receive Packages

1. Go to **Front Desk** → **Shipments** → **Receive** tab.
2. Find shipments with status **Confirmed** (from driver).
3. Click **Receive Package**.
4. Add notes and confirm.

### Ship Packages (to Another Branch)

1. Go to **Packages** (`/frontdesk/packages`).
2. Filter packages with status **Received at Branch**.
3. Select packages to ship.
4. Click **Ship Packages**.
5. Choose the destination branch and confirm.

### Assign Driver

1. Go to **Shipments** → **Assign Driver** tab.
2. Find shipments ready for assignment.
3. Select a driver.
4. Confirm assignment.

### Retry Failed Delivery

1. Go to **Shipments** → **Retry Delivery** tab.
2. Find failed shipments.
3. Click **Retry Delivery**.
4. Add notes and confirm.

### Receive at Delivery Branch

1. Go to **Shipments** → **Delivery Receive** tab.
2. Find shipments **Shipped** or **In Transit** that arrived at your branch.
3. Click **Receive at Delivery Branch**.
4. Confirm.

### Manage Packages

- **Packages** (`/frontdesk/packages`): Register, edit, ship, and manage packages.
- **Quotes** (`/frontdesk/quotes`): View and manage quote requests.
- **Drivers** (`/frontdesk/drivers`): View and manage drivers.
- **Analytics** (`/frontdesk/analytics`): Branch-level analytics.

---

## Branch Manager

**URL**: `/branch-manager`

### Dashboard

- Overview of branch performance.
- Total packages, revenue, and trends.

### Manage Staff

- **Staff** (`/branch-manager/staff`): View and manage branch staff.

### Manage Drivers

- **Drivers** (`/branch-manager/drivers`): View and manage drivers for the branch.

### Packages

- **Packages** (`/branch-manager/packages`): View packages for the branch.

### Reports

- **Reports** (`/branch-manager/reports`): Shipment reports with filters for date range and status.
- Export to CSV.

---

## HR

**URL**: `/hr`

### Staff Management

- **Staff** (`/hr/staff`): View and manage staff across branches.
- Create, edit, and deactivate users.
- Assign roles and branches.

### Attendance

- **Attendance** (`/hr/attendance`): View attendance records.
- Check-in/check-out times and status.

### Branches

- **Branches** (`/hr/branches`): View branch information.

### Reports

- **Reports** (`/hr/reports`): Staff reports (shipments, revenue) and attendance reports.
- Filter by branch and date range.

---

## Editor

**URL**: `/editor`

### Blog Management

- **Posts** (`/editor/posts`): Create, edit, and publish blog posts.
- **Categories** (`/editor/categories`): Manage post categories.

### Gallery Management

- **Gallery** (`/editor/gallery`): Upload, edit, and manage gallery images.
- Add categories and tags.

### Content Calendar

- Use the content calendar to plan and schedule posts.

---

## Super Admin

**URL**: `/admin`

### Dashboard Overview

- System health.
- Total shipments, revenue, pending/completed/failed shipments.
- Performance metrics (average delivery time, on-time rate, etc.).
- Alerts (critical, high, medium, low).

### User Management

- **Users** (`/admin/users`): Create, edit, delete users.
- Assign roles and branches.
- Search and filter users.

### Branch Management

- **Branches** (`/admin/branches`): Create, edit, and manage branches.

### Shipment Types

- **Shipment Types** (`/admin/shipment-types`): Manage shipment types and pricing.

### Blog & Gallery

- **Blog** (`/admin/blog`): Manage blog posts and categories.
- **Gallery** (`/admin/gallery`): Manage gallery images.

### Reports

- **Reports** (`/admin/reports`): Shipment, accounting, attendance, inventory, and system analytics reports.

### System

- **System** (`/admin/system`): System configuration, maintenance, backups, logs.
- Cache management, rate limiting, diagnostics.

### Backup & Logs

- **Backup** (`/admin/backup`): Create and restore backups.
- **Logs** (`/admin/logs`): View system logs.

---

## Quick Reference

### Shipment Status Flow

```
pending → confirmed → picked_up → in_transit → ready_for_delivery → delivered
                                                      ↓
                                                   failed (retry available)
```

### Main URLs

| Page | URL |
|------|-----|
| Home | `/` |
| Track | `/tracking` |
| Book | `/booking` |
| Customer Dashboard | `/dashboard` |
| Driver Dashboard | `/driver/dashboard` |
| Front Desk | `/frontdesk` |
| Branch Manager | `/branch-manager` |
| HR | `/hr` |
| Editor | `/editor` |
| Admin | `/admin` |

### Navigation After Sign-In

After signing in, use the user menu (top right) to open your role-specific dashboard:

- **Super Admin**: Admin Dashboard, Editor Dashboard
- **Editor**: Editor Dashboard
- **Dispatcher / Front Desk**: Front Desk Dashboard
- **Branch Manager**: Branch Manager Dashboard
- **Driver**: Driver Dashboard
- **HR**: HR Dashboard
- **Customer**: Dashboard (shipments, profile, settings)

---

## Support

For technical setup and development, see:

- [DEVELOPMENT.md](./DEVELOPMENT.md) – Development environment setup
- [API.md](./API.md) – API documentation
- [DEPLOYMENT.md](./DEPLOYMENT.md) – Deployment instructions
