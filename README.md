# Automation & QA Assessment – [Your Name]

## Task 2 – Morning Brief (n8n)

**Flow:** Webhook (POST `/morning-brief`) → GitHub Search API → Code (top 5) → GitHub README API (per repo) → Code (attach snippet) → IF stars > 1000 → Set (🔥 hot / normal) → Merge → Code (compose) → Discord webhook.

**APIs used and why**
- **GitHub REST API** (`/search/repositories`, `/repos/{repo}/readme`): free, no signup needed to test, stable JSON, and the second endpoint gives a natural enrichment (README) for each item.
- **Discord webhook**: fastest free notification channel to set up; no bot or OAuth needed.

**Transformation:** The Code node drops archived repos, sorts by `stargazers_count` descending, keeps the top 5, and reduces each to `name, full_name, stars, url, description`. A second Code node strips markdown characters from the README and keeps the first 140 characters.

**Threshold:** `stars > 1000` marks a repo as "hot" (🔥 prefix). 1000 is a common popularity signal, and the topic search returns mostly repos above it, so both branches are exercised. Change it in the IF node.

**Error path**
- `GitHub Search` and `Top 5 Repos` use *On Error → Continue (error output)*. Failures (HTTP error, rate limit, empty result) go to `Error Info` → a Discord "⚠️ Morning Brief FAILED" message with the error text.
- `GitHub Search` also retries 3× (2 s apart) before failing.
- `Fetch README` uses *Continue (regular output)*: if one README fails, that repo still appears in the digest with "(README unavailable)" instead of killing the run.
- Credentials: `GitHub Token` (Header Auth: `Authorization: Bearer <token>`, optional but raises the rate limit from 60 to 5,000/hr) and `Discord Webhook` are stored in the n8n credentials store, none in the JSON.

**Test:** `curl -X POST https://<your-n8n>/webhook-test/morning-brief`

## Bonus – Uptime Monitor
Every 5 min: Ping the app (10 s timeout, 3 retries, `neverError` so non-2xx codes are captured) → Code records status, response time, and running stats → IF not 200 → Discord alert. A second trigger at 09:00 posts a daily summary (checks, failures, uptime %, avg/max response time). Stats use workflow static data, so they only persist while the workflow is **active**. Retries are included in the measured response time. If the target is unreachable, code is `0` and it still alerts.

## Setup
1. Import both JSON files. 2. Create credentials and select them on the nodes marked `REPLACE_ME`. 3. Run and screenshot.

---

# Task 1 – QA Report Template (fill in from your own testing)

Test target: https://demo.realworld.io. Checklist of edge cases to try:
- Sign-up: empty fields, invalid email, duplicate email/username, very long username, password with spaces, `<script>` in username
- Login: wrong password (is the message specific enough / too specific?), session after refresh, expired/deleted token in localStorage
- Articles: empty title/body, 10k-character body, emoji and special characters, markdown/HTML injection, duplicate titles, tags with spaces
- Edit/delete: editing another user's article via URL, delete then back button, double-click submit
- Comments, favorites, follow while logged out
- Accessibility: keyboard-only navigation, missing labels, contrast, screen-reader names
- Network: throttle to Slow 3G, go offline mid-submit, check the console and Network tab for errors

| # | Title / Summary | Steps to Reproduce | Expected vs Actual | Severity | Suspected Cause |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |

**Root-cause analysis (5–10 sentences):** what happens → why → how to fix.
