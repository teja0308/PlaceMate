# PlaceMate

> A role-based campus placement platform that brings students, recruiters, placement
> coordinators (SPOCs), and administrators into one workflow.

PlaceMate is a full-stack placement management system. Students can discover and apply
for jobs, recruiters can publish openings and manage candidates, SPOCs can coordinate
job discussions and student groups, and administrators can verify companies, manage
users, and send targeted notifications.

## What PlaceMate provides

### For students

- Complete a profile with academic details, skills, preferred job type, and backlogs.
- Upload and manage a resume.
- Discover eligible job opportunities and submit applications.
- Track application status, upcoming deadlines, and notifications.
- Sign in with Google, LinkedIn, or the platform authentication flow.

### For recruiters

- Complete recruiter onboarding and company KYC verification.
- Create, edit, and manage job postings.
- Upload job-description and verification documents.
- Review applicants and communicate with the assigned SPOC.
- View recruiter-specific jobs, candidates, messages, and notifications.

### For SPOCs

- View and manage assigned jobs.
- Coordinate discussions between recruiters and the placement team.
- Track job status, changes, and message counts.
- Organize student groups and manage job-related notifications.

### For administrators

- View dashboard statistics and manage students.
- Create and verify companies.
- Review recruiter KYC submissions.
- Manage SPOC assignments.
- Review jobs and update application statuses in bulk.
- Send notifications to specific roles or upload recipient lists.

## Highlights

- **Role-based experience:** Student, recruiter, SPOC, and administrator workflows.
- **Secure sessions:** Keycloak-backed authentication with access and refresh tokens
  stored through HTTP-only cookies.
- **Placement workflow:** Jobs, eligibility, applications, statuses, deadlines, and
  recruiter/SPOC coordination.
- **Document handling:** Resume and document uploads with Cloudinary support, plus
  local upload support for development.
- **Resume parsing:** Optional Hugging Face-powered extraction of candidate details from
  PDF resumes.
- **Notifications and messaging:** Role-targeted notifications and conversations around
  assigned jobs.
- **Scheduled updates:** The backend periodically updates job statuses based on dates.
- **PostgreSQL data model:** Users, jobs, applications, notifications, alumni stories,
  SPOC assignments, and recruiter KYC records.

## Architecture

```text
Placemate/
├── frontend/                 React 19 + Vite client
│   ├── src/components/       Shared UI, auth, recruiter, SPOC components
│   ├── src/pages/            Student, recruiter, SPOC, and admin screens
│   ├── src/context/          Auth and application context
│   ├── src/store/             Redux store and slices
│   └── src/api/               Axios API client
├── backend/                  Express 5 API
│   └── src/
│       ├── controllers/       HTTP request handlers
│       ├── services/          Business logic
│       ├── repo/              PostgreSQL data access
│       ├── db/                Schema and migrations
│       ├── middleware/        Authentication middleware
│       └── utils/             Cloudinary and job-status scheduler
├── ai/                       Optional Flask resume parser
├── readme files/             Feature and migration notes
├── docker-compose.yml        Local Keycloak service
└── sample_students.*         Example bulk-import files
```

### Request flow

```text
Browser
  │
  ▼
React/Vite frontend ── credentials ──► Express API (:4000)
                                          │
                         ┌────────────────┼────────────────┐
                         ▼                ▼                ▼
                    PostgreSQL        Keycloak         Cloudinary
                  users/jobs/etc.   authentication    resumes/docs
                                          │
                                          ▼
                              Optional Flask parser (:7001)
```

## Technology stack

| Area | Technologies |
| --- | --- |
| Frontend | React, React Router, Vite, Redux Toolkit, Axios |
| UI | Tailwind CSS, Mantine, Radix UI, Lucide, Recharts |
| Backend | Node.js, Express, Multer, CORS, cookie-parser |
| Authentication | Keycloak, JWT, Google OAuth, LinkedIn OAuth |
| Database | PostgreSQL with `pg` and SQL migrations |
| File storage | Cloudinary and local uploads |
| Resume parsing | Flask, LangChain PDF loader, Hugging Face Inference API |
| Scheduling | `node-cron` |

## Prerequisites

- Node.js 18+ and npm
- Python 3.9+ (only if the optional resume parser is used)
- PostgreSQL database (the backend is currently configured for a PostgreSQL
  connection string)
- Keycloak 24.0 or a compatible Keycloak instance
- Cloudinary account for cloud document storage
- Google/LinkedIn OAuth credentials if those sign-in providers are enabled
- Hugging Face token if AI resume parsing is enabled

## Local setup

The commands below are written for PowerShell on Windows. Run each service in its own
terminal.

### 1. Start Keycloak

From the project root:

```powershell
docker compose up -d keycloak
```

Keycloak is exposed at `http://localhost:8080`. Configure a realm and client, then
use those values in the backend environment file.

### 2. Configure the backend

```powershell
cd backend
Copy-Item "env files" .env
npm install
```

Update `backend\.env` with your own values. At minimum, configure:

```env
NODE_ENV=development
PORT=4000
DATABASE_URL=postgresql://<user>:<password>@<host>:<port>/<database>

KEYCLOAK_BASE_URL=http://localhost:8080
KEYCLOAK_REALM=<your-realm>
KEYCLOAK_CLIENT_ID=<your-client-id>
KEYCLOAK_CLIENT_SECRET=<your-client-secret>

GOOGLE_CLIENT_ID=<your-google-client-id>
GOOGLE_CLIENT_SECRET=<your-google-client-secret>
LINKEDIN_CLIENT_ID=<your-linkedin-client-id>
LINKEDIN_CLIENT_SECRET=<your-linkedin-client-secret>

CLOUDINARY_CLOUD_NAME=<your-cloud-name>
CLOUDINARY_API_KEY=<your-api-key>
CLOUDINARY_API_SECRET=<your-api-secret>

HF_TOKEN=<your-hugging-face-token>
```

Do not commit `.env` files or real credentials. If credentials have ever been exposed,
rotate them before using the project outside a local development environment.

Initialize the database schema and apply migrations:

```powershell
psql $env:DATABASE_URL -f src/db/schema.sql
npm run migrate:all
```

Start the API:

```powershell
npm run dev
```

The API is available at `http://localhost:4000/api`.

### 3. Configure and start the frontend

```powershell
cd ..\frontend
npm install
```

Create `frontend\.env.local`:

```env
VITE_API_URL=http://localhost:4000/api
VITE_GOOGLE_CLIENT_ID=<your-google-client-id>
```

Start the Vite development server:

```powershell
npm run dev
```

Open the URL printed by Vite, normally `http://localhost:5173`.

### 4. Optional: start the resume parser

The Flask service is optional. The backend also contains resume-parsing integration,
but this service can be run independently when the AI parser is needed.

```powershell
cd ..\ai
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
$env:HF_TOKEN = "<your-hugging-face-token>"
python app.py
```

The parser listens on `http://localhost:7001` and accepts PDF uploads at
`POST /parse-resume`.

## Useful commands

### Frontend

```powershell
cd frontend
npm run dev       # Start Vite with hot reload
npm run build     # Create a production build
npm run lint      # Run ESLint
npm run preview   # Preview the production build
```

### Backend

```powershell
cd backend
npm run dev       # Start with nodemon
npm start         # Start the API normally
npm run migrate:all
```

## API overview

All protected endpoints are served below `/api` and require the authenticated access
cookie.

| Group | Examples |
| --- | --- |
| Authentication | `/auth/register`, `/auth/login`, `/auth/refresh`, `/auth/logout`, `/auth/me` |
| Jobs | `/jobs`, `/recruiter/jobs` |
| Applications | `/applications`, `/applications/:jobId`, `/applications/:jobId/apply/:studentId` |
| Profiles | `/profile`, `/profile/resume`, `/profile/onboarding`, `/parse-resume` |
| Companies | `/companies`, `/admin/companies` |
| Notifications | `/notifications`, `/notifications/send-by-role` |
| Administration | `/admin/dashboard/stats`, `/admin/students`, `/admin/spocs` |
| SPOC coordination | `/spoc/assigned-jobs`, `/spoc/assign-job`, `/spoc/.../conversation` |
| Recruiter KYC | `/recruiter/kyc`, `/admin/recruiter-kyc/...` |
| Uploads | `/upload/document` |

For the complete route definitions, see `backend/src/routes.js`.

## Database

The initial schema is in `backend/src/db/schema.sql`. Incremental changes are kept in
`backend/src/db/migrations/` and can be applied with:

```powershell
cd backend
npm run migrate:all
```

The main entities are:

`Users` · `jobs` · `applications` · `notifications` · `user_notification_status` ·
`alumni_stories` · `spoc_job_assignments` · `recruiter_kyc`

## Documentation

The [`readme files/`](./readme%20files/) directory contains focused notes for:

- Keycloak and JWT authentication
- Google and LinkedIn sign-in
- Cloudinary setup
- Recruiter KYC verification
- Job-status automation
- Notifications and application lifecycle
- SPOC assignment and messaging flows

## Testing and quality checks

Run the frontend lint/build checks:

```powershell
cd frontend
npm run lint
npm run build
```

Backend and notification flow checks are available in `test/` and
`backend/test_jwt_auth.js`. They may require a configured database, authentication
provider, and environment variables.

## Deployment notes

- Set `VITE_API_URL` to the deployed backend URL before building the frontend.
- Configure the deployed backend’s CORS allow-list for the frontend origin.
- Use managed PostgreSQL and Cloudinary in production rather than local file storage.
- Store Keycloak, database, OAuth, Cloudinary, and Hugging Face credentials in the
  hosting provider’s secret manager.
- Run database migrations as a release step before starting the API.
- Ensure uploaded files and cookies use HTTPS in production.

## Project status

PlaceMate is an actively evolving application. The `readme files/` directory records
feature implementations and migrations as the platform develops.