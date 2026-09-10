# BioGold Transition Platform

A JavaScript full-stack prototype for low-connectivity artisanal and small-scale gold mining formalisation work. It preserves the original Figma Make prototype's forest-green, gold, cream, brown and muted-blue visual language while adding a separated REST backend, seedable JSON data, role-aware navigation, action plans, training, structured MFARI scoring, scenario-specific calculator inputs, and offline storage.

## Structure

```text
frontend/   React + Vite + JSX application
backend/    Node + Express REST API and JSON repository
```

There are no TypeScript files in the application. The JSON files in `backend/data` are development data only and must be replaced or protected before production use.

## Requirements

- Node.js 20 or newer
- npm

## Run locally

Terminal 1:

```powershell
cd backend
npm install
npm run seed
npm run dev
```

Terminal 2:

```powershell
cd frontend
npm install
npm run dev
```

Open `http://localhost:5173`. The API runs at `http://localhost:4000`.

The seed command creates 24 fictional participants, raw 25-question responses, calculated assessments, barriers, action plans, training records, scenarios and referrals. It is safe to rerun in a development environment because it rebuilds the JSON collections.

## Demo accounts

The prototype login endpoint accepts any of these fictional email addresses. It returns a signed development JWT; no production credentials are included.

- `admin.demo@example.local` — administrator
- `coordinator.demo@example.local` — coordinator
- `facilitator.demo@example.local` — facilitator
- `participant.demo@example.local` — participant

## Main API endpoints

- `GET /api/health`
- `POST /api/auth/login`
- `GET /api/mfari/questions`
- `POST /api/mfari/assessments`
- `GET /api/mfari/assessments/:id/results`
- `GET|POST|PATCH /api/action-plans`
- `GET /api/training`
- `POST /api/training/:id/progress`
- `POST /api/calculator/calculate`
- `GET /api/dashboard/summary`
- `GET /api/referrals`
- `GET|POST /api/safety-beacon/sessions`

Protected endpoints require `Authorization: Bearer <token>`. Aggregate dashboard output contains derived counts and distributions, not individual participant records.

## Offline architecture

The frontend registers `frontend/public/sw.js` to cache the application shell. Structured records and pending submissions use IndexedDB through `frontend/src/services/offlineStore.js`, with a localStorage fallback for browsers that do not provide IndexedDB. The connection indicator updates from browser online/offline events. Failed assessment submissions are stored locally and placed in a synchronization queue.

The current prototype provides the queue and storage boundary; a production sync worker still needs retry policies, conflict resolution rules, authentication refresh and server acknowledgement handling. The Safety Beacon screen/API is explicitly a demo/manual session log and does not detect mercury.

## Design and domain boundaries

- MFARI questions are configuration data in `frontend/src/data/mfariQuestions.js` and mirrored by the backend scoring service.
- Scoring, recommendations, calculator calculations and dashboard aggregation are reusable backend services.
- Calculator current and alternative scenarios have separate labour, equipment, maintenance, consumable, fuel and recovery assumptions.
- Negative profit/loss values remain visibly negative.
- The calculator disclaimer is always shown.
- Action plans use `not_started`, `in_progress`, `completed` and `cancelled` statuses.
- The dashboard uses backend aggregation when available and labels its fallback as demo data when the API is unavailable.

## Security and production roadmap

The backend includes Helmet, CORS configuration, rate limiting, request checks, JWT role middleware and structured error responses. Set `JWT_SECRET`, `CORS_ORIGIN`, `PORT` and `DATABASE_PATH` through an environment file based on `backend/.env.example`; never put real secrets in frontend code.

The JSON repository is intentionally isolated behind `backend/src/repositories/jsonRepository.js`, so it can later be replaced by SQLite, PostgreSQL or Supabase without moving calculation or route logic into UI components. Before production deployment, add a real identity provider, password policy or external authentication, durable database transactions, audit logging, encrypted transport/storage, formal consent/deletion workflows, automated tests and a complete sync conflict model.
