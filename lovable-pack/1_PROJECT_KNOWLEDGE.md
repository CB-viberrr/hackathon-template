# Project Knowledge (paste into Project Settings → Knowledge. Max ~10k chars)

## Who I am / how to work with me
- Solo builder at a one-day hackathon. Optimise for a flawless 60-second live demo, not breadth.
- Work in small, verifiable steps. After each change, tell me in 2–3 plain-English lines what changed and what to click to see it.
- Never rewrite working features unprompted. Never delete data or tables without asking.
- If a request is ambiguous, pick the option that best serves the demo and state the assumption.

## Stack (do not deviate)
- React + Vite + TypeScript + Tailwind + shadcn/ui. React Router for pages.
- Lovable Cloud for database, auth, storage and edge functions.
- AI features: call Lovable AI from an edge function only. Never call AI from the browser, and never put keys in client code.

## Non-negotiable: every feature works end to end
- Each feature ships with real DB persistence plus loading, empty and error states. No dead buttons, no "coming soon".
- Seed realistic demo data, so a brand-new account sees a populated app. Provide a "Reset demo data" action for the signed-in user.
- Multi-step flows save their progress, so a refresh never loses work.

## Security baseline (apply to every table and function)
- RLS ON for every table, created in the same migration. Default policy: a user can only select, insert, update or delete rows where user_id = auth.uid(). Shared or public data needs an explicit, read-only policy.
- Roles live in a separate `user_roles` table (enum app_role: 'admin','user'), checked via a SECURITY DEFINER `has_role(uid, role)` function. Never store roles on profiles or in client state.
- Validate every input with zod on the client AND in edge functions. Strip unknown fields. Enforce length limits.
- Edge functions verify the JWT, re-check authorisation server-side and return generic error messages (log details server-side).
- Never use the service-role key outside edge functions. Never render user content with dangerouslySetInnerHTML.
- Keep an `audit_log` table (actor, action, entity, created_at) written by edge functions for sensitive actions. Admin read-only.
- Rate-limit AI and write-heavy edge functions per user (simple counter table, e.g. 30 calls per hour).
- Before I publish, run the security scan and fix every finding.

## AI pattern (every AI feature)
- Edge function → Lovable AI with structured output (tool calling / JSON schema). Validate the response with zod.
- If the call fails, times out, is rate-limited, returns invalid JSON or the kill switch is off → return a scripted, realistic fallback response with the same shape. The demo must never break on AI.
- Site-wide daily AI budget: an `ai_usage` table. Refuse to call (use the fallback) once the daily cap is hit.
- Show a subtle "AI" / "Sample" badge so I know which path ran.

## Admin console (/admin, admin role only)
- Feature kill switches and an AI kill switch (`feature_flags` table, read by both UI and edge functions).
- AI usage today vs cap, the latest 50 audit-log rows and a user count.

## Design system (strict)
- All colour, type, spacing, radius and motion come from semantic tokens in index.css and tailwind.config.ts (HSL CSS variables). NO raw hex or Tailwind colour classes (e.g. text-white, bg-blue-500) in components. No inline styles.
- Use shadcn components and variants. Add variants rather than one-off classes.
- Use a fixed set of layouts: AppShell (nav + content), PageHeader (title, subtitle, primary action), Section, Card grid, StatTile, Chart card (recharts).
- Light AND dark mode both polished. Fully responsive. Check 375px mobile on every page.
- Tone: confident, clean, premium. Generous whitespace, one accent colour, subtle motion (150–250ms).

## Definition of done (check before you say "done")
1. Works logged-out (redirects to auth) and logged-in. 2. Loading, empty and error states. 3. RLS verified for the new table(s). 4. Mobile plus dark mode look right. 5. No console errors. 6. Demo data present.
