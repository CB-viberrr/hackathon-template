# Kindred: Project Knowledge (paste into Project Settings → Knowledge)

## Working style
- The team is 4 people and partly new to code, so use plain English and explain jargon in one line. Before each change, say in 1–2 sentences what will change and why. After it, say exactly what to click to see it.
- Make small steps that always leave the app working. Get each feature working end to end in ~15 min, then polish it.
- Only touch the feature named in the prompt. Never rewrite working features, delete data or drop tables without asking.
- Prefer 4 sharp features over 8 broken ones. Never weaken the security or design rules for speed.
- Freeze: in the last 60 min before demos, warn before adding anything new and fix only demo-path bugs.

## Stack
React + Vite + TS + Tailwind + shadcn/ui, Lovable Cloud (DB, auth, storage, realtime, edge functions), Lovable AI from edge functions only. No third-party scripts or CDNs, and no browser fetch to outside sites (proxy through edge functions).

## Security
- **Auth:**
  - Passwords 8–128 chars, with the leaked-password check on.
  - Use the same error for an unknown user and a wrong password.
  - Never store tokens in localStorage beyond the Supabase default. No service-role key in the client.
- **RLS on every table, in the same migration.** Each table uses one access model:
  - `private`: owner only.
  - `shared`: owner + members.
  - `public`: all signed-in users read; only the creator writes.
  - Forbidden and unknown both look like "not found".
- **Roles:**
  - Roles live in a `user_roles` table (app_role: admin, charity_admin, volunteer) and are checked with a SECURITY DEFINER `has_role()`.
  - Never store roles on profiles or in client state.
- **Edge functions:**
  - Validate input with zod and drop unknown fields. Set string min/max lengths and array max sizes; cap the body at 64 KB; accept JSON only.
  - Verify the JWT and re-check authorisation on the server.
  - Expected errors return a human message; never leak internals.
- **Counters and booking:**
  - Booking spots and counters use atomic SQL or an RPC (`UPDATE … WHERE spots_left > 0`), never read-modify-write.
- **Rate limits** (Postgres counter table):

  | Bucket | Limit |
  |---|---|
  | Writes | 120/min per user |
  | AI calls | 40/min and 800/day per user |
  | Sign-ups | 5 per IP per hour |

  A limited request returns a friendly "Slow down a little. Try again in Ns".
- **Audit:** an `audit_log` table (user_id, action, jsonb detail, created_at) for sign-up, role change, booking, attendance marked, credential issued and kill-switch flips. Readable by admins only.
- **Content:** no dangerouslySetInnerHTML, eval or inline scripts. Before user text is stored as chat or sent to AI, redact emails, phone numbers, card numbers and street addresses as [EMAIL], [PHONE] etc.
- Before publishing, run the security scan and fix every finding.

## AI pattern (every AI feature)
- Server-side only. User text goes in a separate data field wrapped in `<user_data>…</user_data>`, never in the instructions.
- Structured output, re-validated server-side with zod. Trim over-length strings instead of rejecting them. A failed validation counts as a fallback.
- **Mandatory fallback:** no key, budget spent, kill switch off, error or timeout → return a scripted or precomputed answer of the same shape. Show one calm notice ("AI is resting for today") and a loading state on the button.
- **Budget:** an `ai_budget` row per day, capped at $1/day.
  - Reserve the worst-case cost before each call with a conditional UPDATE (`… WHERE reserved + x <= cap`). If no row updates, make no call; afterwards, reconcile to the actual cost.
  - Regular users can use 80% of the cap. Admins can use 100%, which saves the last 20% for the live demo.
- **Usage log:** `ai_usage` has one row per call (user, feature, tokens, cost, fallback yes/no, reason). Never store the prompts.
- **Models:** use a fast, cheap model by default and the strongest model only for the wow answer.

## Admin (/admin, admin role only, enforced by RLS)
Users and roles, per-feature kill switches (a switched-off feature shows "switched off"), an AI kill switch, AI spend today vs cap, and the latest audit log.

## Design system (locked)
- **Colour:** semantic tokens only, as HSL CSS variables in index.css, mapped in tailwind.config:
  - surfaces `--bg --surface --surface-2`; borders `--border`; text `--text --text-2 --text-3`
  - brand `--brand --brand-soft --on-brand`, plus `--accent`
  - status `--success --warning --danger`, each with a `-soft` variant
  - Brand is deep teal (174 62% 30%) and accent warm coral (12 85% 62%). Dark mode is required.
- **Type:** Inter for the UI; Fraunces only for page titles. Scale 12 / 13.5 / 15 / 17 / 20 / 24 / 30 / 38px. Tabular numbers in KPIs and tables.
- **Spacing and shape:** spacing scale 4 / 8 / 12 / 16 / 20 / 24 / 32 / 40 / 56px; radius soft 12px; shadows sm / md / lg.
- **Motion:** 120ms hover, 200ms state change, 360ms page enter (fade-rise). Lists stagger 40ms (max 12 items). Respect reduced motion.
- **Banned in feature code:** raw hex or Tailwind palette colours (bg-blue-500, text-white), arbitrary values (text-[13px], p-[7px]), inline styles, custom keyframes, hand-written h1/h2, custom page shells.
- **Shared layout components:**
  - Page templates: StandardPage (eyebrow, title, lead, actions, body), DetailPage (back link, narrow), WidePage (dashboards), WorkspacePage (left 300px, main, right 360px; use it for chat).
  - Section (the only heading inside a page), Card (whole card is a link, lifts on hover), KPI row, Badge, Empty, Skeleton, Toast, Confirm.
- **Buttons:** one primary per view. Variants: primary, secondary, soft, ghost, danger.
- **Copy:**
  - Page lead ≤12 words. A card is a title, one line and one action. No explanatory paragraphs.
  - Sentence case; buttons are 1–3 word verbs. Empty states say what's missing plus the next step.
  - Format numbers like "84%" and "6.8 hrs".
- **Every screen needs:** a title, one clear primary action, a designed empty state with a CTA, a loading skeleton, an error toast, and a mobile layout where the nav becomes a bottom bar.

## Demo data and flows
- **Seed per user:** on a user's first visit, seed 3–8 realistic records per feature. Make it idempotent (guard row plus a unique constraint), use no AI calls, spread dates over recent days, and make the records deletable. There is also a global seed of the 6 fictional orgs, 30 shifts and 5 training modules.
- **Demo logins:** "Try as Maya" (volunteer), "Try as Alex" (volunteer) and "Try as Food Bank" (charity_admin) buttons on the auth page.
- **Multi-step flows** (trainings, posting a shift): keep a server-side `progress` row (user, feature, key, status, step, total, draft).
  - Save on every step. Reopening shows "Welcome back, picking up at step 3".
  - Offer a "Start over" button.
- **Live:** use Supabase Realtime for chat and for spots left.

## Definition of done (check before saying done)
1. Create, see, reload, still there.
2. Seed data present, so no empty screens.
3. Empty, loading and error states designed.
4. AI has a fallback and redacted input.
5. RLS matches the access model, and another user gets "not found".
6. The feature is part of the demo story, and every step is a real click.
7. No console errors, no horizontal scroll at 390px, and dark mode correct.
