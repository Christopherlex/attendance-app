# AttendTrack — GPS Attendance Web App

A full-stack Flask attendance management system with GPS check-in/check-out, admin dashboard, and Excel export. Ready to deploy on Hostinger.

---

## Features

| Feature | Detail |
|---|---|
| **Login** | Role-based redirect (admin or worker) |
| **Worker dashboard** | Check-in / check-out with GPS capture |
| **GPS** | Browser Geolocation API, high accuracy, graceful fallback |
| **Admin dashboard** | All records table, date filter, status badges |
| **GPS links** | Clickable Google Maps links per record |
| **Excel export** | XLSX download with color-formatted headers |
| **Security** | Parameterized queries, session-based auth |
| **Mobile-friendly** | Responsive design for phones and tablets |

---

## Project Structure

```
attendance_app/
├── app.py                 # Main Flask app
├── passenger_wsgi.py      # Hostinger WSGI entry point
├── .htaccess              # Apache/Passenger config for Hostinger
├── requirements.txt       # Python dependencies
├── .env.example           # Environment variable template
└── templates/
    ├── login.html         # Login page
    ├── worker.html        # Worker dashboard
    └── admin.html         # Admin dashboard
```

---

## Database Setup (Neon PostgreSQL)

### 1. Create tables

Visit this URL once after deploying (replace `yourtoken`):

```
https://yourdomain.com/setup?token=yourtoken
```

Make sure `SETUP_TOKEN=yourtoken` matches your `.env`.

### 2. Manually create tables (alternative)

Run in your Neon SQL editor:

```sql
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(80) UNIQUE NOT NULL,
    password TEXT NOT NULL,
    full_name TEXT NOT NULL,
    role VARCHAR(10) NOT NULL CHECK (role IN ('admin','worker'))
);

CREATE TABLE IF NOT EXISTS attendance (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    checkin TIMESTAMP NOT NULL,
    checkout TIMESTAMP,
    location_name TEXT NOT NULL,
    checkin_lat FLOAT,
    checkin_lng FLOAT,
    checkout_lat FLOAT,
    checkout_lng FLOAT,
    ip_address TEXT,
    user_agent TEXT
);
```

### 3. Seed users

```sql
-- Admin
INSERT INTO users (username, password, full_name, role)
VALUES ('admin', 'adminpass', 'Site Administrator', 'admin');

-- Worker
INSERT INTO users (username, password, full_name, role)
VALUES ('worker1', 'worker123', 'John Smith', 'worker');
```

---

## Local Development

```bash
# 1. Clone / copy files
cd attendance_app

# 2. Create virtual environment
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment
cp .env.example .env
# Edit .env with your DATABASE_URL and SECRET_KEY

# 5. Run
flask run
# or
python app.py
```

---

## Deploy on Hostinger

### Prerequisites
- Hostinger VPS or Business/Cloud hosting plan with Python support
- A subdomain pointed to your hosting

### Steps

1. **Upload files** via FTP or Hostinger File Manager to your subdomain's public folder (e.g. `public_html/attend.yourdomain.com/`).

2. **Install Python dependencies** via SSH:
   ```bash
   cd ~/public_html/attend.yourdomain.com
   pip install -r requirements.txt --user
   ```

3. **Create `.env`** in the same folder:
   ```
   DATABASE_URL=postgresql://...your_neon_url...?sslmode=require
   SECRET_KEY=your-long-random-secret
   SETUP_TOKEN=your-setup-token
   ```

4. **Set up tables** by visiting:
   ```
   https://attend.yourdomain.com/setup?token=your-setup-token
   ```
   You should see: `Tables created.`

5. **Seed your first admin user** via Neon SQL editor (see above).

6. **Visit** `https://attend.yourdomain.com` — login page should appear.

### Hostinger Python App Configuration

In Hostinger hPanel → **Websites** → **Manage** → **Advanced** → **Python**:

- **Python version**: 3.10+
- **Application root**: `/home/user/public_html/attend.yourdomain.com`
- **Application URL**: your subdomain
- **Application startup file**: `passenger_wsgi.py`
- **Application entry point**: `application`

---

## Environment Variables

| Variable | Description | Required |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection URL | Yes |
| `SECRET_KEY` | Flask session encryption key | Yes |
| `SETUP_TOKEN` | Token to access `/setup` route | Yes |

---

## API Endpoints

| Method | URL | Description |
|---|---|---|
| GET/POST | `/` | Login |
| GET | `/logout` | Logout |
| GET | `/worker` | Worker dashboard |
| POST | `/api/checkin` | JSON: `{location_name, lat, lng}` |
| POST | `/api/checkout` | JSON: `{lat, lng}` |
| GET | `/api/status` | Current session status |
| GET | `/admin` | Admin dashboard (`?date=YYYY-MM-DD`) |
| GET | `/admin/export` | Download XLSX (`?date=YYYY-MM-DD`) |
| GET | `/setup` | Create tables (`?token=...`) |

---

## Security Notes

- Passwords are stored plain text as per requirements. For production, consider hashing with `bcrypt`.
- The `/setup` route is protected by `SETUP_TOKEN`. Remove it or restrict access after initial setup.
- Always use HTTPS in production.
- `SECRET_KEY` must be a long, random string (32+ characters).

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `psycopg2` import error | `pip install psycopg2-binary` |
| Tables not found | Visit `/setup?token=...` |
| GPS not working | Requires HTTPS; works on mobile browsers |
| 500 error on Hostinger | Check `.env` exists and `DATABASE_URL` is correct |
| Session not persisting | Ensure `SECRET_KEY` is consistent across restarts |
