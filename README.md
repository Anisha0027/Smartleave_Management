# SmartLeave – AI-Powered Leave Management System

**“Simplifying Leave. Smarter Decisions. Better Management.”**

SmartLeave is a responsive college leave-management mini project for students, faculty, and administrators. It combines a polished React dashboard with a FastAPI/SQLite REST API, JWT role-based authorization, notifications, reports, and lightweight local machine learning.

## What is included

- **Student:** dashboard, balances, application workflow, automatic duration and overlap checks, searchable history, profile/preferences, notifications, and a local app-help assistant.
- **Faculty:** department requests, approval/rejection workflow, mandatory rejection remarks, student records, and statistics.
- **Admin:** institution dashboard, users, policy configuration, analytics, filters, CSV export, and printable reports.
- **Security:** bcrypt password hashing, JWT access tokens, API role guards, Pydantic validation, CORS, upload type/size checks, and environment-based secrets.
- **AI:** trained scikit-learn TF-IDF + Logistic Regression reason classifier, history-based leave insight, deterministic overlap detection, and a predefined-intent assistant that needs no external API.
- **Languages:** English, Tamil, and Hindi translation JSON with a persistent language preference.
- **Development fallback:** the frontend opens in a complete local demo mode when `VITE_API_URL` is not set. Configure the API URL to authenticate against FastAPI.

## Structure

```text
.
├── src/
│   ├── components/       # Shared UI and SmartLeave Assistant
│   ├── context/          # Demo session and language state
│   ├── layouts/          # Responsive role-aware app shell
│   ├── pages/            # Login, dashboards, leave and management screens
│   ├── services/         # Axios client and token interceptor
│   └── translations/     # en.json, ta.json, hi.json
├── backend/
│   ├── app/
│   │   ├── ai_service.py # TF-IDF classifier
│   │   ├── database.py   # Portable SQLAlchemy setup
│   │   ├── models.py     # Relational data model
│   │   ├── schemas.py    # Pydantic request/response validation
│   │   ├── security.py   # JWT, password hashing, role guards
│   │   ├── seed.py       # Safe idempotent demo seed
│   │   └── main.py       # FastAPI REST endpoints
│   ├── main.py           # uvicorn entry point
│   └── requirements.txt
└── README.md
```

## Demo accounts

All local demo accounts use the password `SmartLeave@123`.

| Role | Email |
|---|---|
| Student | `student@example.com` |
| Faculty | `faculty@example.com` |
| Admin | `admin@example.com` |

These credentials are for local demonstration only. Replace them and set a strong `SECRET_KEY` before any deployment.

## Run the frontend

Requirements: Node.js 22 and pnpm (versions are documented in `.mise.toml`).

```bash
pnpm install
pnpm dev
```

In Figma Make, the Vite server is already supervised and should not be started manually. Source changes hot reload in the preview panel.

To connect authentication and API calls to the backend, create a root `.env.local`:

```env
VITE_API_URL=http://localhost:8000/api
```

Without that variable, the UI intentionally uses seeded browser demo data for easy presentation.

## Run the backend

From the repository root:

```bash
cd backend
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
uvicorn main:app --reload
```

- API: `http://localhost:8000`
- Swagger/OpenAPI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

### Database initialization

No separate migration command is required for the demonstration. On first API startup, SmartLeave:

1. Creates `backend/data/smartleave.db`.
2. Creates all tables and foreign keys.
3. Seeds departments, leave types, balances, demo users, a leave request, and a notification.

The seed is idempotent. Delete the local database and restart the API only when you intentionally want a clean demo reset. `DATABASE_URL` can later point to PostgreSQL without changing model code (install the appropriate PostgreSQL driver).

## API overview

| Area | Endpoints |
|---|---|
| Authentication | `POST /api/auth/register`, `/login`, `/logout`, `/change-password` |
| Profile | `GET/PUT /api/users/me` |
| Leave | `POST/GET /api/leaves`, `GET/DELETE /api/leaves/{id}` |
| Review | `PUT /api/leaves/{id}/approve`, `/reject` |
| Balance | `GET /api/leave-balance` |
| Notifications | `GET /api/notifications`, `PUT /api/notifications/{id}/read` |
| AI | `POST /api/ai/classify-reason`, `GET /api/ai/leave-insight` |
| Admin | `GET /api/admin/users`, `/analytics`, `/reports` |
| Upload | `POST /api/uploads` (PDF/JPG/PNG, maximum 5 MB) |

Email delivery is deliberately non-failing in development. In-app notifications are always created; `EMAIL_ENABLED` is reserved for a future provider integration and no API keys are hardcoded.

## Test the main flows

1. Open the frontend; it starts in the student demo.
2. Review dashboard charts and balances.
3. Open **Apply Leave**, enter dates and a reason such as “Doctor appointment for fever.” The AI category changes to Medical.
4. Try dates overlapping an existing request to see duplicate prevention.
5. Submit a valid request and inspect the confirmation state.
6. Use the role switcher in the top bar to enter Faculty mode.
7. Open **Leave Requests**, inspect a request, approve it, or reject it with a required remark.
8. Switch to Admin to view analytics, users, leave policies, CSV export, and print output.
9. Change language in the top bar and refresh to verify persistence.
10. Ask the SmartLeave Assistant about balances, request status, leave types, or cancellation.

For backend API testing, sign in through Swagger with a demo account, copy the returned token, click **Authorize**, and use `Bearer <token>`.

## Verification

```bash
pnpm build
python -m compileall -q backend
```

The frontend production build and backend syntax compilation are expected to pass. The generated frontend bundle may report a non-blocking large-chunk warning because Recharts is included; route-level lazy loading is a straightforward future optimization.

## Current demonstration boundaries

- Attachments are securely validated but returned in development storage mode rather than permanently stored.
- Email is represented by reliable in-app notifications until a provider is configured.
- The frontend supports complete offline demo interaction. Persistent browser-to-API mutations can be expanded using the provided Axios client; all corresponding backend endpoints are implemented.
- PDF generation is omitted by design; CSV and print-friendly reports are included.
