# Build prompts: paste one at a time, in order (Agent mode). Check the preview after each.

## Step 1: Foundation
Implement step 1 of the approved plan: enable Lovable Cloud, email auth (+ Google if quick), profiles table with auto-create trigger, user_roles + has_role(), feature_flags, audit_log, ai_usage tables with RLS. Build the design tokens for light and dark from the plan's theme, AppShell, PageHeader, Section, StatTile, Chart card, a theme toggle and protected routes. Make me admin: the first user to sign up gets the admin role.

## Step 2…N: Features (repeat per feature, in plan order)
Build feature <N: name> exactly as planned. End to end: tables + RLS, edge function(s), UI with loading/empty/error states, seeded demo data. Follow the AI pattern (structured output, zod validation, scripted fallback, budget, kill switch). Then run the Definition of Done checklist and report the result.

## Admin console
Build /admin per Project Knowledge: kill switches (each feature + AI), AI usage today vs cap, latest audit log, user count. Admin-only, enforced by RLS and has_role, not just hidden in the UI.

## Demo-proofing
Make the app demo-proof: (1) a new signup sees rich seeded data instantly, (2) a "Reset demo data" button, (3) test every AI feature with AI switched OFF in admin and confirm the fallbacks look real, (4) fix any console errors, (5) check every page at 375px and in dark mode and fix issues.

## Security pass
Run a full security review: every table has RLS with correct policies (no `true` policies on private data), roles only via has_role, zod validation in all edge functions, no secrets client-side, rate limits on AI functions, generic error messages. Then run the security scan and fix all findings. List what you changed.

## Polish
Polish for judges: landing page with the pitch + one CTA, empty states with helpful copy, micro-interactions (150–250ms), consistent spacing, favicon + page titles + OG meta. Do not change functionality.

## Wrap (Plan mode)
Write a README: pitch, the 60-second demo script, features, architecture (tables, edge functions, AI + fallback design, security model) and what's next. Also give me 5 bullet talking points for judges.

---
### Rescue prompts
- Stuck in an error loop: "Stop. Explain the root cause in plain English, list 2 fix options, recommend one. No code yet."  (Plan mode)
- Drift from the design system: "Audit all components for raw colours, inline styles and non-token classes, then replace them with semantic tokens/variants."
- Before risky changes, bookmark the working version in History so you can revert.
