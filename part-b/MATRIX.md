# Impact vs Effort Matrix
 
## The Matrix
 
|                   | Low Effort         | High Effort        |
|-------------------|--------------------|--------------------|
| **High Impact**   | Problem 2 (Filters)<br>Problem 6 (Session)    | Problem 1 (Tatkal)<br>Problem 4 (CAPTCHA)    |
| **Low Impact**    | Problem 5 (Vikalp UI)    | Problem 3 (Seat Selection)    |
 
## How I Scored Each Dimension
 
### Impact Scoring (1–5)
I scored Impact based on:
- Number of users affected (from Part A frequency analysis).
- Whether the problem blocks the core booking flow directly.
- Severity of consequence for the user (e.g., losing all data vs. UI confusion).
 
### Effort Scoring (1–5)
I scored Effort based on:
- Number of system components touched.
- Whether new infrastructure (like queuing servers or ML models) is required.
- Risk of breaking existing crucial flows (like authentication).
- Dependencies on legacy backend APIs.
 
---
 
## Placement Justifications
 
### Problem 1 (Tatkal Booking Crashes) — High Impact, High Effort
This issue affects hundreds of thousands of daily users and completely breaks the core revenue-generating flow of the platform. However, the effort is very high because it requires deploying edge-level queuing infrastructure (like Cloudflare Waiting Room) and modifying the core booking gateway to validate queue tokens. It is the most critical but hardest feature to build.
 
### Problem 2 (Search Filters Reset) — High Impact, Low Effort
Filter resetting causes extreme friction for millions of users who rely on the search page to plan travel. The effort is low because it only requires frontend changes using standard URL query parameter syncing and React Router state management. There are zero backend API changes required.
 
### Problem 3 (Seat Selection Resets) — Low Impact, High Effort
While frustrating for elderly passengers, it affects a smaller subset of total bookings, keeping the overall impact lower. The effort is surprisingly high because it requires untangling legacy form state management, fixing serialization payloads, and safely ensuring backend APIs correctly bind the enum values without breaking the entire passenger validation flow.
 
### Problem 4 (CAPTCHA Loop) — High Impact, High Effort
This completely blocks the funnel at the very first step for all users, making its impact massive. The effort is high because replacing authentication security mechanisms with behavioral analysis (Turnstile) requires extensive security auditing, middleware rewriting, and rigorous testing against botnets.
 
### Problem 5 (Vikalp Scheme UI) — Low Impact, Low Effort
The confusion mainly affects waitlisted passengers, making its scope narrower than core booking failures. The effort is extremely low because it only involves replacing a single frontend checkbox with a static educational modal and clearer copy, requiring zero backend changes.
 
### Problem 6 (Session Expiry Data Loss) — High Impact, Low Effort
Losing 10 minutes of group booking data causes immense user rage and cart abandonment. The effort is low because implementing local browser caching (`localStorage`) and a frontend countdown timer requires only client-side logic and basic API session refresh calls.
 
---
 
## Recommended Sprint Order
1. **Problem 2 (Search Filters Reset)** — Quickest win to dramatically improve search UX without backend risk.
2. **Problem 6 (Session Expiry Data Loss)** — High value for user retention with low engineering overhead.
3. **Problem 5 (Vikalp Scheme UI)** — Trivial UI fix that reduces customer support burden immediately.
4. **Problem 4 (CAPTCHA Loop)** — Critical funnel blocker, prioritize security engineering to unblock users.
5. **Problem 1 (Tatkal Virtual Queue)** — Massive project, requires dedicated infrastructure team to begin parallel work.
6. **Problem 3 (Seat Selection Resets)** — Lower priority, tackle as technical debt cleanup in frontend state management.
