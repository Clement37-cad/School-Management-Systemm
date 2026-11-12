# School-Management-Systemm
USTED ERP — PHP Edition
A PHP + MySQL rewrite of the original PyQt5 desktop application, for
University of Skills Training and Entrepreneurial Development (USTED).
What changed from the desktop version
PyQt5 → PHP + HTML/CSS, runs in any browser, served by Apache/Nginx/PHP's built-in server.
SQLite → MySQL, designed to be created and inspected in MySQL Workbench.
The drawn crest widget was replaced with the official USTED logo image (`assets/logo.png`),
used on the login screen, sidebar, and printable report.
CSV export and a printable "PDF" summary report (browser print-to-PDF) replace the desktop
ReportLab export — no extra PHP libraries required to get started.
1. Set up the database in MySQL Workbench
Open MySQL Workbench and connect to your local MySQL server.
Open `schema.sql` (File → Open SQL Script) and run it (the lightning bolt icon).
This creates the `usted_erp` database, all 10 tables, and one seed admin account.
(Optional) Reverse-engineer the ERD directly in Workbench:
Database → Reverse Engineer… → select the `usted_erp` schema → follow the
wizard to get an editable EER diagram inside Workbench itself.
A ready-made ERD image is also included: `usted_erd.png`.
2. Configure the PHP app
Edit `config/db.php` and match it to your MySQL Workbench connection:
```php
define('DB_HOST', '127.0.0.1');
define('DB_PORT', '3306');
define('DB_NAME', 'usted_erp');
define('DB_USER', 'root');
define('DB_PASS', 'your_password');
```
3. Run it
With PHP's built-in server (fastest way to try it locally):
```bash
cd usted_php
php -S localhost:8000
```
Then open `http://localhost:8000/login.php`.
Or drop the `usted_php` folder into your Apache/Nginx web root (e.g. XAMPP's `htdocs`,
or `/var/www/html`) and browse to it there.
Default login: `admin` / `admin123`
Folder structure
```
usted_php/
├── schema.sql              # Run this in MySQL Workbench first
├── usted_erd.png            # Entity-relationship diagram
├── config/
│   ├── db.php               # MySQL connection (edit this)
│   └── app.php               # Shared constants, session, helpers
├── includes/
│   ├── header.php           # Sidebar + topbar (uses assets/logo.png)
│   └── footer.php
├── assets/
│   ├── style.css
│   └── logo.png              # Official USTED crest
├── login.php / logout.php
├── index.php                 # Analytics dashboard (Chart.js)
├── students.php              # Student management (CRUD)
├── registration.php          # Course registration
├── gpa.php                    # GPA calculator
├── financial.php             # Invoices & payments
├── staff_attendance.php      # Staff directory & attendance
└── reports.php                # CSV export + printable PDF report
```
Notes
Passwords are hashed with SHA-256 to match the original scheme (`SHA2(..., 256)` in MySQL).
For production use, consider migrating to PHP's `password_hash()` (bcrypt/argon2) instead.
All queries use PDO prepared statements to prevent SQL injection.
Default passwords for records created through the UI: students → `student123`, staff → `staff123`.
---
Update: Student Portal + Active Email
Upgrade an existing database
Run `migration_portal.sql` once in MySQL Workbench (new installs: `schema.sql` already has everything).
It adds email verification columns, reset tokens, the portal tables, and a demo student:
Index number `UST/0001` / password `student123`.
Turn on real email
Edit `config/mail.php`. For Gmail: enable 2-Step Verification, create an App password, then set
`MAIL_USERNAME` (your Gmail) and `MAIL_PASSWORD` (the 16-character app password).
Until you do, nothing is sent: every email is written to `storage/mail.log` so you can test the flows.
Set `APP_BASE_URL` when deployed so links in emails point to your real domain.
Emails sent: welcome + verify (new student), verification (changed email), password reset,
password-changed alert, payment receipt, registration confirmation, and acknowledgement/updates
for resits-ICs, internships, complaints and medicals.
Student portal (`/portal`)
Students log in at `login.php` with their index number and land in the portal, laid out like the
screenshots: side menu (Home, Registration, Results, Fees, Resits and ICs, Internship, Complaint,
Medicals, Evaluation) and a Home profile with Personal / Programme tabs (plus Account: email, phone,
Telegram, change password). Staff manage requests at Student Requests in the admin sidebar; saving
a reply emails the student.
Forgot password
Email works as soon as SMTP is set. Telegram needs `TELEGRAM_BOT_TOKEN` (the student must put their chat ID
in Account). SMS needs `SMS_API_KEY` (Arkesel-style API; adapt `send_sms()` for another provider).
Security notes
Student accounts can no longer open admin pages. Forms use CSRF tokens; reset links are single-use,
hashed in the database and expire in 60 minutes. Fee payment in the portal is still simulated.
Automated messages (SMS / Telegram / Email)
Admin > Automated Messages: edit the text templates (placeholders like `{name}`, `{amount}`), choose channels per template,
send announcements to all students / a level / a programme, and view the message log.
Sent automatically on: payment, registration submit, request updates, and fee reminders.
Daily reminders: schedule `php cron_reminders.php` in Windows Task Scheduler (exact command shown on the page).
Students receive SMS only if a phone is saved in their Account tab; set the SMS key under Email Settings.
Python automation (`automation/`)
`automation/auto_messenger.py` creates and sends the automated texts from Python. It reads the same MySQL database,
templates, log and Email Settings as the PHP app (DB login is read from `config/db.php`, no duplicate settings).
    cd automation
    pip install -r requirements.txt
    python auto_messenger.py preview                     # who would be reminded (sends nothing)
    python auto_messenger.py reminders                   # send fee reminders once
    python auto_messenger.py run --every-hours 24        # keep running on a schedule
    python auto_messenger.py announce "Lectures resume Monday" --level "Level 100"
    python auto_messenger.py test-sms 0240000000

Use this instead of `cron_reminders.php` in Windows Task Scheduler (`python.exe ...\auto_messenger.py reminders`, daily).
