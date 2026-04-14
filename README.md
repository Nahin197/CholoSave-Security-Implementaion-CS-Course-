# 🔐 CholoSave — Secure Collaborative Savings Platform
### Computer Security Course Project | *Team: Team Xenon*

![PHP](https://img.shields.io/badge/PHP-8.x-blue?logo=php)
![MySQL](https://img.shields.io/badge/MySQL-Database-orange?logo=mysql)
![Security](https://img.shields.io/badge/Security-Hardened-red?logo=shield)
![License](https://img.shields.io/badge/License-Academic-green)

---

## 📌 Project Overview

**CholoSave** is a feature-rich, security-hardened **collaborative savings and DPS (Deposit Pension Scheme)** web application built with PHP and MySQL. The project was developed as part of a **Computer Security course** with the primary goal of implementing and demonstrating a wide array of real-world security measures in a functional financial application.

The application allows users to create and join savings groups, make installment payments, request loans, manage investments, and communicate — all protected by an enterprise-grade, multi-layer security architecture following the threat model of **Infiltration → Propagation → Aggregation → Exfiltration**.

---

## 🎯 Security Threat Model

The security architecture follows a four-phase threat mitigation model defined by Team **Breach Quadrilateral**:

| Phase | Threat | Countermeasures Implemented |
|---|---|---|
| **Infiltration** | Unauthorized access | Bcrypt/Argon2id hashing, Rate limiting, 2FA (OTP), CAPTCHA |
| **Propagation** | Privilege escalation | RBAC, Group-Based Access Control, Session management, Secure cookies |
| **Aggregation** | Data manipulation | Input validation, SQL injection prevention, XSS protection, Security/Activity logs |
| **Exfiltration** | Data theft | HTTPS enforcement, Encrypted storage, User activity logs, Email/SMS alerts |

---

## 🛡️ Security Features Implemented

### 🔑 1. Authentication & Password Security
- **Argon2id Password Hashing** — Passwords hashed with Argon2id (memory: 64MB, 4 iterations, 3 threads) via a custom `PasswordUtils` class
- **Automatic Salt Generation** — PHP's `password_hash()` auto-generates unique salts per password
- **Password Strength Enforcement** — Minimum 8 characters, must include uppercase, lowercase, numbers, and symbols
- **Automatic Password Rehash** — Checks if existing hashes need upgrading via `needsRehash()`

### 🔁 2. Two-Factor Authentication (2FA)
- **Login OTP** — After correct password entry, a 6-digit OTP is sent via email using **PHPMailer + Gmail SMTP**
- **OTP Hashing** — OTPs are SHA-256 hashed before storage; never stored in plaintext
- **OTP Expiry** — Login OTPs expire in **5 minutes**; password-reset OTPs expire in **2 minutes**
- **Payment OTP** — Financial transactions require a separate OTP verification step
- **OTP Attempt Limiting** — Max 3 wrong OTP attempts before the OTP is invalidated

### 🤖 3. CAPTCHA Protection (Cloudflare Turnstile)
- All public forms (Login, Register, Contact Us, Create Group) are protected by **Cloudflare Turnstile CAPTCHA**
- Bot-verified via server-side cURL call to Cloudflare's `siteverify` endpoint
- CAPTCHA token validated with SSL peer verification enabled

### 🚦 4. Rate Limiting
- **Login**: Max 5 attempts per 5 minutes per IP
- **Registration**: Max 3 submissions per 5 minutes per IP
- **Contact Us**: Max 3 messages per 5 minutes per IP
- **Group Join Requests**: Max 3 requests per 5 minutes per IP
- **Profile Updates**: Max 5 updates per 10 minutes
- Rate limit state tracked in PHP sessions with automatic time-window reset

### 🔒 5. Session Security
- `session.cookie_httponly = 1` — Prevents JavaScript XSS cookie theft
- `session.cookie_secure = 1` — Cookies only transmitted over HTTPS
- `session.use_strict_mode = 1` — Rejects uninitialized session IDs
- `session.cookie_samesite = Strict` — Prevents CSRF via cookie
- **Session ID Regeneration** on login (`session_regenerate_id(true)`)
- **Pending 2FA sessions** use a separate `pending_2fa_user_id` key to prevent session fixation

### 🌐 6. HTTPS Enforcement
- All pages redirect HTTP to HTTPS using server-side header redirect
- Localhost/127.0.0.1 is whitelisted for local development (no forced redirect)

### 🛡️ 7. CSRF Protection
- CSRF tokens generated via `bin2hex(random_bytes(32))`
- Validated on every POST submission for Register, Contact Us, and Forum forms
- CSRF token rotated after every successful submission

### 💉 8. SQL Injection Prevention
- **100% Prepared Statements** — All database interactions use `mysqli::prepare()` and `bind_param()`
- No raw string interpolation in SQL queries on security-critical pages
- `FILTER_SANITIZE_NUMBER_FLOAT` and `intval()` used for numeric inputs

### 🧹 9. XSS Prevention
- `sanitize_input()` function used throughout:
  ```php
  $data = htmlspecialchars($data, ENT_QUOTES, 'UTF-8');
  ```
- All output rendered with `htmlspecialchars()` and `ENT_QUOTES | ENT_SUBSTITUTE`
- `htmlspecialchars()` applied to all reflected data in HTML output

### 🏛️ 10. Role-Based Access Control (RBAC)
- **3 Access Levels**: Platform Admin, Group Admin, Group Member
- Session role checks guard every admin page (`$_SESSION['role'] !== 'admin'`)
- Group admin status verified by querying `my_group.group_admin_id` on each request
- Unauthorized access redirects to custom error page

### 📋 11. Security & Activity Logging
- **`security_logs` table** — Records event type, details, IP address, timestamp for:
  - Login success/failure, 2FA events, CAPTCHA failures
  - Rate limit violations, CSRF attempts, password changes
  - Registration events, OTP send/fail events
- **`activity_log` table** — Tracks group join requests, duplicate join attempts, membership changes
- All logging uses parameterized queries

### 📧 12. Email Notifications
- Password change email notification sent after successful reset
- Payment confirmation email sent after OTP-verified transaction
- PHPMailer with SMTP over STARTTLS on port 587

### 🔐 13. Secure File Uploads (Profile Pictures)
- Whitelist of allowed extensions: `.jpg`, `.jpeg`, `.png`
- Uploaded files renamed to `user_{id}.{ext}` to prevent path traversal
- Upload directory access rate-limited (5 uploads per 10 minutes)

---

## 👥 User Roles & Permissions

### 🔴 Platform Admin (`/admin/`)
- View platform-wide statistics (total savings, members, groups, investments)
- Manage financial expert profiles (Add, Edit, Delete)
- View all contact/support messages
- Access savings overview charts

### 🟡 Group Admin (`/group_admin/`)
- Full group dashboard with financial graphs
- Approve/reject member join and leave requests
- Manage loan requests (create polls, approve, track repayment)
- Manage withdrawal requests
- View payment history & manage investment records
- Generate downloadable PDF financial reports (via FPDF)
- Manage group settings and savings closure

### 🟢 Group Member (`/group_member/`)
- View group dashboard (group savings, personal savings, loans, withdrawals)
- Make installment payments (via bKash, Rocket, Nagad) with OTP verification
- Submit emergency loan requests with member voting (polls)
- Request withdrawals
- View loan history, payment history, investment details
- Participate in polls

---

## ✨ Application Features

### 💰 Financial Features
- **DPS Plans** — Monthly or weekly installment savings
- **Group Goal Tracking** — Visual progress bar toward savings goal
- **Emergency Fund** — Group-level reserve fund with admin-controlled loans
- **Investment Management** — Group admins can log and track investments and returns
- **PDF Financial Reports** — Downloadable reports with group overview, transactions, loans, withdrawals

### 🗳️ Democratic Governance
- **Voting Polls** — Members vote on join requests, admin loan requests, and other decisions
- Polls auto-created when a member requests to join or when admin requests a loan

### 🏆 Leaderboard System
- Groups earn points based on activity:
  - **+5 points**: Group created
  - **+5 points**: New member joins
  - **+1% of payment**: Per member installment payment
  - **+1% of emergency fund**: At group creation
  - **−10 points**: Member leaves

### 💬 Community Features
- **Group Chat** — Real-time messaging within groups (Ratchet WebSocket)
- **Community Forum** — Q&A board with CSRF-protected question submission, replies, and view counts
- **Expert Directory** — Browse and contact financial experts
- **AI Chatbot** — Rule-based chatbot assistant for account, login, payment, and group queries
- **Notification Center** — Real-time notification cards with read/delete functionality
- **Payment Reminders** — Automated alerts for upcoming installment dues

### 👤 User Management
- User profile with photo upload (JPG/PNG), phone, role display
- In-app password change with strength validation
- Forgot Password → OTP via email → Secure Reset flow

---

## 🗂️ Project Directory Structure

```
CholoSave/
├── 📄 index.php                  # Landing / home page
├── 📄 login.php                  # Secure login with 2FA & CAPTCHA
├── 📄 register.php               # Registration with CSRF, CAPTCHA, rate limiting
├── 📄 forgot_password.php        # Forgot password (email OTP)
├── 📄 verify_otp.php             # OTP verification for password reset
├── 📄 reset_password.php         # Password reset with strength check & email alert
├── 📄 logout.php                 # Session destroy
├── 📄 password_utils.php         # Argon2id hashing, verification, strength validation
├── 📄 session.php                # Session management helpers
├── 📄 db.php                     # Database connection (root)
├── 📄 groups.php                 # Browse & join groups with rate-limited join requests
├── 📄 create_group.php           # Multi-step group creation with CAPTCHA
├── 📄 group_session.php          # Group session switching
├── 📄 forum.php                  # Community Q&A forum
├── 📄 leaderboard.php            # Groups ranked by activity points
├── 📄 notification.php           # User notification center
├── 📄 profile.php                # User profile & password change
├── 📄 contact_us.php             # Contact form (CSRF + CAPTCHA + rate limiting)
├── 📄 chat_bot_final.php         # AI chatbot assistant (NLP-style JS)
├── 📄 expert.php                 # Financial expert directory
├── 📄 payment_reminders.php      # Payment due date reminders
├── 📄 cholosave_db.sql           # Full database schema
├── 📄 Security Features          # Security feature specification doc
├── 📄 Security_measures.txt      # Security threat model document
├── 📄 composer.json              # Dependencies (PHPMailer, Ratchet)
│
├── 📁 admin/                     # Platform admin panel
│   ├── admin_dashboard.php       # Platform stats dashboard
│   ├── session.php               # Admin RBAC session guard
│   ├── add_expert.php
│   ├── edit_expert.php
│   ├── delete_expert.php
│   └── ...
│
├── 📁 group_admin/               # Group admin panel
│   ├── group_admin_dashboard.php # Group financial dashboard
│   ├── group_admin_loan_request.php
│   ├── group_admin_withdraw_request.php
│   ├── group_admin_payment_history.php
│   ├── group_generate_report.php # FPDF PDF report generator
│   ├── otp_verify_payment.php    # OTP-verified payment processing
│   ├── investment_history.php
│   ├── member_join_request.php
│   └── ...
│
├── 📁 group_member/              # Group member panel
│   ├── group_member_dashboard.php
│   ├── group_member_payment.php
│   ├── group_member_emergency_loan_req.php
│   ├── group_member_withdraw_request.php
│   ├── otp_verify_payment.php
│   └── ...
│
├── 📁 includes/                  # Shared header/footer templates
├── 📁 assets/                    # CSS, images, icons
├── 📁 uploads/                   # User-uploaded profile pictures
└── 📁 vendor/                    # Composer dependencies (PHPMailer, Ratchet)
```

---

## 🛠️ Technology Stack

| Layer | Technology |
|---|---|
| **Backend** | PHP 8.x |
| **Database** | MySQL via MySQLi |
| **Password Hashing** | Argon2id (PHP `password_hash`) |
| **Email** | PHPMailer 6.x + Gmail SMTP (STARTTLS) |
| **CAPTCHA** | Cloudflare Turnstile |
| **PDF Generation** | FPDF |
| **WebSocket** | Ratchet (for real-time chat) |
| **Frontend** | HTML5, Tailwind CSS (CDN), Vanilla JS |
| **Fonts** | Google Fonts (Poppins, Inter) |
| **Icons** | FontAwesome 6 |
| **Charts** | Chart.js |
| **UI Alerts** | SweetAlert2 |

---

## ⚙️ Setup Instructions

### Prerequisites
- PHP 8.0+ with `mysqli`, `openssl`, `curl`, `intl` extensions
- MySQL 5.7+ or MariaDB
- Composer (for PHPMailer & Ratchet)
- A web server (Apache/Nginx) or XAMPP/WAMP

### Installation Steps

**1. Clone the repository**
```bash
git clone https://github.com/your-username/CholoSave-Security-Implementation.git
cd CholoSave-Security-Implementation
```

**2. Install PHP dependencies**
```bash
composer install
```

**3. Import the database**
```bash
mysql -u root -p < cholosave_db.sql
```
Or import via **phpMyAdmin** → Create DB `cholosave_cs` → Import `cholosave_db.sql`

**4. Configure database connection**

Edit `db.php` (and all `admin/db.php`, `group_admin/db.php`, `group_member/db.php`):
```php
$host = "localhost";
$username = "root";
$password = "your_password";
$dbname = "cholosave_cs";
```

**5. Configure PHPMailer (Gmail SMTP)**

In `login.php`, `forgot_password.php`, `reset_password.php`, and OTP payment files — update:
```php
$mail->Username = 'your-email@gmail.com';
$mail->Password = 'your-app-password';  // Gmail App Password
```

**6. Configure Cloudflare Turnstile**

Replace the site key and secret key in all forms:
```php
$turnstile_secret = "your-turnstile-secret-key";
```
```html
<div class="cf-turnstile" data-sitekey="your-turnstile-site-key"></div>
```

**7. Configure virtual host**

Apache `httpd-vhosts.conf` (or equivalent):
```apache
<VirtualHost *:80>
    ServerName localhost
    DocumentRoot "/path/to/project"
    Alias /CholoSave-CS "/path/to/project"
</VirtualHost>
```

**8. Access the application**
```
http://localhost/CholoSave-CS/
```

---

## 🗄️ Database Schema Highlights

Key tables in `cholosave_db.sql`:

| Table | Purpose |
|---|---|
| `users` | User accounts with hashed passwords, OTP fields, role |
| `my_group` | Savings groups with payment methods (bKash/Rocket/Nagad) |
| `group_membership` | Member-group relationships with status, admin flag |
| `savings` | Payment/installment records |
| `loan_request` | Loan applications with approval workflow |
| `withdrawal` | Withdrawal requests with approval workflow |
| `investments` | Group investment records |
| `investment_returns` | Returns on investments |
| `polls` | Democratic voting records |
| `leaderboard` | Group point rankings |
| `notifications` | User notification centre entries |
| `security_logs` | Security event audit trail (event_type, IP, timestamp) |
| `activity_log` | User action logs (join, leave, payment actions) |
| `payment_otps` | Temporary OTP storage for payment verification |
| `contact_us` | Contact form submissions |
| `questions` / `replies` | Forum posts and replies |

---

## 🔏 Security Configurations Summary

```
✅ Argon2id password hashing (64MB memory, 4 iterations, 3 threads)
✅ SHA-256 hashed OTPs (never plaintext)
✅ Session: HttpOnly + Secure + SameSite=Strict cookies
✅ Session ID rotation on login
✅ HTTPS enforcement (non-localhost)
✅ CSRF token on all state-changing forms
✅ Cloudflare Turnstile CAPTCHA on all public forms
✅ Rate limiting (login: 5/5min, register: 3/5min, contact: 3/5min)
✅ OTP attempt limiting (3 max before invalidation)
✅ 100% Prepared Statements (no raw SQL)
✅ htmlspecialchars() on all user output
✅ Admin role verified per-request (not just at login)
✅ Group admin verified per-request from DB
✅ Security event logging to DB (IP + timestamp)
✅ Activity logging for audit trail
✅ Email alerts on password change
✅ File upload: extension whitelist + renamed files
```

---

## 👨‍💻 Team — Team Xenon

This project was developed as an academic security implementation for a **Computer Security** course.

## 👨‍💻 Authors

- **Developer** — [@Nahin](https://github.com/Nahin197)
- **Developer** — [@Montasir](https://github.com/Irfanuzzaman-Montasir)
- **Developer** — [@Imran](https://github.com/imran-bhuiyan)


---

## 📄 License

This project is submitted as an academic assignment. All rights reserved by the development team. Not for commercial use.

---

> *"Security is not a product, but a process."* — Bruce Schneier
