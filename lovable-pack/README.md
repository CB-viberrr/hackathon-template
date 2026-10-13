# Lovable setup in about 5 minutes (solo)

1. New project → (optional) Workspace Settings → Knowledge: paste "Who I am / how to work with me" from file 1.
2. Project Settings → **Knowledge**: paste all of `1_PROJECT_KNOWLEDGE.md` (under the ~10k char limit).
3. Toggle **Plan/Chat mode** → paste `2_FIRST_PROMPT.md` with your idea filled in → iterate → approve.
4. Switch to **Agent mode** → run `3_BUILD_PROMPTS.md` top to bottom, one prompt per message.
5. Before you publish: run the Security scan, then the Demo-proofing prompt.

Optional: once Lovable creates the repo (GitHub sync), add the knowledge file as `AGENTS.md` at the root. Lovable reads it on every request, which hardens the rules against long-session drift.
