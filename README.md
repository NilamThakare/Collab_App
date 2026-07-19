# Collab App

A web-based team collaboration platform built with **Flask + MySQL + Jinja2**, developed during an internship at **Sparklab Solutions**.

---

## Features

- **OTP-based login** — Email OTP verification on every login via Gmail SMTP
- **Single-device enforcement** — A user can only be logged in on one device at a time
- **2-hour session lock** — Users cannot log out before completing a 2-hour session (admin can grant early logout)
- **Global team chat** — Real-time messaging with delete-for-me and delete-for-everyone
- **Admin ↔ User private chat** — One-on-one messaging between admin and any user
- **Task management** — Admin creates and assigns tasks with deadlines; users mark tasks complete
- **File sharing** — Upload and download ZIP files shared across the team
- **Code sharing** — Share code snippets with download and delete options
- **Admin dashboard** — View all users, login/logout history, and grant logout permissions
- **Profile management** — Edit username, upload profile picture, delete account
- **Dark theme** — Consistent dark UI across all pages

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python 3, Flask |
| Database | MySQL |
| Frontend | Jinja2 templates, HTML, CSS |
| Auth | OTP via Gmail SMTP, session-based |
| Environment | python-dotenv |
| Password hashing | Werkzeug |

---

## Project Structure

```
Collab_App/
├── app.py                  # Main Flask application (38 routes)
├── requirements.txt        # Python dependencies
├── .env.example            # Environment variable template
├── .gitignore
├── static/
│   ├── logo.png
│   └── profile_pics/       # User profile pictures
├── templates/              # 18 Jinja2 HTML templates
│   ├── login.html
│   ├── reg.html
│   ├── verify_otp.html
│   ├── dashboard.html
│   ├── chat.html
│   ├── admin.html
│   ├── admin_tasks.html
│   ├── admin_private_chat.html
│   ├── user_admin_chat.html
│   ├── user_tasks.html
│   ├── view_tasks.html
│   ├── task_details.html
│   ├── profile.html
│   ├── edit_profile.html
│   ├── logout_blocked.html
│   ├── about.html
│   └── contact.html
└── uploads/                # Shared ZIP files (gitignored)
```

---

## Setup & Installation

### 1. Clone the repository

```bash
git clone https://github.com/NilamThakare/Collab_App.git
cd Collab_App
```

### 2. Create a virtual environment

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# Mac/Linux
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

```bash
cp .env.example .env
```

Edit `.env` with your real values:

```env
FLASK_SECRET_KEY=your_long_random_secret_key
REGISTRATION_SECRET=your_registration_gate_key

DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=your_mysql_password
DB_NAME=collab_app

MAIL_USERNAME=your_email@gmail.com
MAIL_APP_PASSWORD=your_16_char_gmail_app_password
```

> To generate a Gmail App Password: [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)  
> 2-Step Verification must be enabled on your Google account.

### 5. Set up the MySQL database

Create the database in MySQL Workbench or terminal:

```sql
CREATE DATABASE collab_app;
USE collab_app;
```

Then create the required tables:

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(80) NOT NULL,
    email VARCHAR(120) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(20) DEFAULT 'user',
    profile_pic VARCHAR(255),
    otp VARCHAR(6),
    is_logged_in TINYINT(1) DEFAULT 0,
    can_logout TINYINT(1) DEFAULT 0
);

CREATE TABLE message (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    message TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE TABLE deleted_messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    message_id INT,
    UNIQUE KEY unique_delete (user_id, message_id)
);

CREATE TABLE admin_chats (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    admin_id INT,
    message TEXT,
    sender VARCHAR(10),
    time DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    task_title VARCHAR(200),
    task_description TEXT,
    assigned_to VARCHAR(80),
    deadline DATETIME,
    created_by VARCHAR(80),
    status VARCHAR(20) DEFAULT 'Pending',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE shared_files (
    id INT AUTO_INCREMENT PRIMARY KEY,
    filename VARCHAR(255),
    size INT,
    uploaded_by VARCHAR(80),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE codes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(80),
    code TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE deleted_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    item_type VARCHAR(20),
    item_id INT,
    UNIQUE KEY unique_item (user_id, item_type, item_id)
);

CREATE TABLE login_logs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    login_time DATETIME,
    logout_time DATETIME
);

CREATE TABLE contact_messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(80),
    email VARCHAR(120),
    msg TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 6. Create the first admin user

Register normally through the app, then run this SQL to promote your account:

```sql
UPDATE users SET role='admin' WHERE email='your_email@example.com';
```

### 7. Run the app

```bash
python app.py
```

Visit `http://localhost:5000`

---

## Environment Variables

| Variable | Description |
|---|---|
| `FLASK_SECRET_KEY` | Secret key for Flask session signing |
| `REGISTRATION_SECRET` | Gate key required to register a new account |
| `DB_HOST` | MySQL host (default: localhost) |
| `DB_PORT` | MySQL port (default: 3306) |
| `DB_USER` | MySQL username |
| `DB_PASSWORD` | MySQL password |
| `DB_NAME` | MySQL database name |
| `MAIL_USERNAME` | Gmail address used to send OTPs |
| `MAIL_APP_PASSWORD` | Gmail App Password (not your real password) |

---

## Security Notes

- All secrets are loaded from `.env` — never hardcoded in source
- `.env` is gitignored and never pushed to GitHub
- Passwords are hashed using `werkzeug.security.generate_password_hash`
- Single-device login is enforced via `is_logged_in` flag in the database
- Every route opens its own DB connection and closes it after use (no shared global cursor)

---

## Internship

Built during a web development internship at **Sparklab Solutions**, Armavati.

---

## License

This project is for educational and portfolio purposes.

---<img width="1600" height="728" alt="teamchat" src="https://github.com/user-attachments/assets/fba19ce6-4c52-4159-855c-060d1c8422b0" />
<img width="1600" height="745" alt="userdashboard" src="https://github.com/user-attachments/assets/7e945192-4233-4805-a807-c948a7350d1a" />
<img width="1600" height="729" alt="tasks" src="https://github.com/user-attachments/assets/3b826e63-6a6c-4b1a-a3ee-fd21d42b4e8b" />
<img width="1600" height="732" alt="adminuserchat" src="https://github.com/user-attachments/assets/24093815-cf7f-486b-abe9-5dc0c106b60d" />
<img width="1600" height="745" alt="admindashboard" src="https://github.com/user-attachments/assets/8ca8d443-7002-4e24-a763-7c105e09aa0c" />
<img width="1600" height="746" alt="loginCollabapp" src="https://github.com/user-attachments/assets/7bf5695b-c3fd-413f-b4e0-9db07dbdc3ef" />


##Author
Nilam Thakare

GitHub:https://github.com/NilamThakare
LinkedIn: https://linkedin.com/in/nilam-t
