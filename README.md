# AutoCare Smart Vehicle Service Platform

## Run the application

Start the backend first, then the frontend. The backend requires a configured PostgreSQL database and a bootstrap administrator account; see [`../backend/README.md`](../backend/README.md).

```powershell
cd D:\Project\frontend
npm install
Copy-Item .env.example .env.local
npm run dev
```

By default, Vite runs at `http://localhost:5173` and the API at `http://localhost:8080/api`. Set `VITE_API_BASE_URL` in `.env.local` if the API is hosted elsewhere.

## API and persistence

The authenticated API is the source of truth for vehicles, bookings and mechanic assignments, inventory and stock transactions, invoices and payments, maintenance reminders, workshop settings, and preferences. The frontend loads the workspace after server login and writes changes with an `If-Match` revision. Concurrent stale updates receive a conflict instead of replacing newer server state.

API endpoints added:

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/api/auth/csrf` | Obtain the CSRF token for session-authenticated requests |
| `POST` | `/api/auth/register` | Create a Customer account |
| `POST` | `/api/auth/login` | Verify email/password and establish a server session |
| `GET` | `/api/auth/me` | Return the authenticated server identity and role |
| `POST` | `/api/auth/logout` | Invalidate the server session |
| `POST` | `/api/auth/password` | Verify and change the account password |
| `GET` | `/api/workspace` | Load the authenticated workspace |
| `PUT` | `/api/workspace` | Atomically save workspace changes with an `If-Match` revision |

The workspace API preserves the current frontend record shapes. Each vehicle, booking, mechanic, inventory item, stock transaction, invoice, payment, and reminder is stored as an individual JSONB row linked to its workspace. The service validates record IDs, booking/mechanic assignments, stock transaction deltas, invoice calculations, payment balances, and reminder-to-vehicle references before a transaction commits. Workshop settings and preferences are stored as workspace records. This is an initial compatibility-oriented schema; domain-specific columns and standalone CRUD endpoints are not yet exposed.

Existing localStorage data is deliberately not loaded or uploaded. It remains at `autocare-app-data:v1`; use **Settings → Export legacy browser data** to download it. Review and reconcile the export before any manual import. Authentication secrets are never written to localStorage; the API uses an HttpOnly session cookie and a CSRF token.

Create, update, and related record changes are applied to the UI only after the workspace API confirms the write. A successful response includes the persisted workspace and next revision; refreshing loads it from `GET /api/workspace`.

## Authentication

The development role selector is removed. Roles come from the server and backend authorization protects workspace endpoints. Public registration creates a `Customer` account; elevated roles are not accepted from the client. Bootstrap the first `Admin` using `BOOTSTRAP_ADMIN_EMAIL` and `BOOTSTRAP_ADMIN_PASSWORD`; the password must be at least 12 characters. These variables are only used to create the account if that email does not exist. There are no demo accounts or automatic sign-ins.

## Frontend checks

```powershell
npm test
npm run lint
npm run build
```
