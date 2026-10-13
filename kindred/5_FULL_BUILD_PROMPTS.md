# Kindred: full build phase prompts (Agent mode)
Step 1 (Foundation) is done. Paste ONE block per message, in order. After each: C tests it, then the driver bookmarks the version in History.
If Lovable reports a problem, fix it before moving on: "Fix the issues you listed, then re-run the Definition of done."

---
## STEP 2: Opportunity marketplace
```
Build feature 1: Opportunity marketplace (/explore + /shift/:id), end to end per Project Knowledge.

Data (one migration, RLS in the same migration):
- organisations (public read; only that org's charity_admin can update): name, slug, cause, city, bio, logo_initials, verified bool.
- org_members: org_id, user_id (charity_admin link). Members can read their own row.
- credentials_catalog (public read): code, name, description, minutes_to_complete, issuer default 'Kindred Shared Standard'.
- shifts (public read; only that org's charity_admin can create, update or delete): org_id, title, description, cause, city, starts_at, ends_at, commitment ('one-off'|'weekly'|'2 hrs'), spots_total, spots_left, required_credential_codes text[].
- user_credentials (owner can read; insert only server-side): user_id, credential_code, issued_at, expires_at (issued + 2 years). Unique (user_id, credential_code).

Seed globally and idempotently:
- 6 fictional orgs: Harbor Red Relief (disaster response), City Food Bank, Northside Hospice, Green Streets Cleanup, Youth Code Mentors, Shelter Together.
- 5 credentials: SAFE-101 Safeguarding Basics (10 min), FOOD-L1 Food Safety L1 (12 min), FA-AWARE First Aid Awareness (15 min), MH-101 Manual Handling (8 min), PRIV-101 Data Privacy for Volunteers (6 min).
- 30 shifts over the next 14 days. About 40% need FOOD-L1 or SAFE-101. City Food Bank has at least 6 FOOD-L1 shifts, and across all orgs exactly 14 shifts require FOOD-L1 (and nothing Maya lacks besides it).
- Maya holds SAFE-101 only. Alex holds 4 credentials. Link foodbank@demo to City Food Bank as charity_admin.

/explore (StandardPage, eyebrow "Explore", title "Find your next shift"):
- Filters for cause, city, date range and commitment, held in the URL.
- A "Qualified only" toggle.
- Card grid; each card shows org initials avatar, title, date and time, city, spots left, and a badge:
  - "You're qualified" (success) if the user holds every required credential;
  - otherwise "Needs: Food Safety L1" (warning) listing what's missing.
- Skeleton, empty and error states.

/shift/:id (DetailPage):
- Org header and description.
- Required credentials as chips showing held or missing.
- Spots left, live via Realtime.
- Primary button: "Book shift" if qualified, else "Unlock with Food Safety L1 · 12 min", linking to /learn/FOOD-L1.
- The booking button can be a stub for now.

Run the Definition of done and report.
```

---
## STEP 3: Unified trainings, credential, unlock (THE WOW MOMENT: max polish)
```
Build feature 2: Unified trainings (/learn, /learn/:code). This is our demo's wow moment, so polish it most.

Data:
- training_modules (public read): credential_code, lessons jsonb [{title, body_md, minutes}], quiz jsonb [{q, options[4], answer_index}], accepted_by_org_ids uuid[]. Seed 5 modules (3 short lessons + 5 quiz questions each, realistic content; every module accepted by all 6 orgs).
- progress (private): user_id, feature 'training', key credential_code, status, step, total, draft jsonb. Save on every step.

/learn (StandardPage "Learn once. Qualify everywhere."):
- A card per module: minutes, an "Accepted by 6 charities" row of org avatars, and a status ("Start", "Step 2 of 4", or "Earned ✓").
- The user's wallet summary at the top.

/learn/:code (DetailPage): lesson stepper → quiz.
- Resumes from progress ("Welcome back, picking up at step 2"), with a "Start over" option.
- Add a "Demo mode" button visible only to demo accounts: it marks the lessons read and pre-selects the correct answers, so the demo takes 10 seconds.

Server function issue_credential(code):
- Re-grades the submitted answers server-side (pass = 4/5 or better).
- Inserts user_credentials idempotently and writes the audit_log.
- Returns { credential, unlocked_shift_count, unlocked_org_count }, computed from shifts the user is now fully qualified for that they weren't before.

Success screen:
- Credential badge animates in (.pop); confetti-free, tasteful.
- Headline "Food Safety L1 earned", then "Unlocked 14 shifts at 6 charities".
- Primary button "See unlocked shifts" goes to /explore?qualified=1&highlight=new; newly unlocked cards get a subtle "New" badge.
- If the user arrived from a shift, show a second button "Back to shift".

Run the Definition of done and report.
```

---
## STEP 4: One-tap booking + eligibility engine
```
Build feature 3: One-tap booking.

Data: bookings (volunteer reads own; that org's charity_admin reads its shift's bookings): shift_id, user_id, status ('booked'|'cancelled'|'attended'|'no_show'), created_at, unique (shift_id, user_id).

Server function book_shift(shift_id), one transaction:
- Check auth and the kill switch.
- Check the user holds all required_credential_codes; if not, return { ok:false, missing:[...] }.
- Run an atomic UPDATE shifts SET spots_left = spots_left - 1 WHERE id = $1 AND spots_left > 0. If no row updates, return "Shift is full".
- Insert the booking, add the user to the shift chat (next step; create the chat_members row if the table exists), and write the audit_log.
- Also add cancel_booking(shift_id), which gives the spot back atomically.

UI:
- The shift page button books in one tap with an optimistic loading state.
- On success: a toast "You're in!", then go to /shift/:id/chat (placeholder page until the next step).
- If credentials are missing: an inline callout "Complete Food Safety L1 (12 min) to unlock this and 13 more shifts" with a button.
- Add /me/shifts: upcoming and past bookings, with cancel.

Run the Definition of done (include: two users booking the last spot → only one succeeds).
```

---
## STEP 5: Shift group chat
```
Build feature 4: Shift group chat (/shift/:id/chat, WorkspacePage).

Data:
- chats: one per shift, created with the shift.
- chat_members (shared access model): chat_id, user_id, role 'volunteer'|'coordinator'. Booked volunteers and the org's charity_admin are members.
- messages (members only can read and insert their own; no update or delete except their own within 5 min): chat_id, user_id, body (1–1000 chars), pinned bool, created_at.

Behaviour:
- Realtime via Supabase subscriptions, cleaned up on unmount.
- The server-side insert redacts emails, phone numbers and card numbers in the body.
- Rate limit: 30 messages per minute per user.

Layout:
- Left 300px: the user's shift chats.
- Main: messages (own on the right; coordinator gets a "Coordinator" badge).
- Right 360px: shift details, pinned logistics (address, what to bring, contact), and the member list with their verified credential badges.

Seed: every seeded shift chat gets a pinned coordinator welcome ("Welcome! Meet at the side entrance at 8:45. Closed-toe shoes please.") plus 2–3 messages from other volunteers. When Maya books, she lands in a chat that already feels alive.

Mobile: chat list → chat → details as stacked views.

Run the Definition of done (another user who isn't a member gets "not found").
```

---
## STEP 6: Volunteer profile + Credential Wallet
```
Build feature 5: Volunteer profile (/me edit, /v/:handle public).

Data:
- profiles (public read for public fields; owner writes): handle, display_name, headline, city, causes text[], skills text[], availability jsonb (7 days × morning/afternoon/evening).
- endorsements (public read; insert only by a charity_admin of an org where the volunteer attended a shift): volunteer_id, org_id, author_id, body ≤280 chars.
- Hours: compute from bookings with status 'attended' (ends_at − starts_at).

/v/:handle ("LinkedIn for good"):
- Header: avatar initials, name, headline, city, "Verified volunteer".
- KPI row: hours given, charities helped, shifts completed, credentials.
- Credential Wallet: a grid of badge cards, each showing name, issuer "Kindred Shared Standard", issued date, expiry, and "Accepted by 6 charities".
- Causes and skills chips, an availability grid, and endorsements as quote cards.
- Share button that copies the link.

/me: edit form with zod validation. The availability grid is clickable toggles.

Seed: Maya has 6 attended past shifts (~18 hrs) and 2 endorsements. Alex is richer.

Run the Definition of done.
```

---
## STEP 7: Charity dashboard
```
Build feature 6: Charity dashboard (/org, charity_admin only; others get "not found").

WidePage:
- KPI row for the admin's org: fill rate this week (%), volunteers this month, hours delivered, open spots. Tabular numbers.
- Charts: a bar chart of bookings per day for the next 14 days, and a ring chart of fill rate.
- Shifts table: title, date, filled/total, needed credentials, and a "Manage" link.

/org/shifts/new (multi-step, resumable via progress):
1. Basics
2. Time & spots
3. Required credentials (pick from the catalog, showing "X volunteers nearby already qualify")
4. Review → publish

/org/shifts/:id manage:
- Applicants list, each with name, a mini credential wallet, hours, and a link to their profile.
- Actions: "Mark attended" or "No-show" (server function: updates the booking, logs hours, writes the audit_log) and an "Endorse" modal (≤280 chars).

Seed: City Food Bank shows a healthy dashboard (fill rate ~72%). Maya's booking appears live when she books.

Run the Definition of done.
```

---
## STEP 8: AI matchmaker + AI shift drafting
```
Build feature 7: AI, following the Project Knowledge AI pattern exactly (server-side, structured output + zod re-validation, $1/day budget reserved before each call, ai_usage log, kill switch, scripted fallback, user text inside <user_data>).

A) "For you" on /explore: a server function match_shifts().
- Send the profile (causes, skills, availability, city, credentials) plus up to 40 candidate shifts (id, title, org, cause, time, city, required creds, qualified bool).
- Model returns [{shift_id, reason ≤90 chars}] ×3, re-validated, and shift_ids must exist among the candidates.
- Fallback: a deterministic score (qualified ×3 + cause match ×2 + availability match ×2 + same city ×1), with templated reasons like "Matches your Saturday mornings + Food Safety credential".
- Show 3 highlighted cards at the top of /explore, each with an "AI" or "Sample" badge. Cache the result per user for 10 minutes.

B) AI shift draft on /org/shifts/new step 1: a textarea "Describe the shift in a sentence" with a "Draft it" button.
- Model returns {title, description ≤600, cause, commitment, spots_total, suggested_credential_codes[] from the catalog, duration_hours}. It prefills the form for the admin to edit.
- Fallback: keyword rules (food → FOOD-L1, kids/youth → SAFE-101, lifting/warehouse → MH-101) plus a templated description.

Use a fast, cheap model for both. Show the loading state on the button and "AI is resting for today" when the fallback is used.

Run the Definition of done, including: turn AI off in admin → both features still work with the fallback.
```

---
## STEP 9: Admin console
```
Build /admin (admin role only, enforced by RLS + has_role; others get "not found"). WidePage with tabs:
- Overview: users, orgs, shifts, bookings this week, credentials issued.
- Switches: a toggle per feature (marketplace, training, booking, chat, profile, org dashboard, AI). Read by both the UI and the server functions; a switched-off feature shows "switched off".
- AI: spend today vs the $1 cap (bar), calls today, fallback rate, and the last 20 ai_usage rows (no prompts).
- Users: search, role change (admin/charity_admin/volunteer, audit-logged), disable.
- Audit log: latest 50 rows, filterable by action.
Run the Definition of done.
```

---
## STEP 10: Landing page + demo-proofing
```
1) Landing page (/ when signed out):
- Hero "Joining a charity shouldn't feel like a job application." with the sub-line "One verified profile. Portable training. One-tap shifts."
- Primary CTA "Try as Maya", secondary "I'm a charity".
- A 3-step "How it works" row (Learn once → Qualify everywhere → Show up), a stat strip of seeded KPIs, and footer.

2) Demo-proof everything:
a) "Try as Maya / Alex / Food Bank" always work, and a "Reset demo" in the user menu (demo accounts only) restores the seed state, including removing Maya's FOOD-L1 credential so the wow moment can be replayed.
b) Walk the full demo path and fix anything slow or awkward: landing → Try as Maya → /explore For you → FOOD-L1 shift → Unlock → /learn demo mode → "Unlocked 14 shifts" → book → chat. Then switch to Food Bank → /org shows Maya's booking → AI-draft a shift.
c) Turn AI off and confirm the fallbacks look real.
d) Fix all console errors; no horizontal scroll at 390px; check dark mode on every page.
Report what you fixed.
```

---
## STEP 11: Security pass
```
Run a full security review against Project Knowledge, then fix and list every change:
- Every table has RLS with the correct access model (no `true` policies on private data); another user gets "not found" on read, update and delete.
- Roles only via has_role; charity_admin can only touch their own org's shifts, bookings and chats.
- All server functions: zod validation, auth check, kill-switch check, rate limits (including 5 sign-ups per IP per hour, which is still open from step 1), generic errors.
- Booking is atomic; chat input is redacted; nothing uses dangerouslySetInnerHTML; no secrets in the client.
- audit_log covers sign-up, role change, booking, attendance, credential issue and switch flips.
Then run the Lovable security scan and fix all findings.
```

---
## STEP 12: Polish (no new features)
```
Polish for judges without changing functionality:
- Page enter fade-rise and a 40ms list stagger; .pop on the credential earned and "You're in!".
- Copy pass: page leads ≤12 words, buttons are 1–3 word verbs, every empty state has a next step.
- Consistent spacing; one primary button per view.
- Favicon (teal heart-handshake), page titles, OG meta "Kindred: Volunteer once. Qualify everywhere."
- Screenshot every page at 1440 and 390 wide, light and dark, and fix anything clipped.
```

---
## STEP 13: Wrap (Plan mode)
```
Write README.md: the pitch, the problem (with the "joining feels like a full-time job" insight), the 60-second demo script, features, architecture (tables + access models, server functions, AI pattern + fallback + $1/day budget, security model), and what's next (real background-check partner, charity SSO, credential verification API). Then give 5 bullet talking points for judges.
```

---
### Rescue prompts
- Error loop (Plan mode): "Stop. Explain the root cause in plain English, give 2 fix options, recommend one. No code yet."
- Design drift: "Audit every component for raw colours, arbitrary values, inline styles and hand-written headings, and replace them with tokens and shared components."
- Something broke: restore the last bookmarked version in History, then retry the step in smaller pieces.
- Running out of time? Cut in this order: Step 8B (AI drafting) → endorsements → availability grid → admin Users tab. Never cut steps 2–5.
