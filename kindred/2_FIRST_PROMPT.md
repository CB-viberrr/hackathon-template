# First prompt: paste in PLAN mode. Approve the plan, then switch to Agent mode and run 3_BUILD_PROMPTS.md

Read the Project Knowledge first and follow it strictly. Here is the approved concept. Turn it into the build plan (data model with RLS, routes, edge functions, build order) and wait for my approval. Don't change the concept.

## Name: **Kindred**. "Volunteer once. Qualify everywhere."
Problem: joining an established charity (Red Cross, food banks, hospices) feels like applying for a full-time job: separate applications, background checks and trainings per organisation, weeks of onboarding. Willing volunteers drop out; charities can't staff peaks.
Solution: a two-sided marketplace (Facebook Marketplace × LinkedIn × group chat). Volunteers keep ONE verified profile with portable qualifications. Charities post shifts that require specific credentials. Qualified volunteers book in one tap. Each shift gets a group chat.

## Roles
- volunteer · charity_admin (belongs to one organisation) · platform admin (user_roles + has_role).

## Features (ranked by demo value)
1. **Opportunity marketplace** (/explore): card grid of shifts with filters (cause, distance/city, date, time commitment: "2 hrs", "one-off", "weekly") and a "You're qualified" badge vs "Needs: Food Safety L1". Detail page with org, map placeholder, spots left, required credentials, "Book shift".
2. **Volunteer profile = LinkedIn for good** (/me, /v/:id): headline, causes, skills, availability grid, a **Credential Wallet** (verified badges with issuer + expiry), hours logged, orgs helped, endorsements from charities.
3. **Unified trainings** (/learn): a shared catalogue of short modules (Safeguarding Basics, Food Safety L1, First Aid Awareness, Manual Handling, Data Privacy for Volunteers), each 3–5 lessons + a 5-question quiz. Passing issues a credential to the wallet that is accepted by every participating charity. Show "Accepted by: Red Cross, City Food Bank, …".
4. **One-tap booking + eligibility engine**: booking checks the required credentials server-side (edge function). If something is missing: "Complete Food Safety L1 (12 min) to unlock 14 shifts". This is the "aha" moment.
5. **Shift group chat** (/shift/:id/chat): realtime chat for everyone booked + the org coordinator, with pinned logistics. Supabase Realtime.
6. **Charity dashboard** (/org): post a shift (title, date, spots, required credentials), see applicants and their wallets, mark attendance → logs hours + optional endorsement. KPI tiles: fill rate, volunteers this month, hours delivered.
7. **AI matchmaker** (edge function, Lovable AI): "For you" ranked shifts with a one-line reason ("Matches your Saturday availability + Food Safety credential"), and AI-assisted shift posting (charity types one sentence → structured shift draft). Scripted fallback per Project Knowledge.

## 60-second demo story
1. Landing: "Joining the Red Cross shouldn't feel like a job application." → Sign in as **Maya (volunteer)**.
2. /explore: "For you" shows 3 AI-matched shifts. One food bank shift says "Needs Food Safety L1".
3. Tap → /learn → complete a 3-lesson module + quiz (fast demo mode) → credential appears in the wallet → "Unlocked 14 shifts at 6 charities".
4. Book the food bank shift in one tap → you land in the shift group chat; the coordinator's welcome message is already there.
5. Switch to **City Food Bank admin**: fill rate jumps, Maya's verified wallet is visible, AI-draft a new shift from one sentence.
6. Close: profile shows hours + endorsements, a portable volunteer CV.

## Seed data (realistic, fictional orgs inspired by real categories. Do NOT use real charity logos/brands as partners)
- 6 orgs: "Harbor Red Relief" (disaster response), "City Food Bank", "Northside Hospice", "Green Streets Cleanup", "Youth Code Mentors", "Shelter Together".
- 30 shifts over the next 14 days, 5 training modules, 3 demo users: maya@demo (volunteer, 1 credential), alex@demo (volunteer, 4 credentials), foodbank@demo (charity_admin). One-click "Try as Maya / Try as Food Bank" buttons on the auth page.

## Brand
- Warm, trustworthy, modern. Primary: deep teal (HSL ~ 174 62% 30%); accent: warm coral (~ 12 85% 62%); success green; neutral warm greys. Font: Inter for UI + a humanist serif (Fraunces) for headings. Rounded-2xl cards, friendly illustrations as simple icons (lucide).

## Cut list if behind
Cut in this order: AI shift drafting → endorsements → availability grid → map. Never cut: marketplace, training → credential → unlock, booking, chat.
