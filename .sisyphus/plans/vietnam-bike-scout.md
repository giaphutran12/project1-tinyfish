# Vietnam Bike Price Scout — Build Plan

## TL;DR

> **Quick Summary**: Build a Next.js app that uses TinyFish Mino API to scrape 18 Vietnam motorbike rental websites in parallel via SSE streaming, displaying a real-time comparison dashboard. Deploy to Vercel, record demo, submit PR to tinyfish-io/tinyfish-cookbook.
> 
> **Deliverables**:
> - Working Next.js app deployed to Vercel
> - Live demo URL
> - Demo video (30-60s screen recording)
> - App screenshot
> - PR to tinyfish-io/tinyfish-cookbook with README
> - Social post via tinyfish-social.vercel.app
> 
> **Estimated Effort**: Short (4-6 hours)
> **Parallel Execution**: YES — 3 waves + final verification
> **Critical Path**: Task 1 (validate API) → Task 3 (SSE proxy) → Task 4 (frontend hook) → Task 6 (cards + grid) → Task 8 (deploy) → Task 10 (PR)

---

## Context

### Original Request
Build "Vietnam Bike Price Scout" — a motorbike rental price comparison tool for Vietnam. Users pick a city, Mino scrapes 15-18 rental shop websites in parallel, results stream in real-time to a comparison dashboard.

### Interview Summary
**Key Discussions**:
- Use case chosen for: zero API territory, Vietnam local advantage, zero CAPTCHA risk, strong parallel demo
- 18 verified real websites across 5 cities (HCMC, Hanoi, Da Nang/Hoi An, Nha Trang/Mui Ne)
- User is strong frontend dev, using Claude Code as primary tool
- Env var: `TINYFISH_API_KEY` (already in .env, never read it)
- Building in current repo (project1-tinyfish), deploying to Vercel
- Need to fork tinyfish-io/tinyfish-cookbook for PR

**Research Findings**:
- Correct API URL: `https://agent.tinyfish.ai/v1/automation/run-sse` (NOT mino.ai)
- SSE events: STARTED → STREAMING_URL → PROGRESS → HEARTBEAT → COMPLETE
- Excess requests are queued (PENDING), not rejected — stagger 1-2s recommended
- Vercel Hobby has 10s function timeout — MUST use Edge Runtime for streaming
- Cookbook pattern: ReadableStream + getReader() + buffer accumulation

### Metis Review
**Identified Gaps** (all addressed in this plan):
- Brief's code had 3 bugs: wrong API URL, buffered SSE, env var mismatch — all fixed
- Vercel timeout requires Edge Runtime — added as architectural guardrail
- Need per-request AbortController (90s timeout) — included in Task 3
- Need site validation before building UI — Task 1
- City count mismatch (4 keys vs 5 buttons) — resolved: 4 city groups
- Nha Trang only has 2 sites — flagged, show "limited results" if needed

---

## Work Objectives

### Core Objective
Ship a working Mino-powered motorbike rental price comparison app that demonstrates TinyFish's parallel web scraping capability with real-time SSE streaming.

### Concrete Deliverables
- Next.js 16 (React 19) app with Edge Runtime SSE proxy API route
- Comparison dashboard with city selector, bike cards, real-time streaming
- Vercel deployment at a live URL
- Demo video showing HCMC search completing with cards populating live
- PR to tinyfish-io/tinyfish-cookbook with 7-section README
- Social post via tinyfish-social.vercel.app

### Definition of Done
- [ ] `npm run build` exits cleanly (zero TypeScript errors)
- [ ] Live Vercel URL returns the app
- [ ] Searching HCMC returns ≥1 shop with bike data
- [ ] Results stream in real-time (cards appear one by one, not all at once)
- [ ] PR opened to tinyfish-io/tinyfish-cookbook with correct folder structure

### Must Have
- Real-time SSE streaming (cards populate as agents complete)
- City selector (HCMC, Hanoi, Da Nang, Nha Trang)
- Bike cards showing: name, type, daily/weekly/monthly price, deposit, shop name
- "Book Now" link to original rental site on each card
- Error handling: unavailable shops show graceful fallback
- Edge Runtime on API route (Vercel timeout protection)
- Loading state: progress indicator ("3/5 shops loaded")

### Must NOT Have (Guardrails)
- No map integration
- No user accounts / auth / persistence
- No database (no Supabase, Postgres, Redis)
- No caching layer
- No free-text search (city selector only)
- No dark mode
- No more than 30 minutes on loading animations
- No `await response.text()` for SSE parsing (MUST use getReader() stream)
- No direct Mino calls from frontend (API key stays server-side)
- Do NOT use brief's code snippet verbatim — it has 3 confirmed bugs

---

## Verification Strategy

> **ZERO HUMAN INTERVENTION** — ALL verification is agent-executed.

### Test Decision
- **Infrastructure exists**: NO (greenfield)
- **Automated tests**: None (MVP, 4-6h timeline)
- **Framework**: N/A

### QA Policy
Every task MUST include agent-executed QA scenarios.
Evidence saved to `.sisyphus/evidence/task-{N}-{scenario-slug}.{ext}`.

- **Frontend/UI**: Use Playwright (playwright skill) — Navigate, interact, assert DOM, screenshot
- **API/Backend**: Use Bash (curl) — Send requests, assert status + response fields
- **Build**: Use Bash — `npm run build`, check exit code

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (Foundation — start immediately):
├── Task 1: Validate Mino API against real sites [quick]
├── Task 2: Scaffold Next.js + Tailwind + shadcn/ui [quick]
└── (Task 1 and 2 can run in parallel)

Wave 2 (Core — after Wave 1):
├── Task 3: Build SSE proxy API route (Edge Runtime) [deep]
├── Task 4: Build frontend SSE consumption hook [quick]
└── Task 5: Build city selector + search UI [visual-engineering]

Wave 3 (Integration + Polish — after Wave 2):
├── Task 6: Build bike card component + results grid [visual-engineering]
├── Task 7: Error handling + edge cases + normalization [quick]
└── Task 8: Deploy to Vercel + verify live [quick]

Wave 4 (Deliverables — after Wave 3):
├── Task 9: Record demo video + take screenshot [quick]
├── Task 10: Fork cookbook + prepare PR + README [quick]
├── Task 11: Generate PRD doc [quick]
└── Task 12: Create social post [quick]

Wave FINAL (Verification — after ALL tasks):
├── F1: Plan compliance audit [oracle]
├── F2: Code quality review [unspecified-high]
├── F3: Real manual QA [unspecified-high]
└── F4: Scope fidelity check [deep]

Critical Path: T1 → T3 → T4 → T6 → T8 → T10
Parallel Speedup: ~50% faster than sequential
Max Concurrent: 3 (Waves 2 & 3)
```

### Dependency Matrix

| Task | Depends On | Blocks |
|------|-----------|--------|
| 1 | — | 3, 7 |
| 2 | — | 3, 4, 5 |
| 3 | 1, 2 | 4, 6, 7, 8 |
| 4 | 2, 3 | 6 |
| 5 | 2 | 6 |
| 6 | 3, 4, 5 | 7, 8 |
| 7 | 3, 6 | 8 |
| 8 | 6, 7 | 9, 10 |
| 9 | 8 | 10 |
| 10 | 8, 9 | — |
| 11 | 8 | — |
| 12 | 8 | — |

### Agent Dispatch Summary

- **Wave 1**: 2 tasks — T1 `quick`, T2 `quick`
- **Wave 2**: 3 tasks — T3 `deep`, T4 `quick`, T5 `visual-engineering`
- **Wave 3**: 3 tasks — T6 `visual-engineering`, T7 `quick`, T8 `quick`
- **Wave 4**: 4 tasks — T9-T12 all `quick`
- **FINAL**: 4 tasks — F1 `oracle`, F2 `unspecified-high`, F3 `unspecified-high`, F4 `deep`

---

## TODOs

### Wave 1: Foundation

- [ ] **Task 1: Validate Mino API against real sites** — `quick` + `Nia` skill
  - **Goal**: Confirm the API works and returns usable bike pricing data from real sites before building anything.
  - **Steps**:
    1. Use env var `TINYFISH_API_KEY` (already in .env — DO NOT read the file, just reference the var name)
    2. Send 1 test request per city to a simple target site:
       - HCMC: `https://motorbikerentaldanang.com/` (simple, prices on homepage)
       - Hanoi: `https://motorbikerentalinhanoi.com/`
       - Da Nang: `https://hoianbikerental.com/pricing/`
       - Nha Trang: `https://motorbikemuine.com/`
    3. Use the SSE endpoint: `https://agent.tinyfish.ai/v1/automation/run-sse`
    4. Use the exact Mino goal prompt from `use-case-brief.md` (the 5-step prompt)
    5. Log the `streamingUrl` from each response for debugging
    6. Record which sites return valid bike data and which fail
  - **API Details** (from Nia-indexed docs, source `123d2659-98bb-4843-8a26-f6c6f4c09a30`):
    - Endpoint: `POST https://agent.tinyfish.ai/v1/automation/run-sse`
    - Auth header: `X-API-Key: <TINYFISH_API_KEY>`
    - Body: `{ "url": "...", "goal": "..." }`
    - SSE events flow: STARTED → STREAMING_URL → PROGRESS → HEARTBEAT → COMPLETE
    - Parse SSE by reading lines starting with `data: `, JSON-parse each
    - Final event has `status: "COMPLETED"` and `resultJson` with the extracted data
  - **MUST DO**: Save raw API responses to `.sisyphus/evidence/task-1-api-validation.json`
  - **MUST DO**: Record which sites work and which fail — this determines the final CITY_SITES config
  - **MUST NOT**: Build any UI. This is API validation only.
  - **MUST NOT**: Read the .env file. Just use `process.env.TINYFISH_API_KEY` in code.
  - **QA**: At least 1 site per city returns valid JSON with `bikes` array containing ≥1 entry
  - **Output**: Validation report + working curl commands saved to evidence

- [ ] **Task 2: Scaffold Next.js + Tailwind + shadcn/ui** — `quick`
  - **Goal**: Create the project foundation — Next.js 16 + React 19 + Tailwind v4 + shadcn/ui ready to use.
  - **Steps**:
    1. Run `npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"` (installs Next.js 16, React 19, Tailwind v4, TS 5.9 — all latest)
    2. Initialize shadcn/ui: `npx shadcn@latest init`
    3. Add components we'll need: `npx shadcn@latest add button card select badge skeleton`
    4. Create `.env.example` with `TINYFISH_API_KEY=your_api_key_here`
    5. Add `.env` to `.gitignore` (if not already there)
    6. Update `package.json` name to `vietnam-bike-scout`
    7. Verify `npm run dev` starts without errors
    8. Verify `npm run build` exits cleanly
  - **MUST DO**: Use Next.js App Router (not Pages Router)
  - **MUST DO**: Keep existing files (AGENTS.md, use-case-brief.md, .sisyphus/) — don't delete them
  - **MUST NOT**: Install any database, auth, or state management libraries
  - **MUST NOT**: Create any app pages yet — just the scaffold
  - **QA**: `npm run build` exits with code 0
  - **Output**: Working Next.js app skeleton that starts and builds cleanly

### Wave 2: Core Architecture

- [ ] **Task 3: Build SSE proxy API route (Edge Runtime)** — `deep` + `Nia` skill
  - **Goal**: Build the critical backend — an Edge Runtime API route that fans out parallel Mino requests and streams results back to the client as SSE events.
  - **This is the hardest task. Get it right.**
  - **File**: `src/app/api/search/route.ts`
  - **Architecture** (two-layer SSE proxy):
    ```
    Client POST /api/search { city: "hcmc" }
    → API route creates ReadableStream
    → Fires N parallel Mino SSE requests (one per site in that city)
    → As each Mino stream reaches COMPLETE, enqueue custom SSE event to client:
       data: { "type": "SHOP_RESULT", "shop": { ...resultJson } }
    → After all complete (or timeout), send final event:
       data: { "type": "SEARCH_COMPLETE", "total": N, "succeeded": M, "elapsed": "12.3s" }
    → Close stream
    ```
  - **Critical Implementation Details**:
    1. `export const runtime = 'edge'` — MANDATORY (Vercel Hobby has 10s function timeout, Edge has none for streaming)
    2. Use `getReader()` + buffer pattern to consume Mino SSE streams (NOT `await response.text()`)
    3. Add `AbortController` with 90s timeout per Mino request
    4. Stagger requests 500ms apart (API docs recommend not firing all at once)
    5. Use `Promise.allSettled()` — one failed site must NOT kill the whole search
    6. Return `new Response(readableStream, { headers: { 'Content-Type': 'text/event-stream', ... } })`
  - **CITY_SITES config**: Use the validated sites from Task 1. Hardcode the URL map:
    ```typescript
    const CITY_SITES: Record<string, string[]> = {
      hcmc: ["https://www.tigitmotorbikes.com/prices", ...],
      hanoi: ["https://motorbikerentalinhanoi.com/", ...],
      danang: ["https://motorbikerentaldanang.com/", ...],  // includes Hoi An sites
      nhatrang: ["https://moto4free.com/", ...],
    };
    ```
  - **SSE parsing pattern** (from cookbook — DO NOT use `await response.text()`):
    ```typescript
    const reader = response.body!.getReader();
    const decoder = new TextDecoder();
    let buffer = '';
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      buffer += decoder.decode(value, { stream: true });
      const lines = buffer.split('\n');
      buffer = lines.pop()!;  // keep incomplete line in buffer
      for (const line of lines) {
        if (line.startsWith('data: ')) {
          const event = JSON.parse(line.slice(6));
          // process event...
        }
      }
    }
    ```
  - **Env var**: `process.env.TINYFISH_API_KEY` — Edge Runtime supports env vars on Vercel
  - **MUST DO**: Log `[MINO] Starting: ${url}` and `[MINO] Complete: ${url} (${elapsed}s)` for every request
  - **MUST DO**: Log `streamingUrl` when received (debug lifesaver)
  - **MUST NOT**: Use `await response.text()` to parse SSE — this buffers the entire stream and defeats real-time UX
  - **MUST NOT**: Use `mino.ai` as the API URL — correct URL is `agent.tinyfish.ai`
  - **MUST NOT**: Expose the API key to the client — key stays server-side in the Edge function
  - **QA**: `curl -N -X POST http://localhost:3000/api/search -H 'Content-Type: application/json' -d '{"city":"hcmc"}' --max-time 120` returns SSE events with `SHOP_RESULT` type
  - **Reference**: Study `tinyfish-io/tinyfish-cookbook/competitor-analysis/app/api/scrape-pricing/route.ts` for the canonical pattern

- [ ] **Task 4: Build frontend SSE consumption hook** — `quick`
  - **Goal**: Create a React hook that consumes the SSE stream from `/api/search` and provides reactive state for the UI.
  - **File**: `src/hooks/use-bike-search.ts`
  - **Interface**:
    ```typescript
    interface BikeShop {
      shop_name: string;
      city: string;
      website: string;
      bikes: Bike[];
      notes: string | null;
    }
    interface Bike {
      name: string;
      engine_cc: number | null;
      type: 'scooter' | 'semi-auto' | 'manual' | 'adventure';
      price_daily_usd: number | null;
      price_weekly_usd: number | null;
      price_monthly_usd: number | null;
      currency: string;
      deposit_usd: number | null;
      available: boolean;
    }
    interface SearchState {
      shops: BikeShop[];
      isSearching: boolean;
      progress: { completed: number; total: number };
      error: string | null;
      elapsed: string | null;
    }
    function useBikeSearch(): {
      state: SearchState;
      search: (city: string) => void;
      abort: () => void;
    }
    ```
  - **Key behaviors**:
    1. On `search(city)`: abort any in-flight request, POST to `/api/search`, consume SSE stream
    2. On each `SHOP_RESULT` event: append shop to `shops` array, increment `progress.completed`
    3. On `SEARCH_COMPLETE` event: set `isSearching = false`, set `elapsed`
    4. On error/timeout: set `error` message, set `isSearching = false`
    5. On component unmount: abort via `AbortController`
  - **Data normalization**: Add a `normalizeShop(raw: any): BikeShop` function that:
    - Coerces price strings to numbers
    - Defaults missing fields to null
    - Ensures `bikes` is always an array (even if Mino returns object)
    - Filters out bikes with no name
  - **MUST DO**: Use `AbortController` for cleanup on unmount and new search
  - **MUST NOT**: Call Mino API directly from the hook — only call `/api/search`
  - **QA**: Import hook in a test page, trigger search, verify `shops` array grows incrementally (not all at once)

- [ ] **Task 5: Build city selector + search UI** — `visual-engineering` + `frontend-ui-ux` skill
  - **Goal**: Build the main page layout — hero section, city selector buttons, search trigger, and container for results.
  - **File**: `src/app/page.tsx`
  - **Layout**:
    ```
    ┌─────────────────────────────────────────────┐
    │  🛵 Vietnam Bike Price Scout                │
    │  Compare motorbike rental prices across      │
    │  Vietnam in seconds, not hours.              │
    │                                              │
    │  [HCMC] [Hanoi] [Da Nang] [Nha Trang]      │
    │                                              │
    │  Progress: ████░░░░ 3/5 shops loaded         │
    │                                              │
    │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐       │
    │  │Card 1│ │Card 2│ │Card 3│ │Card 4│       │
    │  └──────┘ └──────┘ └──────┘ └──────┘       │
    └─────────────────────────────────────────────┘
    ```
  - **Components needed**: City selector (4 buttons, active state), progress bar, results container
  - **Use shadcn/ui**: `Button` for city selector, `Badge` for bike type tags, `Card` for results container
  - **Wire up** the `useBikeSearch` hook from Task 4
  - **MUST DO**: City buttons disable during active search
  - **MUST DO**: Show skeleton cards while loading (shimmer effect)
  - **MUST NOT**: Add a map, dark mode toggle, or any extra chrome
  - **QA**: Page renders, clicking a city shows loading state, layout doesn't break on mobile

### Wave 3: Integration + Polish

- [ ] **Task 6: Build bike card component + results grid** — `visual-engineering` + `frontend-ui-ux` skill
  - **Goal**: Build the bike card component that displays individual bike listings, grouped by shop.
  - **Files**: `src/components/bike-card.tsx`, `src/components/shop-group.tsx`, `src/components/results-grid.tsx`
  - **Shop Group layout**:
    ```
    ┌─────────────────────────────────────────────────┐
    │  🏪 Tigit Motorbikes — Ho Chi Minh City        │
    │  tigitmotorbikes.com · Helmet + phone included  │
    │                                                  │
    │  ┌──────────────┐ ┌──────────────┐ ┌────────┐  │
    │  │Honda Blade 110│ │Yamaha NVX 155│ │Honda XR│  │
    │  │semi-auto      │ │scooter       │ │manual  │  │
    │  │$8/day         │ │$12/day       │ │$15/day │  │
    │  │$120/mo        │ │$180/mo       │ │$250/mo │  │
    │  │Deposit: $100  │ │Deposit: $150 │ │Dep:$200│  │
    │  │ [Book Now] ✅ │ │ [Book Now] ✅│ │[Book]✅│  │
    │  └──────────────┘ └──────────────┘ └────────┘  │
    └─────────────────────────────────────────────────┘
    ```
  - **Bike card shows**: name, type badge (color-coded), daily/weekly/monthly price, deposit, availability, "Book Now" link to original site
  - **Results grid**: shops appear one at a time as SSE events arrive (animated entrance)
  - **Sort**: cheapest daily price first (across all shops, flatten bikes)
  - **Use shadcn/ui**: `Card`, `Badge`, `Button` (for Book Now links), `Skeleton` (for loading)
  - **MUST DO**: "Book Now" opens original rental site in new tab (`target="_blank"`)
  - **MUST DO**: Show "unavailable" badge if `available: false`
  - **MUST DO**: Show "No pricing found" card if a shop returns empty bikes array
  - **MUST NOT**: Spend more than 30 min on animations. Simple fade-in is fine.
  - **QA**: Cards render with real data from API, "Book Now" links work, responsive on mobile

- [ ] **Task 7: Error handling + edge cases** — `quick`
  - **Goal**: Handle all failure modes gracefully so the demo never shows a broken state.
  - **Edge cases to handle**:
    1. **Site is down**: Show card with "⚠️ [Shop Name] — Temporarily unavailable" + link to site
    2. **All sites fail for a city**: Show "No results found. Try another city or try again."
    3. **Mino returns unexpected JSON**: `normalizeShop()` from Task 4 handles this — ensure it doesn't throw
    4. **Prices in VND not converted**: Frontend fallback — if price > 1000, assume VND, divide by 25000
    5. **Search while previous search is running**: `abort()` previous, start new (already handled in hook)
    6. **SSE connection drops**: 120s overall timeout, show "Search timed out — showing partial results"
    7. **Nha Trang returns 0 results**: Show message "Limited coverage in this city. More shops coming soon."
  - **MUST DO**: Every error state has a user-friendly message (no raw error strings)
  - **MUST DO**: Failed shops don't break the search — other shops still display
  - **MUST NOT**: Add retry buttons (keep it simple for MVP)
  - **QA**: Intentionally test with a bad URL in CITY_SITES to verify graceful failure

- [ ] **Task 8: Deploy to Vercel + verify live** — `quick`
  - **Goal**: Get the app live on a public URL.
  - **Steps**:
    1. `npm run build` — must exit cleanly
    2. Connect repo to Vercel (via `vercel` CLI or Vercel dashboard)
    3. Set environment variable `TINYFISH_API_KEY` in Vercel project settings
    4. Deploy: `vercel --prod`
    5. Verify live URL loads the app
    6. Test a HCMC search on the live URL — verify SSE streaming works
  - **MUST DO**: Verify Edge Runtime works on Vercel (not just localhost)
  - **MUST DO**: Test at least 2 cities on the live URL
  - **MUST NOT**: Include .env in the deployment (Vercel env vars only)
  - **QA**: `curl -s https://<vercel-url> | grep -q 'Vietnam Bike'` succeeds, SSE search returns data

### Wave 4: Deliverables

- [ ] **Task 9: Record demo video + screenshot** — `quick` + `playwright` skill
  - **Goal**: Record a 30-60s screen recording showing the app in action + take an app screenshot.
  - **Demo video should show**:
    1. App loads on Vercel URL
    2. Click "HCMC" city button
    3. Watch cards populate in real-time as agents complete (the money shot)
    4. Scroll through results showing different shops and bikes
    5. Click a "Book Now" link to show it opens the real rental site
  - **Screenshot**: Clean view of the app with results loaded (for TinyFish website showcase)
  - **Use Playwright skill** for automated screenshot if needed
  - **Output**: Demo video file + screenshot saved to repo

- [ ] **Task 10: Fork cookbook + prepare PR** — `quick` + `git-master` skill
  - **Goal**: Fork tinyfish-io/tinyfish-cookbook and prepare the PR with correct folder structure.
  - **Steps**:
    1. Fork `tinyfish-io/tinyfish-cookbook` to user's GitHub account
    2. Clone the fork
    3. Create directory `vietnam-bike-price-scout/`
    4. Copy app source code into the directory
    5. Create `README.md` with the 7 required sections:
       - Title + live link
       - Short 2-3 liner description
       - Demo video (gif or embedded)
       - Code snippet calling TinyFish API
       - How to Run (env vars, setup steps)
       - Architecture diagram (Mermaid format)
    6. Include `.env.example`
    7. Open PR from fork to `tinyfish-io/tinyfish-cookbook` main branch
  - **Folder naming**: kebab-case (`vietnam-bike-price-scout/`) — matches cookbook convention
  - **MUST DO**: PR description summarizes the use case clearly
  - **MUST NOT**: Include .env with real API key
  - **QA**: PR is openable on GitHub, README renders correctly, no secrets committed

- [ ] **Task 11: Generate PRD doc** — `quick`
  - **Goal**: Generate the Duplicable PRD using the SOP's exact prompt.
  - **Use this exact prompt** (from SOP):
    > "Please generate a standard developer documentation example for the Mino API use case I've developed."
  - **PRD must contain**:
    1. Product Architecture Overview
    2. Code Snippet (runnable)
    3. Goal/Prompt (exact Mino prompt)
    4. Sample Output (streaming JSON)
  - **Output**: PRD markdown file added to the cookbook PR

- [ ] **Task 12: Create social post** — `quick`
  - **Goal**: Create a social post using TinyFish's post generator tool.
  - **Tool**: https://tinyfish-social.vercel.app/
  - **Post angle**: "I built a tool that checks 20 Vietnam motorbike rental sites in 30 seconds instead of 2 hours"
  - **Include**: Live demo link, demo video/gif, brief explanation of what Mino does
  - **Tone**: Helpful indie builder, not corporate marketing

## Final Verification Wave

> 4 review agents run in PARALLEL. ALL must APPROVE. Rejection → fix → re-run.

- [ ] F1. **Plan Compliance Audit** — `oracle`
  Read the plan end-to-end. For each "Must Have": verify implementation exists. For each "Must NOT Have": search codebase for forbidden patterns. Check evidence files in .sisyphus/evidence/. Compare deliverables against plan.
  Output: `Must Have [N/N] | Must NOT Have [N/N] | Tasks [N/N] | VERDICT: APPROVE/REJECT`

- [ ] F2. **Code Quality Review** — `unspecified-high`
  Run `npm run build`. Review all files for: `as any`/`@ts-ignore`, empty catches, console.log in prod code (keep [MINO] tagged ones), unused imports. Check for `await response.text()` anti-pattern. Verify Edge Runtime is set.
  Output: `Build [PASS/FAIL] | Files [N clean/N issues] | VERDICT`

- [ ] F3. **Real Manual QA** — `unspecified-high` (+ `playwright` skill)
  Start from clean state. Open live Vercel URL. Test each city search. Verify cards stream in real-time. Test error states (what happens with slow/failed sites). Screenshot evidence.
  Output: `Cities [N/4 work] | Streaming [YES/NO] | Edge Cases [N tested] | VERDICT`

- [ ] F4. **Scope Fidelity Check** — `deep`
  Verify: no map, no auth, no database, no caching, no dark mode. Check all "Must NOT Have" items. Verify PR folder structure matches cookbook convention. Flag unaccounted changes.
  Output: `Tasks [N/N compliant] | Scope [CLEAN/N issues] | VERDICT`

---

## Commit Strategy

- **After Task 2**: `feat: scaffold next.js app with tailwind + shadcn/ui`
- **After Task 3**: `feat: add mino SSE proxy API route with edge runtime`
- **After Tasks 4-5**: `feat: add city selector and SSE streaming hook`
- **After Tasks 6-7**: `feat: add bike cards, results grid, error handling`
- **After Task 8**: `feat: deploy to vercel, verify live`
- **After Tasks 10-12**: `docs: add cookbook README, PRD, prepare PR`

---

## Success Criteria

### Verification Commands
```bash
npm run build        # Expected: exit 0, zero errors
curl -s https://<vercel-url> | grep -q "Vietnam Bike"  # Expected: match
curl -s -N -X POST https://<vercel-url>/api/search -H "Content-Type: application/json" -d '{"city":"hcmc"}' --max-time 120 | head -5  # Expected: SSE data lines
```

### Final Checklist
- [ ] All "Must Have" present
- [ ] All "Must NOT Have" absent
- [ ] Live Vercel URL works
- [ ] Demo video recorded
- [ ] PR opened to tinyfish-io/tinyfish-cookbook
- [ ] Social post created
