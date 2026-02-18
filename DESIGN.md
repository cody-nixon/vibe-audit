# DESIGN.md — Complete Design Document
Product: VibeAudit
Date: 2026-02-18
Repo: https://github.com/cody-nixon/vibe-audit

---

## Executive Summary

**The Problem**: Non-technical founders and indie hackers are using AI coding tools (Cursor, Lovable, v0, Bolt) to build and deploy web apps faster than ever — but they're shipping serious security vulnerabilities they can't see. Wiz Research found 20% of vibe-coded apps deployed publicly without authentication. escape.tech found 2,000+ vulnerabilities across vibe-coded apps. The tools that could catch these issues require developer expertise the vibe coder doesn't have.

**The Solution**: VibeAudit is a URL-based security scanner built specifically for non-technical builders. Paste your app's URL, get a plain-English security report with a VibeScore (0-100), and receive copy-paste "fix prompts" for every issue — text you paste directly back into the AI tool that built your app.

**The Differentiator**: Every other scanner speaks engineer. VibeAudit speaks vibe coder.

---

## Target User Persona

**Name**: Taylor, the Vibe Coder

**Who they are**:
- Non-technical or semi-technical. Used Cursor/Claude/Lovable to build a web app in days or weeks.
- The app is real: it has users, maybe payment, definitely some form of user data.
- Taylor has never written a security test. The word "CVSS" means nothing to them.
- Taylor is excited about shipping but has a nagging feeling: "Is this safe?"

**What Taylor needs**:
- A fast answer to "is my app secure enough to go live?"
- Results they can act on — not a report that requires a security engineer to interpret
- A way to fix issues without learning new skills — by pasting a prompt into Cursor

**Where Taylor hangs out**:
- Twitter/X (@vibecoder, @indiemaker communities)
- Hacker News (Show HN)
- r/SideProject, r/Entrepreneur
- Indie Hackers, Product Hunt

**Taylor's trigger to scan**:
- Just about to launch
- Saw a tweet about someone's vibe-coded app getting hacked
- A user mentioned "your site doesn't seem secure"
- Read an article about vibe coding security problems

---

## Complete Technical Spec

### Tech Stack
| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Frontend | React + Vite + TypeScript | Fast builds, familiar ecosystem |
| UI Components | shadcn/ui + Tailwind CSS | Clean, accessible, fast to build |
| Backend | Node.js + Express + TypeScript | Consistent with team standards |
| Database | Supabase (PostgreSQL) | Auth + DB in one, generous free tier |
| Job Queue | BullMQ + Upstash Redis | Async scan processing, managed Redis |
| Scan Engine | Custom Node.js modules | Orchestrates multiple check types |
| Fix Prompts | Template-based (v1), AI (v2) | Fast, cheap, predictable for MVP |
| Frontend Host | Vercel | Free tier, instant deploy |
| Backend Host | Railway | Simple scaling, worker support |
| Payments | Stripe | Subscriptions for Pro tier |

### Data Model
```
users (via Supabase Auth)
  └── profiles (plan, scans_used, stripe_customer_id)
  └── apps (saved apps for monitoring)
      └── scans (scan results)
          └── findings (individual security issues)
```

### Security Check Suite (30+ checks across 8 categories)
1. **Authentication**: Admin routes, API endpoints without auth, cookie flags
2. **HTTPS/TLS**: Redirect enforcement, certificate validity, HSTS
3. **Security Headers**: CSP, X-Frame-Options, X-Content-Type-Options, Permissions-Policy
4. **Information Disclosure**: Exposed .env, .git, stack traces in errors, source maps, server version headers
5. **CORS**: Access-Control-Allow-Origin wildcards on auth endpoints
6. **Rate Limiting**: Login/API rapid-fire test
7. **Exposed Endpoints**: /admin, /debug, /metrics, /graphql introspection, /api/users
8. **Common Misconfigs**: Default credentials check, GraphQL introspection, open file directories

### Scoring Algorithm
```
VibeScore = 100 - (Critical × 25) - (High × 10) - (Medium × 5) - (Low × 1)
Minimum: 0 (can't go negative)
```

### Pricing
| Tier | Price | Limits |
|------|-------|--------|
| Anonymous | Free | 3 scans/IP/day, reports expire in 7 days |
| Free Account | Free | 10 scans/month, reports saved 30 days |
| Pro | $9/month | Unlimited scans, weekly auto-rescan, email alerts, reports forever |

---

## Wireframes (Summary)
See WIREFRAMES.md for full detail on all 10 screens.

### Key Screens
1. **Landing**: URL input + consent checkbox + "Scan My App" — zero friction to start
2. **Scanning**: Live progress with stages completing in real time
3. **Results**: VibeScore + findings sorted by severity + expandable fix prompts
4. **Share Report**: Public-facing report for sharing/tweeting
5. **Auth**: Google OAuth primary, email/password secondary
6. **Dashboard**: List of saved apps with VibeScore trends
7. **Pricing**: Free vs Pro feature comparison

---

## Implementation Roadmap

### Week 1: Core Scanning Engine + MVP
**Goal**: A URL in, a security report out. No auth, no DB.

- [ ] Project setup (Vite frontend, Express backend, Railway deploy)
- [ ] SSRF protection (IP validation — MUST be done before any URL fetching)
- [ ] HTTP header check module
- [ ] TLS/HTTPS check module
- [ ] Exposed endpoint discovery module (50 common paths)
- [ ] Authentication check module
- [ ] Information disclosure module
- [ ] VibeScore calculation
- [ ] Fix prompt template library (20 most common findings)
- [ ] Results page (working report)
- [ ] Landing page with URL input

**Deliverable**: You can paste a URL and get a real security report.

### Week 2: Persistence + Auth + Sharing
**Goal**: Save scans, share reports, sign in.

- [ ] Supabase setup (database schema)
- [ ] BullMQ + Upstash Redis for async scan queue
- [ ] Scan polling (frontend TanStack Query)
- [ ] Anonymous scan → database storage
- [ ] Share URL generation (share_token)
- [ ] Public share page
- [ ] Supabase Auth setup
- [ ] Google OAuth integration
- [ ] Email/password signup + login
- [ ] "Save scan to account" post-scan CTA

**Deliverable**: Full anonymous flow working. Sign up and save scan.

### Week 3: Dashboard + Payments + Polish
**Goal**: Pro tier, monitoring, production-ready.

- [ ] User dashboard (saved apps, VibeScore trends)
- [ ] Stripe integration (Pro subscription, $9/month)
- [ ] Stripe webhook handler (subscription events)
- [ ] Rate limiting (IP-based for anonymous, account-based for free)
- [ ] Pro feature: weekly auto-rescan scheduler
- [ ] Pro feature: email alerts on score drop
- [ ] Rate limiting on scan endpoints
- [ ] Error states (scan failed, invalid URL, blocked by WAF)
- [ ] "Tweet your score" button
- [ ] Polish: loading states, animations, mobile responsive

**Deliverable**: Full product, launched on Product Hunt.

### Week 4: CORS + Rate Limit Checks + Fix Prompt Quality
**Goal**: Improve accuracy, add remaining checks, improve fix prompts.

- [ ] CORS check module
- [ ] Rate limiting check module
- [ ] GraphQL introspection check
- [ ] Fix prompt quality review (test against 20 real vibe-coded apps)
- [ ] Edge case handling (SPA apps, Cloudflare-protected apps)
- [ ] Basic analytics (PostHog or simple DB tracking)

---

## Estimated Effort (experienced developer)

| Phase | Estimated Hours |
|-------|----------------|
| Core scanning engine | 20-30 hours |
| Frontend (landing + results) | 15-20 hours |
| Auth + database + sharing | 15-20 hours |
| Dashboard + payments | 15-20 hours |
| Polish + edge cases | 10-15 hours |
| **Total** | **75-105 hours (~3-4 weeks solo)** |

---

## Risks and Mitigations

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| SSRF vulnerability in scanner | HIGH (if not addressed) | CRITICAL | First commit must include IP validation. Block private ranges, localhost, AWS metadata. |
| False positives destroy trust | MEDIUM | HIGH | Context-aware severity. If we're unsure, report as "Info" not "Critical". |
| Cloudflare/WAF blocks scanner | HIGH | MEDIUM | Detect 403/429, report as "Protected by WAF — fewer checks possible", still report headers etc. |
| Scan timeout on slow servers | MEDIUM | LOW | 90-second hard timeout. Graceful partial report. |
| Legal exposure (scanning non-owned sites) | MEDIUM | HIGH | Consent checkbox + disclaimer. Store consent timestamp per scan. No automated scanning. |
| Competing tool launches | LOW-MEDIUM | MEDIUM | First-mover advantage. Launch ASAP. Build community (Twitter, Product Hunt). |
| False negatives (missing real issues) | MEDIUM | MEDIUM | Be clear: "VibeAudit catches common issues, not all issues. Use for a first pass." |

---

## Success Metrics

### Launch (Month 1)
- 500 unique URL scans
- 50 signups
- Product Hunt: top 10 of the day
- 3 organic tweets about VibeAudit from real users

### Growth (Month 3)
- 5,000 scans/month
- 200 free accounts
- 20 Pro subscribers (~$180 MRR)
- Average VibeScore for new users: <50 (validates problem is real)

### Product-Market Fit Signal
- Users return to scan after fixing issues (rescan rate > 40%)
- Organic Twitter/X mentions without prompting
- Users sharing their VibeScore publicly

### What "Working" Looks Like
A vibe coder scans their app, finds a critical issue they didn't know about, fixes it with our prompt, rescans, sees their VibeScore go from 34 to 78, and tweets about it. That's the loop.

---

## Key Design Decisions and Rationale

1. **URL-only, no code** — Maximum accessibility. Removes all friction. Anyone with a deployed app can use this.

2. **Fix prompts, not just findings** — The killer differentiator. Every other scanner says "you have a problem." We say "here's exactly what to type to fix it." This is the thing that makes vibe coders share this with each other.

3. **Anonymous first** — Don't gate the core value. Signing up to use a security tool you've never heard of is a big ask. Show value first, ask for account later.

4. **VibeScore** — A single number is easier to care about than a list of CVEs. It's shareable, it's tweetable, it tracks progress over time. It creates an emotional investment.

5. **Template fix prompts (not AI)** — Templates are faster, cheaper, and more reliable. The 20 most common findings have predictable fixes. AI generation adds cost and latency for marginal benefit in v1.

6. **$9/month Pro** — Low enough to be an impulse buy for an indie hacker. High enough to be real revenue. No annual plans at launch (simplicity first).
