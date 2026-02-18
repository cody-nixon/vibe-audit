# RESEARCH.md — Problem Discovery
Date: 2026-02-18

## Sources Searched
- Twitter/X (bird CLI): "someone should build", "why is there no app", "wish there was a tool", "AI replaced my job", "nobody built this yet"
- Reddit: r/SomebodyMakeThis (top/week), r/freelance (top/week), r/Entrepreneur (top/week)
- Hacker News: Ask HN (front page)
- Google/Brave search: vibe coding security, AI layoffs 2026, freelancer payment problems

---

## Candidate Problems Found

### 1. Renter's Digital Vault (Reddit r/SomebodyMakeThis, 16 upvotes)
- **Problem**: Renters lose security deposit disputes because they have no organized, timestamped proof of move-in/out condition
- **Who has it**: ~44M US renters
- **Existing solutions**: General cloud storage (unorganized), landlord-facing tools (Avail, RentRedi)
- **Why inadequate**: No tenant-centric, legally organized, dispute-ready export format
- **Similarity to past builds**: Not in PAST_BUILDS.md — clear!
- **Notes**: Strong idea but reactive (you only need it once a year), low return-user rate

### 2. Vibe-Coded App Security Auditor (Multiple sources — Wiz.io, DevClass, escape.tech)
- **Problem**: Non-technical founders using vibe coding (Cursor, Lovable, v0, Bolt) are deploying apps to production with major security holes. 20% deployed without auth. 2,000+ vulnerabilities found across vibe-coded apps (escape.tech study). Hardcoded secrets, SQL injection, no rate limiting, XSS.
- **Who has it**: ~500K+ vibe coders (estimate based on Cursor's 1M+ users), indie hackers, non-technical founders. Exploding population.
- **Existing solutions**: Snyk, SonarQube — but they require developer setup, GitHub integration, CI/CD pipelines. Too complex for non-technical users. Escape.tech targets enterprises.
- **Why inadequate**: All existing scanners assume you CAN READ CODE. Vibe coders often can't. They need plain-English results, prioritized fixes, and ideally auto-generated prompts they can paste back into Cursor/Claude to fix the issue.
- **Similarity to past builds**: Not in PAST_BUILDS.md — clear!
- **Score**: 🔥 HIGH

### 3. AI Model Deprecation Alerter (Twitter — GPT-4o deprecation outrage)
- **Problem**: OpenAI deprecated GPT-4o with minimal notice (2 weeks for consumers). Thousands of people had workflows and emotional attachments to specific models that vanished overnight. AI API model churn is accelerating.
- **Who has it**: Developers, researchers, power users relying on specific AI model behavior
- **Similarity to past builds**: Dangerously close to PriceShift (change detection + alerts). REJECTED.

### 4. Screen Time / Collective App Lock (Reddit r/SomebodyMakeThis)
- **Problem**: Personal screen time apps fail because you can always override them. Idea: same random 1-hour window daily when social media unlocks for everyone.
- **Notes**: Interesting social angle but requires mobile app (not web), iOS/Android restrictions make this very hard to build. Web app can't enforce this.

### 5. Knowledge Self-Assessment After Learning (Reddit r/SomebodyMakeThis)
- **Problem**: People watch YouTube tutorials and feel like they learned, but can't retain anything. They want personalized assessment based on what they actually learned.
- **Similar to**: Flashcard/spaced repetition territory. Close to NameDrill concepts. REJECTED.

### 6. Elderly Web Accessibility Helper (HN — 43 points)
- **Problem**: Modern websites are confusing and hostile to elderly users. Drives people to tears. 
- **Existing solutions**: Some browser accessibility extensions, OS-level zoom/font settings
- **Gap**: No tool that simplifies complex web pages into elder-friendly versions in real time
- **Notes**: Interesting but requires browser extension (not pure web app), very hard monetization story

### 7. Freelancer Payment Delay / Proof of Work (Multiple)
- **Problem**: 63% of freelancers wait 30+ days to get paid (Jobbers.io 2026 report). Disputes about work quality are common with no formal audit trail.
- **Existing solutions**: Bonsai contracts, Upwork escrow
- **Notes**: Covered by existing market. Not novel enough.

### 8. AI-Driven Business Succession Matching (Twitter)
- **Problem**: 4M+ small businesses need succession plans. Finding buyers is manual.
- **Notes**: Niche B2B, very high trust requirements, not a simple web app.

### 9. Cold Call Sequence Tracker (Reddit r/Entrepreneur, top post)
- **Problem**: Sales reps need multi-channel sequences (call → LinkedIn → email) but most CRMs don't optimize this automatically.
- **Notes**: Extremely crowded space (Apollo.io, Outreach, Salesloft). Skip.

### 10. Vibe Coding to Deployment Security Gap (Deep dive on #2)
- **Problem refinement**: When non-technical builders "vibe code" an app and then deploy it to production (via Vercel, Railway, etc.), they have no idea what security problems they've introduced. They can't read security audit reports (too technical). There's no tool that:
  1. Accepts a URL (the deployed app)
  2. Scans it for security issues
  3. Explains issues in plain English with severity
  4. Gives a "Fix Prompt" — copy-paste text they can paste into their AI coding tool to fix the issue
- **Market**: The vibe-coding population is growing EXPLOSIVELY. Cursor alone crossed 1M users. Lovable, v0, Bolt all growing rapidly. This is a NEW market with NEW security debt being created every day.

---

## Problem Candidates Scored

| # | Problem | Urgency | Impact | Novelty | Feasibility | Monetization | Bookmark Test | Unique vs Past |
|---|---------|---------|--------|---------|-------------|--------------|---------------|----------------|
| 1 | Renter Vault | Med | High | Med | High | Med | Low | ✅ |
| 2 | Vibe App Security Scanner | 🔥 HIGH | 🔥 HIGH | 🔥 HIGH | High | 🔥 HIGH | 🔥 HIGH | ✅ |
| 6 | Elder Web Helper | Med | High | Med | Low | Low | Med | ✅ |
| 7 | Freelancer Proof | Med | High | Low | Med | Med | Med | ✅ |

---

## Winner: Problem #2 — Vibe-Coded App Security Auditor

**Sources:**
- Wiz.io blog: "20% of vibe-coded apps deployed publicly without authentication" (Sept 2025)
- DevClass.com: "Vibe coded applications full of security blunders" (Jan 2026)
- escape.tech: "2,000+ vulnerabilities found in vibe-coded apps" (Oct 2025)
- Invicti: hardcoded secrets are now the most frequent flaw
- Wikipedia: "Vibe Coding Kills Open Source" paper published Jan 2026

The population of people who vibe-code but can't read security reports is **exploding**. No existing tool speaks their language. This is the gap.
