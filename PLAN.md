# PLAN.md — Technical Design
Date: 2026-02-18

## Product: VibeAudit
**Tagline**: "You shipped fast. Now make sure you shipped safe."
**Core Value**: URL → plain-English security report + AI fix prompts for vibe-coded apps.

---

## User Stories (only the ones that matter)

### Must Have (MVP)
1. **As a vibe coder**, I can paste my app's URL and click "Scan Now" to get a security report in under 60 seconds
2. **As a vibe coder**, I can see results sorted by severity (Critical → High → Medium → Low) in plain English, not CVSS jargon
3. **As a vibe coder**, I can click on any finding and get a "Fix Prompt" I can copy and paste directly into Cursor, Claude, or Lovable
4. **As a vibe coder**, I can see a summary security score (0-100) so I know at a glance how bad it is
5. **As a vibe coder**, I can share a link to my scan report with a teammate or client

### Nice to Have (Post-MVP)
6. **As a paid user**, I can save my app and re-scan after each deployment to track improvement over time
7. **As a paid user**, I can get emailed when new vulnerability patterns are discovered that match my app
8. **As a paid user**, I can scan multiple apps and see a dashboard of all their security scores

### Out of Scope
- Code analysis (no GitHub required)
- Full penetration testing (we do automated checks, not manual)
- Compliance certifications (SOC 2, etc.)
- Mobile app scanning

---

## Tech Stack

### Frontend
- **React + Vite** (TypeScript) — fast, familiar, works everywhere
- **shadcn/ui** — component library per LESSONS.md
- **Tailwind CSS** — utility styling
- **TanStack Query** — async state management for polling scan results

**Why not Next.js?** The app is mostly interactive client-side (scan results, real-time polling). Vite is simpler and faster to build.

### Backend
- **Node.js + Express** (TypeScript) — consistent with team standards
- **Supabase** — database (PostgreSQL) + auth + storage
  - Tables: `scans`, `findings`, `users`, `apps`
  - Auth: email/password + Google OAuth
- **Bull/BullMQ** — job queue for running scans asynchronously (scans take 30-90 seconds)
- **Redis** — queue backend (can use Upstash for hosted)

### Scanning Engine (the core)
Custom scan runner that orchestrates multiple check types:

1. **HTTP Header Checks** (fetch + analyze response headers)
   - Missing: `X-Frame-Options`, `Content-Security-Policy`, `X-Content-Type-Options`, `Strict-Transport-Security`, `Permissions-Policy`
   - Exposed: Server version headers, framework signatures

2. **TLS/HTTPS Checks**
   - Is site HTTPS-only?
   - Does it redirect HTTP → HTTPS?
   - Certificate validity

3. **Exposed Endpoint Discovery** (passive crawl)
   - `/api/`, `/admin`, `/.env`, `/.git`, `/config`, `/debug`, `/graphql`
   - Look for directory listing
   - Look for unauthenticated API endpoints (try calling without a token)

4. **Authentication Checks**
   - Are admin routes accessible without a session?
   - Does the app expose a `/api/users` endpoint without auth?
   - Cookie security flags (HttpOnly, Secure, SameSite)

5. **Information Disclosure**
   - Error messages revealing stack traces, database names, file paths
   - Robots.txt revealing sensitive paths
   - Source maps deployed to production (leaks source code)
   - API keys visible in page source or network responses

6. **Rate Limiting Check**
   - Rapid-fire requests to login/API endpoint — does it get blocked?

7. **CORS Check**
   - `Access-Control-Allow-Origin: *` on authenticated endpoints?

8. **Common Misconfiguration Patterns**
   - GraphQL introspection enabled in production
   - Debug endpoints exposed (`/health` with full info, `/metrics`)
   - Default admin passwords on common frameworks

### AI Layer (Fix Prompts)
- **OpenAI GPT-4o-mini** (or Claude Haiku) — cheap, fast
- For each finding, generate a prompt like:
  ```
  "Your app is missing the Content-Security-Policy header. This allows attackers to inject 
  malicious scripts. Tell your AI coding assistant: 'Add a Content-Security-Policy header 
  to all HTTP responses in the Express middleware. Use this policy: [specific policy]. 
  Add it in the main app.js or server.js file where other middleware is configured.'"
  ```
- Prompts are structured: what the problem is (plain English) + what to say to Cursor/Claude

### Infrastructure
- **Vercel** — frontend hosting
- **Railway** — backend + Redis + worker processes
- **Supabase** — database + auth (managed PostgreSQL)
- **Domain**: vibeaudit.com (hypothetical)

---

## Architecture Overview

```
User → Frontend (React/Vite on Vercel)
         ↓ POST /api/scans { url }
       Backend API (Express on Railway)
         ↓ creates scan record (status: pending)
         ↓ pushes job to BullMQ
       Worker Process (Railway)
         ↓ runs scan checks (HTTP, TLS, endpoints, headers, etc.)
         ↓ calls OpenAI to generate fix prompts
         ↓ writes findings to Supabase
         ↓ updates scan status: complete
       Frontend (polling every 2s via TanStack Query)
         ↑ GET /api/scans/:id → returns findings
         ↑ renders report
```

---

## API Design

### POST /api/scans
**Request**:
```json
{
  "url": "https://myvibeapp.vercel.app",
  "email": "user@example.com"  // optional for unauthenticated users
}
```
**Response**:
```json
{
  "scan_id": "uuid",
  "status": "pending",
  "share_url": "https://vibeaudit.com/scan/abc123"
}
```

### GET /api/scans/:id
**Response**:
```json
{
  "scan_id": "uuid",
  "url": "https://myvibeapp.vercel.app",
  "status": "complete | pending | running | failed",
  "score": 42,
  "scanned_at": "2026-02-18T10:00:00Z",
  "summary": {
    "critical": 2,
    "high": 3,
    "medium": 5,
    "low": 4
  },
  "findings": [
    {
      "id": "uuid",
      "severity": "critical",
      "title": "Admin panel accessible without login",
      "plain_english": "Anyone on the internet can visit /admin and see your entire admin dashboard...",
      "technical_detail": "GET /admin returns 200 without Authorization header",
      "fix_prompt": "Tell your AI coding tool: 'Add authentication middleware to the /admin route...'",
      "category": "authentication"
    }
  ]
}
```

### POST /api/auth/signup
```json
{ "email": "...", "password": "..." }
```

### GET /api/apps (authenticated)
Returns list of saved apps for current user.

### POST /api/apps/:id/rescan (authenticated)
Triggers a new scan for a saved app.

---

## Data Model

### `scans` table
```sql
id          UUID PRIMARY KEY
url         TEXT NOT NULL
status      TEXT DEFAULT 'pending'  -- pending|running|complete|failed
score       INT                      -- 0-100, higher = safer
user_id     UUID REFERENCES users(id) ON DELETE SET NULL  -- null if anonymous
share_token TEXT UNIQUE              -- for shareable report URLs
created_at  TIMESTAMPTZ DEFAULT NOW()
completed_at TIMESTAMPTZ
```

### `findings` table
```sql
id              UUID PRIMARY KEY
scan_id         UUID REFERENCES scans(id) ON DELETE CASCADE
severity        TEXT NOT NULL  -- critical|high|medium|low|info
title           TEXT NOT NULL
plain_english   TEXT NOT NULL
technical_detail TEXT
fix_prompt      TEXT           -- AI-generated prompt for the user
category        TEXT           -- authentication|headers|disclosure|tls|cors|ratelimit
created_at      TIMESTAMPTZ DEFAULT NOW()
```

### `apps` table (for saved apps, paid tier)
```sql
id          UUID PRIMARY KEY
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
url         TEXT NOT NULL
name        TEXT
last_scan_id UUID REFERENCES scans(id) ON DELETE SET NULL
created_at  TIMESTAMPTZ DEFAULT NOW()
```

### `users` table (via Supabase Auth)
Standard Supabase auth.users table + `profiles` table:
```sql
id          UUID REFERENCES auth.users(id)
plan        TEXT DEFAULT 'free'   -- free|pro
scans_used  INT DEFAULT 0          -- lifetime count
stripe_customer_id TEXT
```

**Indexes**:
- `scans(user_id)` for user dashboard
- `scans(share_token)` for public share URLs
- `findings(scan_id)` for report rendering
- `apps(user_id)` for user's app list

---

## Auth Flow

### Anonymous users (Free tier)
- Can run scans without signup
- Results accessible via share URL for 7 days
- Limited to 3 scans per IP per day (rate limited in Redis)

### Authenticated users (Free account)
- Email + password signup
- Google OAuth ("Sign in with Google") — creates Google Cloud project, uses cody.nixon.nt@gmail.com console
- Can save scan history for 30 days
- Limited to 10 scans per month

### Pro users ($9/month)
- Unlimited scans
- Continuous monitoring (weekly auto-rescan + email alerts if score drops)
- Dashboard with all saved apps
- Scan history forever
- Priority queue (scans run faster)

---

## UI Flow

### Landing Page
1. Hero: URL input + "Scan Now" button. Tagline front and center.
2. User pastes URL → hits Scan → sees "Scanning..." state
3. Results page loads as findings come in (streaming/polling)

### Results Page
1. Score badge (big number, color-coded: red/orange/yellow/green)
2. Summary bar: Critical X | High X | Medium X | Low X
3. Findings list, sorted by severity
4. Each finding: expand to see plain English + fix prompt (with copy button)
5. CTA: "Save this scan" → signup prompt if not logged in

### Dashboard (logged in, paid)
1. List of saved apps with last score + trend arrow
2. "Rescan" button per app
3. Alert badge if score dropped since last scan

---

## What We're NOT Building
- CLI tool (too much friction for target user)
- VS Code extension (requires coding context they don't have)
- Full DAST scanner (Burp Suite level) — too complex and slow for our MVP
- Mobile app scanning (out of scope for web tool)
- Compliance reports (SOC2, HIPAA) — different market
- Code analysis (no GitHub required — that's our differentiator)

---

## Scan Time Estimates
- HTTP header checks: ~1s
- TLS check: ~2s
- Endpoint discovery (50 common paths): ~15-30s
- Rate limiting test: ~5s
- CORS check: ~2s
- Information disclosure scan: ~10s
- AI fix prompt generation (batch all findings): ~10-20s
- **Total**: ~45-70 seconds per scan

This is acceptable. Show a live progress bar with scan stages.
