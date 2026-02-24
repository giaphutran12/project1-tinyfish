# Agent Instructions

## Research Tool Priority Chain (MANDATORY)

When researching ANY topic (external docs, model comparisons, best practices, library usage, etc.):

1. **Nia FIRST** — Always use Nia (`search.sh deep`, `oracle.sh`, `search.sh web`, `search.sh universal`). Nia provides the most accurate, complete context.
2. **Exa web search SECOND** — If Nia fails (timeout, rate limit, error), fall back to `websearch_web_search_exa`.
3. **Context7 / Webfetch LAST** — Only if both Nia and Exa fail.

**CRITICAL**: NEVER skip Nia. NEVER go straight to Exa or Context7 when Nia is available.
**CRITICAL**: ALWAYS run Nia research via background subagents (`run_in_background=true`) or `task(subagent_type='librarian')` — NEVER block the main agent with synchronous Nia calls.

## Main Agent Must NEVER Block (CRITICAL)

The main agent must ALWAYS remain free to receive user messages. **NEVER run long-running operations synchronously.**

- **Nia research**: ALWAYS delegate to a background subagent with `run_in_background=true`. NEVER run `search.sh deep`, `oracle.sh`, or any Nia script directly from the main agent.
- **Any operation > 5 seconds**: Delegate to a subagent or run in background.
- **While waiting for results**: The main agent should be IDLE and responsive, not blocked on a tool call.
- **If you have nothing to do while waiting**: That's fine — just don't block. The user's next message is more important than any background result.

**Anti-pattern (BLOCKING — never do this):**
```
# WRONG: Main agent runs Nia directly and blocks for 2 minutes
bash ./scripts/search.sh deep "query" (timeout 120000)
```

**Correct pattern:**
```
# RIGHT: Delegate to librarian/explore subagent which runs Nia
task(subagent_type='librarian', run_in_background=true, prompt='Use Nia search.sh deep to research X')
# Main agent is now FREE to take user messages
```

## Background Task Handling & Context Window Protection

**NEVER use `block=true` when calling `background_output`.** Always `block=false`. No exceptions.

### Why `block=true` is banned
- The orchestrator must ALWAYS be available to receive user messages
- No subagent runs without the orchestrator dispatching it — there is never a scenario where you're idle waiting
- The system sends `[BACKGROUND TASK COMPLETED]` notifications — you never need to poll or wait
- If a result isn't ready yet, move on. Do research, exploration, or just be available. Come back when notified.

### Rules
- Launch background agents with `run_in_background=true`
- Wait for system `[BACKGROUND TASK COMPLETED]` notifications
- Only call `background_output` after receiving the completion notification, ALWAYS with `block=false`
- When the user sends a message while tasks are running, respond to the user FIRST, then collect results
- When working on long multi-step plans, use `run_in_background=true` for parallel exploration tasks so the main agent remains free to immediately respond to new user messages

**The main agent must NEVER do work that can be delegated to subagents.** The main agent's context window is the most precious resource — it must remain clean, available, and focused on:

- Orchestrating work and dispatching to subagents
- Receiving and responding to user input immediately
- High-level decision-making and synthesis
- Routing tasks to the right specialist (explore, librarian, quick, deep, etc.)

**NEVER let the main agent:**
- Read large files when an explore agent can summarize them
- Do implementation work that a `task(category='quick')` or `task(category='deep')` can handle
- Research topics when a librarian agent can do it in the background
- Accumulate tool output noise from sequential grep/read/glob calls — delegate to explore instead

**The main agent is the CEO, not the intern.** It delegates, synthesizes, and decides. Subagents do the heavy lifting. This is exactly how ohmyClaude designed the system to work — protect the orchestrator's context at all costs.

## Delegation Protocol

- Always use `delegate_task()` for implementation work
- One task per delegation — never batch multiple tasks
- Use `session_id` for retries and follow-ups (preserves context, saves tokens)
- Store every `session_id` returned from delegations

## Developer & Role Context

- The developer is a **software engineer**, NOT a compliance officer, legal advisor, or AML specialist
- Joined Blue Pearl Mortgage ~5 months ago (September 2025)
- The developer's boss tells them to build features and "make things compliant" — but compliance decisions, legal obligations, and regulatory filings are the brokerage's responsibility, not the developer's
- Do NOT ask the developer questions about FINTRAC filing processes, sanctions screening workflows, or compliance officer appointments — they won't know and it's not their job
- When researching regulatory compliance, focus on what the SOFTWARE needs to do, not what the BUSINESS needs to do (the developer can only control the software)

## AML Feature Context

- The AML report feature is a **PRODUCTIVITY TOOL**, not a compliance system
- It reads credit bureau PDFs (Equifax), extracts text using a PDF parser (NOT an LLM), then sends the extracted text to an LLM for analysis
- The LLM scores deals as low/medium/high AML risk based on credit bureau data — saving brokers 15-25 minutes of manual Word template work per deal
- This feature does NOT handle: FINTRAC filings, STR reports, sanctions screening, KYC verification, PEP/HIO checks, or any other compliance workflow
- The brokerage (Blue Pearl Mortgage) handles actual regulatory compliance separately from this software
- Blue Pearl became a FINTRAC reporting entity on October 11, 2024 (mortgage brokers were added to PCMLTFA)
- The brokerage uses an app called **Velocity** for some compliance processes — this is separate from the portal being built
- When analyzing the AML feature, evaluate it as a productivity/time-saving tool, NOT as a compliance system
- Key compliance risk: the feature's output could be MISTAKEN for a formal compliance document by auditors if proper disclaimers are missing
- **BEFORE making any compliance claims about this feature, READ** `.sisyphus/docs/aml-regulatory-analysis.md` — it contains all legal citations and research so you don't waste tokens re-researching
- The report text (phrases like "Overall Risk Rating", "Identity verified through Equifax") was taken from the brokers' existing Word template — this is how they've always worked
- "Identity verified through Equifax" is a legally prescribed verification method: SOR/2002-184, s. 105(1)(c)
- No Canadian law requires labeling AI-generated internal documents (AIDA/Bill C-27 is dead, BCFSA AI Guideline applies to RESA only)

## Project Context

- Enterprise internal tool ("BP Portal") for Blue Pearl Mortgage Group Inc. — a BC mortgage brokerage in Surrey with ~27 staff and ~50 brokers (reliability critical)
- Next.js 16 + Supabase + Inngest
- Developer is in Vietnam, Supabase servers in US (~300ms round trip per call)
- Use shadcn/ui components for all UI
- Use `&apos;` in JSX markup, not `'`
- Never read .env files
- Never create branches
- Production domain: https://www.bpportal.ca
- For Supabase Realtime, use `createRealtimeClient()` (anon key) NOT `createClient()` (publishable key) — publishable key has a known bug with Realtime subscriptions

## Debug-First Development

Always include server-side logging when building new features. Don't wait for bugs to add observability.

### Server-Side (always include)
- Add `console.log` with a `[TAG]` prefix for every new feature (e.g., `[AML]`, `[SYNC]`, `[AUTH]`)
- Log key decision points: function entry, external API calls, database writes, error paths
- Include relevant IDs in logs (reportId, dealId, userId) for traceability
- Log timing for operations that hit external services (Supabase, Gemini, Velocity API)
- These logs are visible in Vercel/deployment logs — the developer can check them without asking users

### Client-Side (errors only)
- Do NOT add verbose console.log to client components in production
- Use `toast.error()` with a clear message when something fails — users can screenshot the toast
- For debugging during development, add temporary `console.log` with `[DEBUG]` prefix and remove before committing
- If a feature involves real-time connections (WebSocket, Realtime), add connection status logging during development

### Lifecycle
1. Build feature WITH server-side logging from the start
2. During development, add temporary client-side `[DEBUG]` logs as needed
3. Once feature is verified working, remove `[DEBUG]` client logs but keep server-side logs
4. Server-side logs stay permanently — they cost nothing and save hours during incidents

## End-of-Session Protocol (CRITICAL — NON-NEGOTIABLE)

The developer works from Vietnamese internet cafes and temporary machines that get wiped between sessions. **NOTHING can ever be lost.**

At the end of EVERY session, without exception:

```bash
git add -A && git commit -m "chore: end-of-session commit" && git push origin main
```

Then verify `git status` shows a clean working tree before ending.

**This includes ALL agent infrastructure files** — `.agents/`, `.claude/`, `.sisyphus/`, `skills-lock.json` — everything tracked in git.

**NEVER end a session with a dirty working tree.**