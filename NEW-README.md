# ChronoForge — System Update & Multi-Admin Architecture Guide

This document details the complete implementation of the updated **Authentication System**, **Admin Dashboard**, **Multi-Admin Management System**, and **Network Diagnostics Page** for **ChronoForge**, fulfilling all requirements given by the Senior.

---

## 🚀 1. Overview of Tasks & Implementation Summary

### **Senior Tasks Delivered:**
1. **Working Authentication Page with Login Credentials**:
   - Modern, secure administrator login screen (`#view-auth`).
   - Dedicated credential inputs for **Admin User ID / Username** and **Password** (with show/hide toggle).
   - Instant quick-fill card for default administrator credentials (`admin` / `admin123`).
   - Session authentication with secure Bearer tokens and localStorage persistence.

2. **Proper Working Admin Dashboard**:
   - Central administrative command center (`#view-admindashboard`).
   - **4 Live Metric Stat Cards**:
     - 🛡️ **Administrators Network**: Multi-admin system status & active session tracking.
     - 📅 **Schedule Versions**: Active timetable version & weekly scheduled bell rows.
     - ⚡ **ESP32 & Clock Status**: Live hardware connection (ONLINE, MOCK MODE, OFFLINE), IP, and latency.
     - 📜 **Audit & Security Logs**: Total logged events & system warning/error counts.
   - **System Operations & Hardware Maintenance Controls**:
     - Live Hardware Mock Mode toggle.
     - Manual Bell Relay Pulse Test button with duration feedback.
     - Purge Historical Schedule Archives button (safely retains latest active schedule).
     - Clear System Activity Logs button.
   - **Live System Audit Trail**:
     - Real-time audit log stream displaying recent administrative logins, account modifications, and hardware operations.
   - Quick navigation shortcuts to the Timetable Dashboard ("Real Page") and Network Diagnostics.

3. **Removal of Normal User Stuff (No More Normal Users)**:
   - System is now strictly institutional: **only Administrators exist**.
   - Removed public self-registration/signup forms and endpoints.
   - Removed legacy `role = 'user'` seed accounts and purged non-admin users from the database.
   - Removed user/demote/promote logic.
   - Topbar displays `ADMIN` badge and current admin ID.

4. **Multi-Admin System (Admin Creates More Admin User IDs & Passwords)**:
   - Inside the **Admin Dashboard**, active administrators can:
     - View all registered administrators with their Admin ID, Username, Email, Created Date, and Last Login.
     - **Create New Administrator Accounts**: Input custom Admin User ID / Username, Email (optional), and Password (with confirmation check).
     - **Change / Reset Administrator Passwords**: Dedicated modal dialog allowing administrators to update credentials.
     - **Delete Administrator Accounts**: Protected with safety checks (an admin cannot delete their own active session, and the system prevents deleting the last remaining admin account).

---

## 🛠️ 2. Technical Architecture & Endpoints

### **A. Database Schema (`backend/chronoforge.db`)**
```sql
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    email TEXT UNIQUE,
    password_hash TEXT NOT NULL,
    role TEXT NOT NULL DEFAULT 'admin',
    created_at TEXT NOT NULL,
    last_login TEXT
);

CREATE TABLE IF NOT EXISTS sessions (
    token TEXT PRIMARY KEY,
    user_id INTEGER NOT NULL,
    created_at TEXT NOT NULL,
    FOREIGN KEY(user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### **B. Seeded Default Credentials**
- **Default Admin Account**:
  - **Admin User ID**: `admin`
  - **Password**: `admin123`
  - **Role**: `admin`
- **College Wi-Fi Credentials**:
  - **SSID**: `en24270776`
  - **Password**: `Skn@9146777532`

### **C. REST API Endpoints**

| Method | Endpoint | Access | Description |
|--------|----------|--------|-------------|
| `POST` | `/api/auth/login` | Public | Authenticates admin credentials, returns session token |
| `POST` | `/api/auth/logout` | Authenticated | Destroys current session token |
| `GET`  | `/api/auth/me` | Authenticated | Returns current authenticated admin profile |
| `POST` | `/api/auth/signup` | Blocked (403) | Public registration disabled (admins created from dashboard) |
| `GET`  | `/api/admin/stats` | Admin | Returns system metrics (admin count, schedule info, logs, ESP32 status) |
| `GET`  | `/api/admin/users` | Admin | Returns directory of all administrators |
| `POST` | `/api/admin/users` | Admin | Creates a new administrator with Admin ID and Password |
| `PUT`  | `/api/admin/users/<id>/password` | Admin | Updates/resets password for an administrator |
| `DELETE`| `/api/admin/users/<id>` | Admin | Deletes an administrator (protected against self/last-admin deletion) |
| `GET`  | `/api/admin/recent-logs` | Admin | Returns the latest 10 system and audit logs |
| `POST` | `/api/admin/logs/clear` | Admin | Clears all activity logs |
| `POST` | `/api/admin/schedules/purge` | Admin | Purges historical schedule versions |
| `GET`  | `/api/network/diagnostics` | Public/Admin | Wi-Fi and ESP32 ping diagnostic and latency check |

---

## 🏃 3. How to Run and Test

### **Step 1: Start the Backend Server**
```bash
python backend/app.py
```
Server runs on: **`http://127.0.0.1:5000`**

### **Step 2: Login Flow**
1. Open `http://127.0.0.1:5000`.
2. Notice the Authentication Page with the login credentials card.
3. Click **⚡ Fill Admin Credentials** (or enter `admin` and `admin123`), then click **Authenticate & Sign In**.
4. You are immediately redirected to the **Admin Dashboard**.

### **Step 3: Multi-Admin Flow**
1. In the **Admin Dashboard**, locate the **System Administrators** panel.
2. Click **+ Create New Admin**.
3. Enter a new Admin User ID (e.g. `admin_aiman`), an optional email, and a password (e.g. `pass1234`). Confirm the password and submit.
4. Verify the new administrator appears in the directory table and the administrator stat counter increments.
5. Click **🔑 Password** next to an admin account to update or reset their password.
6. Test deletion safety: Attempting to delete your own active account is prohibited.

### **Step 4: Real Page & Hardware Flow**
1. In the top navigation bar or from the Admin Dashboard, click **⚡ Go to Real Page (Timetable Dashboard)** to manage bell schedules.
2. Click **🌐 Diagnostics** to test the College Wi-Fi (`en24270776`) and ESP32 link status.

---

## 📄 File Sitemap
- `backend/app.py` — Flask server, multi-admin database logic, authentication, and REST APIs.
- `frontend/index.html` — Updated Admin Authentication screen, Multi-Admin Dashboard, and modals.
- `frontend/script.js` — Client-side authentication, multi-admin management, password dialogs, and real-time statistics.
- `frontend/style.css` — Modern glassmorphism styles, credential badges, and dashboard layout.
