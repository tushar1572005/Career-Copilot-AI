# GenAi Project

A full-stack application that generates tailored interview reports and ATS-friendly resumes using Google GenAI. The backend (Node/Express) provides APIs for user auth, resume upload, AI-powered interview reports and PDF resume generation. The frontend (Vite + React) provides authentication and UI for requesting reports and downloading resumes.

**Features**
- **AI Reports:** Generate interview reports containing a match score, technical & behavioral questions, skill gaps, and a day-wise preparation plan.
- **Resume PDF:** Produce a tailored, ATS-friendly resume PDF based on candidate info and job description.
- **Authentication:** Cookie-based JWT auth with register/login/logout and protected routes.
- **Persistence:** Interview reports are stored in MongoDB per user.

**Repository Layout**
- **Backend:** [Backend](Backend)
- **Frontend:** [Frontend](Frontend)

**Important files**
- [Backend/server.js](Backend/server.js)
- [Backend/src/app.js](Backend/src/app.js)
- [Backend/src/services/ai.service.js](Backend/src/services/ai.service.js)

**Tech stack**
- Backend: Node.js, Express, MongoDB, Mongoose
- AI: @google/genai (Gemini models)
- PDF generation: Puppeteer
- Frontend: React (Vite)

**Prerequisites**
- Node.js 18+ and npm
- A running MongoDB instance (connection URI)
- Google GenAI API key with required quota

Environment
Create a `.env` file in the `Backend/` folder with the following variables:

```
MONGO_URI=<your-mongo-uri>
JWT_SECRET=<your-jwt-secret>
GOOGLE_GENAI_API_KEY=<your-google-genai-api-key>
```

Backend — run locally

Install and start the backend:

```bash
cd Backend
npm install
npm run dev
```

- Server listens on port `3000` (see [Backend/server.js](Backend/server.js)).
- CORS allows `http://localhost:5173` by default (see [Backend/src/app.js](Backend/src/app.js)).

Frontend — run locally

```bash
cd Frontend
npm install
npm run dev
```

- Vite dev server default: `http://localhost:5173`.

API Overview

Authentication
- POST /api/auth/register
  - Body (JSON): { "username": "jdoe", "email": "jdoe@example.com", "password": "secret" }
  - Response: 201, sets cookie `token`.

- POST /api/auth/login
  - Body (JSON): { "email": "jdoe@example.com", "password": "secret" }
  - Response: 200, sets cookie `token`.

- GET /api/auth/logout
  - Clears auth cookie and blacklists token.

- GET /api/auth/get-me
  - Protected: returns logged-in user details.

Interview endpoints (protected)
- POST /api/interview/
  - Content-Type: multipart/form-data
  - Fields: `resume` (file, PDF), `selfDescription` (string), `jobDescription` (string)
  - Response: 201 { interviewReport }

- GET /api/interview/
  - Response: 200 { interviewReports }

- GET /api/interview/report/:interviewId
  - Response: 200 { interviewReport }

- POST /api/interview/resume/pdf/:interviewReportId
  - Response: `application/pdf` (attachment)

Example curl requests

Register:
```bash
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"jdoe","email":"jdoe@example.com","password":"secret"}' \
  -c cookies.txt
```

Login:
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"jdoe@example.com","password":"secret"}' \
  -c cookies.txt
```

Generate interview report (upload resume):
```bash
curl -X POST http://localhost:3000/api/interview/ \
  -b cookies.txt -c cookies.txt \
  -F "resume=@/path/to/resume.pdf" \
  -F "selfDescription=Experienced backend engineer..." \
  -F "jobDescription=Senior Node.js Developer role at X"
```

Download generated resume PDF:
```bash
curl -X POST http://localhost:3000/api/interview/resume/pdf/<INTERVIEW_ID> \
  -b cookies.txt -o tailored_resume.pdf
```

Notes & troubleshooting
- AI integration: Ensure `GOOGLE_GENAI_API_KEY` is valid and has quota. See `Backend/src/services/ai.service.js`.
- Puppeteer: On some Linux environments (containers/CI), additional system dependencies are required for Chromium. Consult Puppeteer docs if PDF generation fails.
- If the frontend is hosted elsewhere, update CORS origin in [Backend/src/app.js](Backend/src/app.js).