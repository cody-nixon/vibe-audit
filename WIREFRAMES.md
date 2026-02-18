# WIREFRAMES.md — UI/UX Design
Date: 2026-02-18

All wireframes are text-based. Each describes layout, elements, copy, and interactions.

---

## Screen 1: Landing Page (Homepage)

### Layout
```
┌──────────────────────────────────────────────────────────────────────┐
│  [VibeAudit logo]                    [Sign in]  [Sign up — Free]     │  ← Nav
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│         Is your AI-built app safe to launch?                         │  ← H1 (bold, 3xl)
│                                                                      │
│    Paste your URL. We scan for security holes and show you           │  ← Subhead (gray, lg)
│    exactly how to fix them — even if you never wrote a line          │
│    of code.                                                          │
│                                                                      │
│    ┌─────────────────────────────────────────────┐  ┌────────────┐  │
│    │ https://your-app.vercel.app                 │  │  Scan Now →│  │  ← URL input + CTA
│    └─────────────────────────────────────────────┘  └────────────┘  │
│                                                                      │
│    ☐ I own or have permission to test this URL                       │  ← Consent checkbox
│                                                                      │
│    Free • No signup required • Results in ~60 seconds                │  ← Trust copy
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│                        SOCIAL PROOF                                  │
│                                                                      │
│   "Found 3 critical issues in my Cursor app I had no idea about."   │
│   — @vibecoder_sam, 2.3K followers                                   │
│                                                                      │
│   "The fix prompts saved me hours of Googling."                      │
│   — @indiemaker_jane, 1.1K followers                                 │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│                       HOW IT WORKS (3 steps)                         │
│                                                                      │
│   [1] Paste Your URL    [2] We Scan It      [3] Fix With AI          │
│   Enter your deployed   We check 30+ common  Copy the fix prompt     │
│   app's URL below.      security issues.     into Cursor or Claude.  │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│                       WHAT WE CHECK                                  │
│                                                                      │
│   [🔑 Authentication]  [🔐 HTTPS & TLS]   [📋 Security Headers]     │
│   [🚪 Exposed APIs]    [📦 Info Leakage]   [⚡ Rate Limiting]        │
│   [🌐 CORS Config]     [🔓 Admin Routes]   [🔑 Hardcoded Secrets]   │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│   RECENT PUBLIC SCANS (last 10, anonymized)                          │
│                                                                      │
│   myapp.vercel.app    VibeScore: 34/100  🔴 2 Critical  ● 5 min ago │
│   saas.fly.dev        VibeScore: 78/100  🟡 1 Medium    ● 12 min ago│
│   coolapp.netlify.app VibeScore: 91/100  🟢 All Clear   ● 23 min ago│
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│   VibeAudit © 2026 · Built for vibe coders · Privacy · Terms         │  ← Footer
└──────────────────────────────────────────────────────────────────────┘
```

### Key Interactions
- Typing in URL input: real-time URL validation (show green checkmark if valid URL)
- Consent checkbox: must be checked before "Scan Now" button activates (button is grayed out until checked)
- "Scan Now" click: navigates to /scan/:id with loading state

### Mobile Layout
- Stack: Hero text → URL input → consent → CTA button (full width)
- "How It Works" becomes vertical list (not 3-column)
- Recent scans list hidden on mobile

---

## Screen 2: Scanning in Progress

### Layout
```
┌──────────────────────────────────────────────────────────────────────┐
│  [VibeAudit logo]                                                    │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│              Scanning: myapp.vercel.app                              │  ← URL being scanned
│                                                                      │
│   ┌──────────────────────────────────────────────────────────────┐   │
│   │  ████████████████████░░░░░░░░░░░░  58%                      │   │  ← Progress bar
│   └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│   ✅ HTTPS check complete                                             │  ← Live scan steps
│   ✅ Security headers analyzed                                        │
│   ✅ TLS certificate verified                                         │
│   🔄 Checking API endpoints... (this takes ~30 seconds)              │  ← Currently running
│   ⬜ Testing rate limiting                                            │  ← Pending
│   ⬜ Generating fix prompts                                           │
│                                                                      │
│   Results appear automatically when complete.                        │
│                                                                      │
│   Scanning typically takes 45–90 seconds.                            │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Key Interactions
- Progress bar animates with real data from server (polling /api/scans/:id every 2s)
- Steps update in real time as they complete
- Page auto-redirects to results when status = "complete"
- If user closes tab, scan continues. Returning to URL shows results.

---

## Screen 3: Scan Results Page (Main Feature)

### Layout
```
┌──────────────────────────────────────────────────────────────────────┐
│  [VibeAudit logo]                    [Sign in]  [Sign up — Free]     │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Security Report: myapp.vercel.app                                   │  ← H1
│  Scanned 2026-02-18 at 10:42 AM                                      │
│                                                                      │
│  ┌────────────────┐   ┌───────────────────────────────────────────┐  │
│  │                │   │  🔴 Critical  2                           │  │
│  │   VibeScore    │   │  🟠 High      3                           │  │
│  │                │   │  🟡 Medium    5                           │  │
│  │     34/100     │   │  🔵 Low       4                           │  │
│  │                │   │  ──────────────────────────────────────── │  │
│  │  Needs Work ⚠️  │   │  ℹ️  Info     2                           │  │
│  └────────────────┘   └───────────────────────────────────────────┘  │
│                                                                      │
│  [Share Report]  [Tweet Score]  [Save Scan — Sign Up]                │  ← Action buttons
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│                         FINDINGS                                     │
│                                                                      │
│  ─── CRITICAL (2) ──────────────────────────────────────────────    │
│                                                                      │
│  🔴 Admin panel accessible without login           [Fix with AI ▼]  │
│                                                                      │
│  [Expanded view when clicked:]                                       │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │ WHAT'S WRONG                                                  │    │
│  │ Your /admin page loads for anyone on the internet — no        │    │
│  │ password required. Anyone can view your admin dashboard.      │    │
│  │                                                               │    │
│  │ WHY IT MATTERS                                                │    │
│  │ Attackers regularly scan for exposed admin panels.            │    │
│  │ This could expose your users' data and give full control      │    │
│  │ of your app to anyone who finds it.                           │    │
│  │                                                               │    │
│  │ HOW TO FIX IT                                                 │    │
│  │ Copy this prompt into Cursor, Claude, or your AI tool:        │    │
│  │                                                               │    │
│  │ ┌────────────────────────────────────────────────────────┐   │    │
│  │ │ "My /admin route is publicly accessible without login. │   │    │
│  │ │ Add authentication middleware that checks for a valid  │   │    │
│  │ │ session before allowing access. If the user isn't      │   │    │
│  │ │ logged in, redirect them to /login. Use the existing   │   │    │
│  │ │ auth middleware pattern in the codebase."              │   │    │
│  │ └────────────────────────────────────────────────────────┘   │    │
│  │                              [📋 Copy Prompt]                 │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  🔴 API key exposed in page source                  [Fix with AI ▼]  │
│  ─── HIGH (3) ──────────────────────────────────────────────────     │
│  🟠 No rate limiting on login endpoint              [Fix with AI ▼]  │
│  🟠 Missing Content-Security-Policy header          [Fix with AI ▼]  │
│  🟠 HTTP requests not redirected to HTTPS           [Fix with AI ▼]  │
│  ─── MEDIUM (5) ────────────────────────────────────────────────     │
│  🟡 GraphQL introspection enabled in production     [Fix with AI ▼]  │
│  [... more ...]                                                      │
│  ─── LOW (4) ───────────────────────────────────────────────────     │
│  🔵 X-Frame-Options header missing                  [Fix with AI ▼]  │
│  [... more ...]                                                      │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Want to track your score over time?                                │
│   Sign up free to save this scan and rescan after your next deploy.  │
│                                                                      │
│   [Create Free Account]                                              │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Key Interactions
- Score circle: color changes based on score (0-39 red, 40-69 orange, 70-89 yellow, 90-100 green)
- Each finding: click to expand/collapse
- "Fix with AI" button: expands to show the plain English + fix prompt
- "Copy Prompt": copies fix prompt to clipboard, shows "✅ Copied!" for 2 seconds
- "Share Report": generates a public URL (vibeaudit.com/scan/abc123) and copies it
- "Tweet Score": opens Twitter with pre-filled text: "My app scored 34/100 on @VibeAudit. Found 2 critical issues. Here's how I'm fixing them 👇 [link]"
- "Save Scan" / "Create Free Account": opens signup modal

### Mobile Layout
- Score and summary stack vertically
- Findings list full-width
- Fix prompt in expandable accordion

---

## Screen 4: Share Report (Public View)

### Same as Results page but:
- Nav shows only VibeAudit logo + "Scan your app →" CTA
- No "Save Scan" button (already saved)
- "Scanned by VibeAudit" watermark at bottom
- Score and all findings fully visible to anyone with the link

---

## Screen 5: Auth — Sign Up

### Modal (not new page)
```
┌───────────────────────────────────────────┐
│ X                                         │
│                                           │
│  Create your free account                 │
│                                           │
│  [G] Continue with Google                 │  ← Primary CTA
│                                           │
│  ──────────── or ────────────             │
│                                           │
│  Email                                    │
│  [______________________________]         │
│                                           │
│  Password                                 │
│  [______________________________]         │
│                                           │
│  [Create Account]                         │
│                                           │
│  Already have an account? Sign in         │
│                                           │
│  By signing up you agree to our Terms     │
│  and Privacy Policy.                      │
└───────────────────────────────────────────┘
```

### Key Interactions
- Google OAuth opens popup, closes on success, redirects to dashboard
- Email/password: inline validation (password min 8 chars)
- After signup: if previous scan exists (anonymous), prompts to save it to account

---

## Screen 6: Sign In

Same as Sign Up but with "Sign In" header and no password requirements shown.
Forgot password link → email reset flow (Supabase handles this).

---

## Screen 7: Dashboard (Logged in, Saved Apps)

### Layout
```
┌──────────────────────────────────────────────────────────────────────┐
│  [VibeAudit]  [Dashboard]  [Scan New App]        [@username] [↓]     │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Your Apps                              [+ Add New App]              │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ myapp.vercel.app                                               │  │
│  │ VibeScore: 34 → 67  📈 +33 since last scan    2 days ago      │  │
│  │ [View Report]  [Rescan Now]                                    │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ coolapp.netlify.app                                            │  │
│  │ VibeScore: 91/100  🟢 All Clear                1 week ago     │  │
│  │ [View Report]  [Rescan Now]                                    │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ──────────────────────────────────────────────────────────────      │
│  Free plan: 3 of 10 monthly scans used.                              │
│  [Upgrade to Pro — $9/month] for unlimited scans + monitoring.       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Screen 8: Empty State (No Scans Yet)

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│              🛡️                                                      │
│                                                                      │
│         No scans yet                                                 │
│                                                                      │
│    Scan your first app to see your VibeScore.                        │
│                                                                      │
│         [Scan Your First App →]                                      │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Screen 9: Error State (Scan Failed)

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│   ⚠️  We couldn't complete the scan                                  │
│                                                                      │
│   We tried to scan: myapp.vercel.app                                │
│                                                                      │
│   Possible reasons:                                                  │
│   • The site is down or taking too long to respond                   │
│   • The URL requires a login to access                               │
│   • The site is blocking automated requests                          │
│                                                                      │
│   [Try Again]  [Scan a Different URL]                                │
│                                                                      │
│   If the problem persists, email us: help@vibeaudit.com              │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Screen 10: Upgrade / Pricing Page

### Layout
```
┌──────────────────────────────────────────────────────────────────────┐
│  [Free]                          [Pro — $9/month]                    │
│                                                                      │
│  ✅ 3 scans/month                ✅ Unlimited scans                  │
│  ✅ Full report                  ✅ Weekly auto-rescan               │
│  ✅ Fix prompts                  ✅ Email alerts if score drops       │
│  ✅ Shareable report             ✅ Priority scan queue              │
│  ✅ 7-day report history         ✅ Unlimited report history         │
│                                                                      │
│                                 [Upgrade to Pro]                     │
│                                                                      │
│  Billed monthly. Cancel anytime.                                     │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Mobile Considerations

1. **Landing page**: URL input is full-width, keyboard appears on focus, "Scan Now" stays visible above keyboard (sticky CTA)
2. **Results page**: Score prominent at top, findings are collapsible accordions (tap to expand)
3. **Fix prompt**: Full-width expandable card, "Copy Prompt" button large enough to tap
4. **Navigation**: Hamburger menu on mobile, collapses to icon-only

---

## Microcopy Notes

- Button text: "Scan My App" (not "Submit" or "Start") — action-oriented, personal
- Score label: "VibeScore: 34/100" — branded
- Severity labels: Don't say "CVSS 9.8 Critical" — say "🔴 Critical: This is serious"
- Empty state: "No issues found... yet. Re-scan after your next deployment." — realistic, not false reassurance
- Loading state: "We're checking your app for security holes. This usually takes 45-90 seconds." — accurate expectations
- Error: Always human, never "Error 500" — "Something went wrong. We're on it."
