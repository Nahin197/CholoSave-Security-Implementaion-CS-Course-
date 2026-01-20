

---
# 🚀 CholoSave

### *University Computer Security Project*

CholoSave is a **PHP + MySQL web application** developed as part of a **Computer Security course project**.
The project demonstrates **real-world security implementations** such as authentication hardening, CSRF protection, OTP verification, secure session handling, and database protection — all inside a finance/savings-style group management system.

---

## 🔐 Key Security Features

### ✅ Authentication & Password Security

* Strong password policy validation (`password_utils.php`)
* Secure password hashing:

  * **Argon2id / Argon2i**
  * Fallback to **bcrypt**
* Secure login verification using `password_verify()`

### ✅ CSRF Protection

* CSRF token generation and validation
* Applied on sensitive forms such as:

  * Registration
  * Contact form
  * Password reset

### ✅ Bot Protection

* **Cloudflare Turnstile CAPTCHA**
* Protects forms from automated attacks

### ✅ Rate Limiting

* Session-based throttling
* Prevents spam and brute-force attacks

### ✅ Session Security

* Secure session flags:

  * `HttpOnly`
  * `Secure`
  * `SameSite`
* Session regeneration after login

### ✅ OTP-Based Verification

* Forgot-password OTP system:

  * `forgot_password.php`
  * `verify_otp.php`
  * `reset_password.php`
* OTP expiry enforced using timestamps

### ✅ Security Alerts

* Email notification when login occurs from a **new IP address**

### ✅ SQL Injection Protection

* Prepared statements used throughout the project
* No raw SQL execution

---

## 🧰 Tech Stack

| Layer     | Technology                      |
| --------- | ------------------------------- |
| Backend   | PHP                             |
| Database  | MySQL / MariaDB                 |
| Frontend  | HTML, CSS, Tailwind, JavaScript |
| Email     | PHPMailer                       |
| CAPTCHA   | Cloudflare Turnstile            |
| WebSocket | Ratchet (PHP)                   |
| AI Module | Flask + Groq API                |

---

## 📁 Project Structure

```
/admin          → Admin dashboard  
/group_admin    → Group admin controls  
/group_member   → Member features  
/includes       → Shared layouts and configs  
/ai_tips        → AI-powered finance tips  
server.php      → WebSocket chat server  
cholosave_db.sql → Database schema  
```

---

## ⚙️ Setup Instructions (XAMPP)

### 1️⃣ Requirements

* XAMPP / WAMP / LAMP
* PHP 8+
* Composer
* MySQL

---

### 2️⃣ Place Project

Copy project folder into:

```
C:\xampp\htdocs\CholoSave-CS
```

or (Linux):

```
/var/www/html/CholoSave-CS
```

---

### 3️⃣ Create Database

1. Open **phpMyAdmin**
2. Create database:

```
cholosave_cs
```

3. Import:

```
cholosave_db.sql
```

---

### 4️⃣ Configure Database

Edit `db.php`:

```php
$host = "localhost";
$username = "root";
$password = "";
$dbname = "cholosave_cs";
```

---

### 5️⃣ Install Dependencies

```bash
composer install
```

(Repeat inside `group_admin/` and `group_member/` if needed.)

---

## 🔐 Security Setup (IMPORTANT)

### ✅ Remove Hardcoded Secrets

❌ DO NOT push these:

* Gmail passwords
* API keys
* CAPTCHA secrets

### ✅ Use `.env` File

Create `.env.example`:

```env
DB_HOST=localhost
DB_NAME=cholosave_cs
DB_USER=root
DB_PASS=

SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password

TURNSTILE_SITE_KEY=your_site_key
TURNSTILE_SECRET_KEY=your_secret_key

GROQ_API_KEY=your_api_key
```

Add this to `.gitignore`:

```
.env
```

---

## ▶️ Running the Project

### Start Server

1. Open XAMPP
2. Start **Apache** and **MySQL**
3. Visit:

```
http://localhost/CholoSave-CS/
```

---

## 💬 WebSocket Chat

Start the WebSocket server:

```bash
php server.php
```

Runs on:

```
ws://localhost:8080
```

---

## 🤖 AI Tips Module (Optional)

```bash
cd ai_tips
pip install -r requirements.txt
set GROQ_API_KEY=your_key_here   # Windows
export GROQ_API_KEY=your_key_here # Linux
python generate_tips.py
```

---

## 🗄️ Extra Database Tables

### Security Logs

```sql
CREATE TABLE security_logs (
  id INT AUTO_INCREMENT PRIMARY KEY,
  event_type VARCHAR(50),
  details TEXT,
  ip_address VARCHAR(45),
  timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### OTP Payments

```sql
CREATE TABLE payment_otps (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT,
  group_id INT,
  otp VARCHAR(10),
  otp_expiry DATETIME,
  amount DECIMAL(10,2),
  transaction_id VARCHAR(100),
  payment_method VARCHAR(50),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## 👤 User Roles

| Role        | Access             |
| ----------- | ------------------ |
| user        | Basic features     |
| group_admin | Group control      |
| admin       | Full system access |

Admin Panel:

```
/admin/admin_dashboard.php
```

---

## ⚠️ Security Disclaimer

This project is for **academic purposes only**.

Before deploying:

* Remove all secrets
* Enable HTTPS
* Add login attempt limits
* Harden server permissions
* Use environment variables
* Audit authorization logic

---

## 👨‍💻 Team

Developed by me and my teammates as a **University Computer Security Project**.

---


---


