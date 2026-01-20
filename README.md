

```markdown
# CholoSave (University Computer Security Project)

CholoSave is a PHP + MySQL web application built as a **Computer Security course project**.  
This repository focuses on implementing **practical security controls** (authentication hardening, form protection, OTP flows, alerting, and safe database interaction) inside a finance/savings-style web platform with group features.

## Key Features

### ✅ Security Features Implemented
- **Password Security**
  - Strong password policy validation (`password_utils.php`)
  - Secure password hashing (supports **Argon2id / Argon2i / bcrypt** fallback)
  - Password verification on login using `password_verify()`

- **CSRF Protection**
  - CSRF token generation + verification on forms (e.g., `register.php`, `contact_us.php`)

- **Bot Protection (CAPTCHA)**
  - **Cloudflare Turnstile** verification on sensitive forms (e.g., Register / Contact Us)

- **Rate Limiting / Throttling**
  - Session-based rate limiting for form submissions (e.g., registration/contact) to reduce spam/bruteforce attempts

- **Session Security**
  - Secure cookie flags and strict session options (HttpOnly, Secure, SameSite)
  - `session_regenerate_id(true)` after login/registration to reduce session fixation risk

- **OTP-based Verification**
  - **Forgot Password OTP** email flow (`forgot_password.php` → `verify_otp.php` → `reset_password.php`)
  - OTP expiry enforced using server-side timestamp checks

- **Security Alerting**
  - **Email alert on login from a new IP** (`login.php` compares `last_login_ip`)

- **SQL Injection Mitigation**
  - Uses prepared statements (`$conn->prepare(...)`) across multiple modules

> Note: Some planned items in the "Security Features" notes may not be implemented everywhere yet (example: full login brute-force lockout). This project demonstrates multiple real security controls integrated into a working application.

---

## Tech Stack
- **Backend:** PHP
- **Database:** MySQL / MariaDB
- **Frontend:** HTML, CSS, Tailwind (CDN), JS
- **Email:** PHPMailer
- **CAPTCHA:** Cloudflare Turnstile
- **WebSocket Chat:** Ratchet (PHP) on port `8080`
- **Optional AI Tips Service:** Flask + Groq API (`ai_tips/generate_tips.py`)

---

## Project Structure (High-level)

- `/admin` — admin dashboards and management pages  
- `/group_admin` — group admin features (payments, OTP verification, reports, etc.)
- `/group_member` — group member dashboards and actions  
- `/includes` — shared headers/footers/assets  
- `/ai_tips` — optional AI service for finance tips (Flask)  
- `server.php` — Ratchet WebSocket server for real-time chat  
- `cholosave_db.sql` — database schema and sample data  

---

## Setup Instructions (Localhost using XAMPP)

### 1) Requirements
- XAMPP / WAMP / LAMP (PHP + MySQL)
- PHP CLI (for Ratchet server)
- Composer

### 2) Put Project in htdocs
Copy the project folder into:
- `C:\xampp\htdocs\` (Windows)  
or  
- `/var/www/html/` (Linux)

Example:
```

xampp/htdocs/CholoSave-CS/

```

### 3) Create Database
1. Open `phpMyAdmin`
2. Create a DB named:
```

cholosave_cs

```
3. Import:
```

cholosave_db.sql

````

### 4) Configure Database Connection
Update `db.php` if needed:
```php
$host = "localhost";
$username = "root";
$password = "";
$dbname = "cholosave_cs";
````

### 5) Install PHP Dependencies

In the project root:

```bash
composer install
```

(There are also composer files inside `group_admin/` and `group_member/`. If you use those modules separately, run composer there too.)

---

## ⚠️ Security Setup Before Running (IMPORTANT)

### A) Remove Hardcoded Email Credentials (PHPMailer)

Currently, SMTP credentials are hardcoded in files like:

* `login.php`
* `forgot_password.php`

✅ Recommended fix:

1. Create a config file like `config/secrets.php` (and add it to `.gitignore`)
2. Load SMTP settings from environment variables

Example env variables:

* `SMTP_HOST`
* `SMTP_USER`
* `SMTP_PASS`
* `SMTP_PORT`

### B) Turnstile Keys

Turnstile secret is currently hardcoded (e.g., `register.php`, `contact_us.php`).

✅ Recommended fix:

* Store Turnstile keys in env/config and never commit them to GitHub.

---

## Running the App

### 1) Start Apache + MySQL

From XAMPP control panel.

### 2) Open in Browser

Example:

```
http://localhost/CholoSave-CS/
```

---

## Real-time Chat (WebSocket)

The project includes a Ratchet WebSocket server (`server.php`) running on:

```
ws://localhost:8080
```

To run it:

```bash
php server.php
```

Keep this terminal running while using the chat feature.

---

## Optional: AI Tips Service (Flask + Groq)

There is an optional Python microservice:

* `ai_tips/generate_tips.py`

It expects:

* Python installed
* `GROQ_API_KEY` in environment

Example run:

```bash
cd ai_tips
pip install -r requirements.txt
set GROQ_API_KEY=your_key_here   # Windows
export GROQ_API_KEY=your_key_here # Linux/Mac
python generate_tips.py
```

---

## Database Notes (Extra Tables You May Need)

Some security features reference extra tables that may not be in `cholosave_db.sql` yet, depending on your version.

### Example: `security_logs` table (used in `register.php`)

```sql
CREATE TABLE security_logs (
  id INT AUTO_INCREMENT PRIMARY KEY,
  event_type VARCHAR(50) NOT NULL,
  details TEXT,
  ip_address VARCHAR(45),
  timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Example: `payment_otps` table (used in OTP payment verification)

```sql
CREATE TABLE payment_otps (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  group_id INT NOT NULL,
  otp VARCHAR(10) NOT NULL,
  otp_expiry DATETIME NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  transaction_id VARCHAR(100),
  payment_method VARCHAR(50),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## Demo Roles

The app supports role-based access via `users.role`:

* `user`
* `group_admin`
* `admin`

Admin dashboard route:

* `/admin/admin_dashboard.php`

---

## Security Disclaimer

This is an academic security project. Before deploying publicly:

* Remove all hardcoded secrets
* Enforce HTTPS in production
* Add server-side login brute-force lockouts
* Audit authorization checks for every sensitive action
* Use a proper secrets manager / `.env` system

---

## Team

Built by me and my teammates as a **University Computer Security course project**.

---

## License

This project is for educational use. Add a license if you plan to open-source it.

```

---

 
- and give you a **safe “before pushing to GitHub” checklist** (so you don’t accidentally leak passwords/keys).
```
