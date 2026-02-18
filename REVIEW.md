# REVIEW.md — Multi-Role Design Review
Date: 2026-02-18

---

## 🎯 Product Review

### Core value prop in one sentence?
"Paste your app's URL and find out what security holes it has — written in English you can actually understand, with prompts to fix them using the same AI that built it."

### Is this actually solving the problem?
Yes. The insight is correct: vibe coders deploy fast and have no security feedback loop. The fix prompt angle is genuinely novel — no existing scanner does this. Snyk tells you you have a SQL injection. We tell you what to paste into Cursor to fix it.

### What's the emotional hook?
Fear of embarrassment and liability. Vibe coders are often building real products with real user data. The moment they realize their app is exposed, they'll do anything to fix it. We catch them at that exact moment of vulnerability (pun intended).

### Will people share this?
Yes — "I scanned my app and got a score of 23/100 — here's the report" is shareable. It has a social compare-yourself element. Twitter vibe coders will post their scores.

### Changes needed:
- Add "Tweet your score" button to results page
- Make the share URL a beautiful public-facing report, not just JSON
- Score needs a catchy name: not just "42" but "VibeScore™: 42/100"

---

## 🔨 QA Review

### What's the most likely thing to break?

1. **Scan timeouts**: The endpoint discovery crawl hitting 50+ paths could timeout on slow servers. Sites behind Cloudflare may block the scanner (rate limiting, WAF).
   - **Fix**: Set a 90-second max scan time. If we get blocked, report it as a finding ("Site uses WAF — manual review needed for some checks"). Retry with different User-Agent.

2. **False positives**: Reporting a missing header as "Critical" when it's actually fine for that app type (e.g., a static landing page doesn't need HSTS).
   - **Fix**: Severity must consider context. A missing auth header on `/about` is Low. On `/admin` it's Critical. Context-aware severity scoring.

3. **Scanning malicious URLs**: A user submits `http://192.168.1.1` or `http://localhost:3000` — SSRF vulnerability in OUR app.
   - **Fix**: **MANDATORY** — URL validation must reject: private IPs, localhost, 169.254.x.x (AWS metadata), non-HTTP protocols. Allowlist only public https:// URLs. This is a security vulnerability in the scanner itself.

4. **Rate abuse**: Someone hammers our scanner against a third-party site (DDoS by proxy).
   - **Fix**: Rate limit by IP (Redis). Respect `robots.txt`. Add disclaimer: "By scanning, you confirm you own or have permission to test this URL." Store consent in DB per scan.

5. **Long-running jobs crashing**: Worker process dies mid-scan.
   - **Fix**: BullMQ handles job retries. Scan status moves to 'failed' after 3 attempts. User gets error message.

### Key edge cases:
- URL requires authentication to access (behind login wall): scanner can only test the login page itself, which is a finding in itself ("We couldn't access your app — make sure your login page is secure")
- Single-page app with all content loaded client-side: headers and TLS still testable; endpoint discovery needs to try common API paths
- URL returns 404 immediately: fail fast with clear error
- Scan of an app that no longer exists: fail gracefully

---

## ⚙️ Engineering Review

### Is this the simplest architecture?
Mostly yes. One concern: BullMQ + Redis adds operational complexity. For MVP, consider:
- **Alternative**: Just use `setTimeout` + database polling for simplicity. No queue needed if scan volume is low initially.
- **Decision**: Keep BullMQ but use Upstash Redis (managed, no server to run). Adds ~$0/month for low volume.

### Unnecessary complexity?
- The AI fix prompt generation could be a simple template system for MVP instead of calling OpenAI per finding. This reduces cost and latency.
  - **Decision**: Start with templates for common findings (90% of cases). Add AI generation for rare/complex findings in v2.

### What's genuinely hard here?
- **SSRF protection**: Must be airtight. See QA review.
- **Scan accuracy**: False positives will destroy trust. Better to report fewer findings with high confidence than many uncertain ones.
- **Rate limiting our scanner**: We must be a good citizen. Respect rate limits on target servers.

### Missing technical decisions:
- What happens to scan data? Anonymous scans: delete after 7 days. Authenticated: keep forever (or until user deletes).
- How do we handle scan queuing during traffic spikes? BullMQ concurrency limit: max 5 concurrent scans per worker. Scale workers horizontally on Railway.

---

## 🎨 Design Review

### First-time user lands — do they get it in 5 seconds?
**Current risk**: "Security scanner for vibe-coded apps" might confuse someone who doesn't identify as a "vibe coder."

**Revised copy for hero section:**
- Primary: "Is your AI-built app safe to launch?"
- Sub: "Paste your URL. We scan for common security holes and show you exactly how to fix them — even if you've never written a line of code."
- CTA: "Scan My App — It's free"

This works for both "vibe coders" who know the term and people who just "built an app with Cursor" without knowing the vocabulary.

### What's the visual tone?
- Should feel like a developer tool, but approachable. Not enterprise-heavy (no navy blue, no stock photos of servers).
- Inspiration: similar to Linear's clarity but Vercel's friendliness.
- Color coding for severity is critical: Red = Critical, Orange = High, Yellow = Medium, Blue = Low.
- Score should be bold and prominent. Think: letter grade on a health inspection.

### Onboarding friction?
- Don't force signup before scanning. Anonymous scan first, then show signup prompt after results ("Save your report and track progress").
- Email required after scan to see full findings? No — show everything for free. Gate only on "save" and "continuous monitoring."

---

## 🔒 Security Review

### Attack surface?
1. **SSRF** (most critical): Scanner makes HTTP requests to user-provided URLs. Must validate against private IP ranges, localhost, cloud metadata endpoints. See QA.
2. **Rate limiting on our API**: Prevent scanner from being used as a DDoS bot. Limit to 3 anon scans/IP/day, 10/account/month free tier.
3. **SQL injection in our own app**: Use Supabase client (parameterized queries). Never string-concatenate SQL.
4. **XSS**: Scan reports display user-provided URL and page content excerpts. Sanitize everything before rendering. React's JSX is safe for rendered text, but `dangerouslySetInnerHTML` is forbidden.
5. **API key management**: OpenAI API key, Supabase service key — all in environment variables on Railway. Never in source code.
6. **Webhook verification**: Stripe webhooks for subscription must verify signature.
7. **Consent logging**: Store per-scan: IP address, timestamp, declared ownership. Legal protection if someone uses us to probe sites they don't own.

### Data handling:
- We store: URL scanned, scan results, findings, user account info
- We do NOT store: page content, user data from scanned apps, any PII from scanned sites
- GDPR: User can request scan deletion. Add "Delete my data" to settings.

---

## Plan Revisions Based on Review

1. ✅ Hero copy changed from "vibe-coded app" jargon to "AI-built app" language
2. ✅ Added "Tweet your score" sharing CTA
3. ✅ SSRF protection added as mandatory first-commit security item
4. ✅ Added consent checkbox to scan form ("I own or have permission to test this URL")
5. ✅ Template-based fix prompts for MVP, AI for v2
6. ✅ Anonymous scans: full results shown, gate only on "save" feature
7. ✅ False positive mitigation: context-aware severity scoring
8. ✅ Scan timeout: 90 seconds max, graceful failure
9. ✅ Added "Tweet your VibeScore" to results
10. ✅ "VibeScore" branding for the score number
