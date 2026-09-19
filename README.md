# RecordSaathi

**A guided recovery flow for when your Parivahan (Sarathi/VAHAN) driving licence or vehicle record isn't found online.**

Built for **Build What Moves India** — a hackathon by Varun Mayya and OpenAI, where the challenge was to pick one real, painful government-service problem and rebuild a complete citizen journey around it, using Codex or an OpenAI model as a meaningful part of the build.

🔗 **Live demo:** [recordsaathi.vercel.app](https://recordsaathi.vercel.app)

---

## The problem

Search Parivahan for an older driving licence or a vehicle record and it's common to get back nothing — not an error, just silence. The portal offers no explanation and no next step. In practice this forces people into an undocumented "backlog entry" process at their RTO, often navigated only with the help of paid agents (₹700–2,000), because the system never tells you that's what you need to do.

This is especially acute for:
- Older or out-of-state licences that predate full digitisation
- NRIs who can't easily visit an RTO in person
- Anyone without a paid agent's knowledge of the offline workaround

## What RecordSaathi does

A single, complete citizen journey — not a static mockup:

**Home → Search → Record Not Found → Diagnostic → Remedy → Status**

1. **Search** a DL or RC number the way you would on Parivahan.
2. If it's **not found**, instead of a dead end, a short **diagnostic** figures out *why* it's likely missing (never digitised, mobile/detail mismatch, recently registered, etc.).
3. RecordSaathi generates a **personalised RTO letter** — the actual document a citizen would otherwise have needed an agent to draft — pre-filled with their details and the right case type.
4. The citizen can **book an RTO appointment** and then **track the request's status** afterward, including escalating it with a follow-up letter if it stalls.

## Tech stack & AI usage

| Layer | Choice |
|---|---|
| Frontend | React, deployed on Vercel |
| Backend | Node/Express, deployed on Render |
| AI-generated letters | Groq's OpenAI-compatible API running `openai/gpt-oss-20b` |
| Built with | OpenAI Codex, staged and reviewed incrementally through each screen |

**Honest disclosure:** we could not afford hosted OpenAI API costs for this prototype, so letter generation runs on Groq's infrastructure — but the model itself is `openai/gpt-oss-20b`, OpenAI's own open-weight model, so the "powered by an OpenAI model" requirement is met without misrepresenting which API is billing us. This is stated here deliberately rather than hidden, in line with the hackathon's own "Honesty" judging criterion.

## What's mocked (and why)

Per the hackathon brief, no live government system is touched. Specifically:
- DL/RC records are **seeded mock data**, not real Parivahan data (sample "found" values are in Local development below).
- There is **no authentication** — no real citizen accounts, OTPs, or logins are involved.
- RTO appointment booking and status tracking are **simulated**; no real RTO backend exists.
- No real Aadhaar, PAN, payment, or health data is used or requested anywhere.

## Known issues

This build has been through two adversarial QA passes, plus fixes since the second pass. Rather than hide the results, they're tracked here:

| # | Issue | Severity | Status |
|---|---|---|---|
| A1 | Non-root routes 404 on direct load/refresh (missing SPA rewrite) | High | **Fixed** — catch-all rewrite added to `vercel.json` |
| A2 | Generated letter was cached in `sessionStorage` and reused on a second diagnostic run in the same tab, showing stale content | High | **Fixed** — cache is now keyed to the record, answers, and language it was generated for |
| A3 | Browser back/forward during the diagnostic loses in-progress answers | High | Open |
| A4 | Escalating a request didn't persist — re-visiting the ticket showed it as not-yet-escalated | High | **Fixed, pending re-verification** — an `escalatedAt` marker is now written and displayed on the ticket |
| B1 | SQL-injection-pattern input (`' OR '1'='1'; DROP TABLE records;--`) causes an unstyled "Failed to fetch" instead of a graceful error | Medium | Open |
| B2 | Rapid triple-click on search submit fires 3 requests (no debounce/disable-on-submit) | Medium | Open |
| B3 | "Book RTO appointment" didn't look visually disabled before the review checkbox is ticked, even though it was functionally disabled | Low | **Fixed** — disabled styling now matches "Download copy" |

Confirmed working correctly: seeded record matching (including whitespace/case/date-format variance), non-seeded values correctly returning Not Found, input validation, XSS-safe handling of script/emoji input, native `disabled` attributes (no keyboard bypass), i18n (English/Hindi/Kannada) with no layout breakage, and mobile rendering at 375px.

## Local development

1. Copy `.env.example` to `.env` and add your Groq API key before Stage 2 features will work.
2. Install dependencies:
   ```
   npm install
   npm --prefix frontend install
   npm --prefix backend install
   ```
3. Run the dev server:
   ```
   npm run dev
   ```
4. Open [http://localhost:5173](http://localhost:5173)

**Sample seeded records:**
- Not-found DL: `KA01 9999 2005`, DOB `1978-11-02`
- Not-found RC: `KA05 AB 1234`
- Found DL: `KA01 1234 2005`, DOB `12-05-1980`
- Found RC: `KA01 CD 4567`

## Project structure

```
.claude/     — build notes and staged prompts used during development
backend/     — Express API, letter generation, mock record store
frontend/    — React app (search, diagnostic, remedy, status screens)
```

## About the hackathon

RecordSaathi was built for **Build What Moves India**, selected into the **Top 250 out of 13,694 submissions**. Judging criteria included: is the problem real, does the main journey actually work, is it simpler and more usable, is the thinking end-to-end, and are limitations honestly disclosed — the Known Issues section above exists because of that last criterion.

## Contributors

- [Sarvesh-107](https://github.com/Sarvesh-107)
- Built in collaboration with Claude (Anthropic) for planning, QA, and code review.
