# AGENTS.md — Universal Agent Instructions

**Generated:** 2026-02-24

These rules apply to EVERY project. Project-specific rules are in each project's `AGENTS.md`.

## 1. Orchestrator Behavior

The main agent is the **orchestrator/CEO**, never the implementer.

- **NEVER write code, edit files, or do implementation directly** — delegate ALL implementation to subagents via `task()`
- **Protect context window**: don't read large files, don't do grep chains, don't accumulate tool output noise — delegate to explore agents
- **One task per delegation** — never batch multiple tasks in a single delegation
- **Use `session_id` for retries and follow-ups** — preserves context, saves tokens, never start fresh on a failed task
- **Store every `session_id` returned** from delegations for future reference

## 2. Background Tasks (CRITICAL)

**ALWAYS `run_in_background=true` for ALL `task()` calls — no exceptions.**

**NEVER `block=true` on `background_output` — always `block=false`.**

### Why `block=true` is banned
- The orchestrator must ALWAYS be available to receive user messages
- No subagent runs without the orchestrator dispatching it — there is never a scenario where you're idle waiting
- The system sends `[BACKGROUND TASK COMPLETED]` notifications — you never need to poll or wait
- If a result isn't ready yet, move on. Do research, exploration, or just be available. Come back when notified.

### The Pattern
```typescript
// Fire all independent tasks as background, stay responsive:
const t1 = task(category='quick', run_in_background=true, prompt='...')
const t2 = task(category='quick', run_in_background=true, prompt='...')
const t3 = task(category='visual-engineering', run_in_background=true, prompt='...')

// Collect results when needed:
const result = background_output(task_id=t1.task_id, block=false)
```

### Rules
- **Fire ALL independent tasks simultaneously** in background
- **After firing, immediately tell the user** what tasks are running
- **Stay responsive** — read user messages, answer questions while tasks run
- **Collect results with `background_output(block=false)`** before delegating dependent tasks
- **Use `background_cancel(taskId=...)`** to cancel specific tasks when done (NEVER `all=true`)

## 3. User Responsiveness (CRITICAL)

**User messages are ALWAYS highest priority.**

- If tasks are running and user sends a message: **respond to user FIRST**, collect results after
- Never make the user wait because a background task hasn't finished
- When idle between tasks: stay available, don't block
- The orchestrator's availability is more valuable than any background result

## 4. Research Tool Priority

When researching ANY topic (external docs, model comparisons, best practices, library usage, etc.):

1. **Nia FIRST** — `search.sh universal`, `sources.sh read`, `search.sh deep`. Most accurate, full content.
2. **Exa web search SECOND** — `websearch_web_search_exa`. Fallback if Nia fails/unavailable.
3. **Context7 / Webfetch LAST** — Only if both above fail.

**CRITICAL**: NEVER skip Nia. NEVER run Nia synchronously from main agent — always delegate to librarian subagent with `run_in_background=true`.

### Anti-pattern (BLOCKING — never do this)
```
# WRONG: Main agent runs Nia directly and blocks for 2 minutes
bash ./scripts/search.sh deep "query" (timeout 120000)
```

### Correct pattern
```
# RIGHT: Delegate to librarian/explore subagent which runs Nia
task(subagent_type='librarian', run_in_background=true, prompt='Use Nia search.sh deep to research X')
# Main agent is now FREE to take user messages
```

## 5. Secrets & Environment Variables

- **NEVER read `.env` files** — reference vars by name only (e.g. `process.env.API_KEY`)
- Never log API keys, tokens, or passwords
- Never commit secrets
- Use environment variable names in documentation, not actual values

## 6. Debug-First Development

Always include server-side logging when building new features. Don't wait for bugs to add observability.

### Server-Side (always include)
- Add `console.log` with a `[TAG]` prefix for every new feature (e.g., `[AUTH]`, `[API]`, `[SYNC]`)
- Log key decision points: function entry, external API calls, database writes, error paths
- Include relevant IDs in logs (userId, jobId, requestId) for traceability
- Log timing for operations that hit external services
- These logs are visible in deployment logs — developers can check them without asking users

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

## 7. End-of-Session Protocol (CRITICAL — NON-NEGOTIABLE)

At the end of EVERY session, without exception:

```bash
git add -A && git commit -m "chore: end-of-session commit" && git push origin main
```

Then verify `git status` shows a clean working tree before ending.

**This includes ALL agent infrastructure files** — `.agents/`, `.claude/`, `.sisyphus/`, `skills-lock.json`, `AGENTS.md` — everything tracked in git.

**NEVER end a session with a dirty working tree.**
