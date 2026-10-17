# THE HULK

**Move smarter. Go further.** THE HULK is a responsive fitness-tracking app with a custom H-mark logo, real browser GPS recording, workout analysis and a separate API backend.

## Project layout

- `frontend/` — static HTML, CSS, JavaScript and the original `logo.svg`.
- `backend/` — dependency-free Node.js HTTP API and file-backed data store.
- `backend/data/` — created at runtime; contains accounts and GPS activities and is excluded from version control.

## Run the full app

Start the built-in SQLite backend from the project root:

```powershell
py .\backend\server.py
```

Then open <http://127.0.0.1:8000>. The Python server serves the frontend and API from the same origin and creates `backend/data/the_hulk.sqlite3` on first run. Python 3.9 or newer is required; there are no third-party package dependencies.

The site opens to `frontend/login.html`. Sign in with your username and password. Account creation is a separate page at `frontend/signup.html` and asks for your name, a unique username, email and password. Usernames are 3–24 letters, numbers, periods, underscores or hyphens. The separate page links navigate between sign-in and account creation. After authenticating, users arrive at `frontend/index.html`. A guest can choose **Explore as a guest**. Passwords are salted and hashed with `scrypt`; bearer tokens are randomly generated and kept in the browser session. Signed-in profiles and saved activities are backed up to the backend. Local browser-only activities are uploaded when a user explicitly signs in. Existing accounts may continue to sign in using their username or email.

Authentication requires the backend to be running.

### Optional SMS sign-in

Phone sign-in uses Twilio Verify and is disabled until the Python backend has credentials. Create a Twilio Verify service, then set these variables in the same PowerShell session before starting the server:

```powershell
$env:TWILIO_ACCOUNT_SID = "your-account-sid"
$env:TWILIO_AUTH_TOKEN = "your-auth-token"
$env:TWILIO_VERIFY_SERVICE_SID = "your-verify-service-sid"
py .\backend\server.py
```

Keep these credentials on the backend; do not add them to frontend files or commit them. Users sign in with SMS only after linking and verifying their phone number in **Settings → Phone sign-in**. Enter numbers in E.164 format (for example, `+14155552671`). The Python API sends codes and issues a login session only after Twilio approves a code. Without credentials, the API reports that SMS is not configured; it does not claim that a message was sent. The optional Node.js backend does not implement SMS endpoints; use the Python backend for phone sign-in.

GPS recording requires a secure browser context (`localhost` qualifies), the user's permission and an available GPS signal. Map tiles and Google Fonts need an internet connection. OpenStreetMap tiles are attributed in the map.

## Frontend-only preview

Alternatively, the separate Node.js backend can serve the same frontend and API:

```powershell
node .\backend\server.js
```

Then visit <http://127.0.0.1:3000>. Its JSON data store is separate from the Python server's SQLite database. Do not run both backends on the same port.

## API

- `GET /api/health`
- `POST /api/auth/signup`, `POST /api/auth/login`, `POST /api/auth/logout`
- `POST /api/auth/phone/send`, `POST /api/auth/phone/verify` (Python backend; Twilio Verify required)
- `GET /api/me`, `PATCH /api/me`
- `GET /api/activities`, `POST /api/activities`
- `PATCH /api/activities/:id`, `DELETE /api/activities/:id`
- `DELETE /api/account`

The API checks that each activity and profile change belongs to the authenticated account, validates request fields, limits sign-in attempts and request body size, and stores passwords as salted scrypt hashes. Social interactions such as follows, clubs, kudos and comments remain local prototype features; no community data is falsely shown as live server data.

## Production notes

The included backends are for local development, not production deployment. Before exposing either to the internet, add HTTPS, durable and revocable session management, email verification/password recovery, distributed rate limiting, backups, monitoring and account-data retention controls, and deploy behind a production-ready database/auth service. Configure a production map tile provider and usage limits if needed. The AI coach currently derives suggestions from logged workout values in the browser; it does not call an AI service. GPS pace, elevation and distance can be estimates affected by device accuracy, signal quality and pauses; no heart-rate or cadence data is invented. THE HULK uses an original geometric strength mark, not an illustration from the Marvel character.
