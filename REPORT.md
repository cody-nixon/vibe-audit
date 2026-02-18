# REPORT.md — Daily Design Summary
Date: 2026-02-18
Product: VibeAudit
GitHub: https://github.com/cody-nixon/vibe-audit

---

## The Problem

**Vibe-coded apps are being deployed to production with critical security holes — and their builders have no way to find or fix them.**

**Sources**:
- Wiz.io Research (Sept 2025): "20% of vibe-coded apps deployed publicly without authentication" — https://www.wiz.io/blog/common-security-risks-in-vibe-coded-apps
- escape.tech (Oct 2025): "2,000+ vulnerabilities found in vibe-coded apps" — https://escape.tech/blog/methodology-how-we-discovered-vulnerabilities-apps-built-with-vibe-coding/
- DevClass (Jan 2026): "Vibe coded applications full of security blunders" — https://devclass.com/2026/01/15/vibe-coded-applications-full-of-security-blunders/
- Invicti: Hardcoded secrets now the #1 security flaw in vibe-coded apps

The vibe-coding population is exploding. Cursor alone crossed 1M users. Lovable, v0, Bolt, Replit add millions more. Every single deployed app is potential attack surface. The problem is accelerating every day.

---

## Why I Chose This

After scanning 10 candidate problems, VibeAudit scored highest on every dimension:

- **Urgency**: Apps are live TODAY with exposed admin panels and API keys in the source code. This is happening right now.
- **Impact**: Millions of vibe coders, growing 40%+/month. Each one deploys multiple apps.
- **Novelty**: Every existing scanner (Snyk, SonarQube, escape.tech, Checkmarx) assumes you can read and write code. The vibe coder market has been completely ignored.
- **Feasibility**: URL → HTTP checks → report. No novel technology required. Scanning is a solved problem; the UX translation is what's new.
- **Monetization**: Clear SaaS model. Free tier converts at launch (Product Hunt lead gen) → Pro tier ($9/month) for continuous monitoring.
- **Bookmark test**: A vibe coder bookmarks this before every deploy and comes back every time they push.

The renter's vault and elder web helper were strong second choices, but neither had the explosive timing + market gap that VibeAudit has.

---

## Design Summary

**VibeAudit** — "You shipped fast. Now make sure you shipped safe."

A URL-based security scanner for non-technical builders:
1. Paste your app's URL
2. Get a VibeScore (0-100) and findings sorted by severity
3. Click any finding to get a plain-English explanation + copy-paste fix prompt for Cursor/Claude/Lovable

**30+ security checks** across: authentication, HTTPS/TLS, security headers, exposed endpoints, CORS, rate limiting, information disclosure, and common misconfigs.

**Tech stack**: React + Vite (frontend), Node.js + Express (backend), Supabase (auth + DB), BullMQ (async scan queue), Vercel + Railway (hosting), Stripe (payments).

**Pricing**: Free (3 scans/day, 7-day history) → Free Account (10/month) → Pro $9/month (unlimited + monitoring).

**4-week implementation roadmap**. Estimated 75-105 hours for an experienced developer solo.

---

## Key Insight

Every other security scanner stops at "here's the problem." VibeAudit says "here's the exact prompt to paste into Cursor to fix it."

This is not a security tool for security engineers. It's a security tool for people who have never thought about security — which is exactly who is building most of the apps being deployed right now.

The fix prompt is the product. The scan is how you justify showing it to them.

---

## GitHub Repo
https://github.com/cody-nixon/vibe-audit

### Files Delivered
- `RESEARCH.md` — 10 candidate problems, sources, scoring
- `DECISION.md` — Problem selection with scoring and rationale
- `PLAN.md` — Full technical spec: stack, API, data model, auth, UI flow
- `REVIEW.md` — Product, QA, Engineering, Design, Security review + revisions
- `WIREFRAMES.md` — 10 screens with detailed layout, copy, and interactions
- `DESIGN.md` — Complete design document: exec summary, persona, spec, roadmap, metrics, risks
- `REPORT.md` — This file
