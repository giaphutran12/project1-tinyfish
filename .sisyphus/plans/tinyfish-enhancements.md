# TinyFish Vietnam Bike Price Scout — Feature Enhancements

## TL;DR

> **Quick Summary**: Add a cache toggle (default OFF for demos) and a multi-select bike type filter to the Vietnam Bike Price Scout app, plus make bike cards clickable with per-bike URLs. Fix the silent type normalization bug so filters actually work.
> 
> **Deliverables**:
> - Cache toggle Switch (default OFF, live scraping for demos)
> - Multi-select pill-button filter for bike types (scooter, semi-auto, manual, adventure)
> - Type normalization safety net in `normalizeShop()`
> - Clickable bike cards with per-bike detail URLs (fallback to shop URL)
> - Hardened Mino prompt with type aliases + URL extraction
> 
> **Estimated Effort**: Short (3-4 hours parallel execution)
> **Parallel Execution**: YES — 3 waves + final verification
> **Critical Path**: Install Switch + Backend param → Frontend UI → Clickable Cards → Build verification

---

## Context

### Original Request
Sky Zhang (TinyFish interviewer) feedback: remove caching so the demo shows the TinyFish API working live. User's counter: add a toggle instead (default OFF for demos, ON for repeat searches). Also add bike type filtering (multi-select: scooter, semi-auto, manual, adventure) since the data already exists but no UI filter is built. Also make bike cards clickable to link to individual bike detail pages.

### Interview Summary
**Key Discussions**:
- Caching toggle preferred over removing caching entirely — better demo story (show live + show cache)
- Multi-select is required (e.g., "show me semi-auto AND scooter together")
- Bike type data already exists in the data model and Mino prompt, but normalization bug silently breaks it
- Clickable cards with per-bike URL extraction; fallback to shop.website for flat-page sites
- Live preview streaming (STREAMING_URL forwarding) explicitly deferred to future work

**Research Findings**:
- Vietnam rental sites use "Automatic" not "Scooter" — normalization bug is confirmed real
- TinyFish API OpenAPI spec confirms STREAMING_URL event type exists (back burner feature)
- rentabikevn.com categorizes bikes as: Manual, Semi-Auto, Automatic — matching our 4-type taxonomy
- shadcn/ui Switch component not yet installed; needs `npx shadcn@latest add switch`
- radix-ui umbrella v1.4.3 is installed — Switch primitive should be included

### Metis Review
**Identified Gaps** (addressed):
- `useCache=false` should skip BOTH cache read AND write (not just read)
- "No filters selected" must show all bikes (standard UX)
- Filter selection must reset when city changes
- Switch must be disabled during active search
- AGENTS.md says "MUST use Edge Runtime" but route.ts correctly uses `"nodejs"` — AGENTS.md is stale, do NOT change runtime
- `normalizeShop` empty string URL → must normalize to `null`
- Need to handle "0 filtered results" state distinctly from "0 search results"

---

## Work Objectives

### Core Objective
Make the Vietnam Bike Price Scout demo-ready for Sky Zhang by adding a cache toggle (default live) and a multi-select bike type filter, while fixing the underlying type normalization bug.

### Concrete Deliverables
- `src/components/ui/switch.tsx` — shadcn Switch component (installed)
- `src/app/api/search/route.ts` — Updated with `useCache` param + hardened GOAL_PROMPT
- `src/hooks/use-bike-search.ts` — Updated Bike interface (+ url field), type normalization, search signature
- `src/app/page.tsx` — Cache toggle Switch + bike type multi-select pills + filter logic
- `src/components/bike-card.tsx` — Clickable card wrapper with external link icon
- `src/components/shop-group.tsx` — Pass shopWebsite prop to BikeCard

### Definition of Done
- [ ] `npm run build` exits 0 with zero TypeScript errors
- [ ] Cache toggle defaults to OFF; when OFF, server logs show NO `[CACHE]` messages
- [ ] Selecting multiple bike types shows union of selected types
- [ ] Deselecting all types shows all bikes
- [ ] Bike cards link to individual detail URLs or fall back to shop URL

### Must Have
- Cache toggle with default OFF (live scraping mode)
- Multi-select bike type filter (1+ types selectable simultaneously)
- Type normalization safety net for Mino return value variants
- Clickable bike cards with per-bike URL fallback to shop URL
- `npm run build` passes after all changes

### Must NOT Have (Guardrails)
- ❌ DO NOT change `export const runtime = "nodejs"` to `"edge"` — despite AGENTS.md saying so. Supabase client requires Node.js runtime. AGENTS.md directive is stale.
- ❌ DO NOT add sorting, price range filtering, or duration toggles — out of scope
- ❌ DO NOT implement STREAMING_URL forwarding (Priority 3, back burner)
- ❌ DO NOT add test infrastructure — demo app, no tests needed
- ❌ DO NOT restructure the single-page layout into multiple routes/pages
- ❌ DO NOT add new npm dependencies beyond the shadcn Switch component
- ❌ DO NOT modify the SSE ReadableStream architecture in route.ts
- ❌ DO NOT change the Supabase schema or add migrations
- ❌ DO NOT touch `convertPrice()` logic (known issue but out of scope)
- ❌ DO NOT touch `layout.tsx` metadata (still says "Create Next App" — polish, not in scope)
- ❌ DO NOT move `normalizeShop()` to a separate file — keep it in `use-bike-search.ts`
- ❌ DO NOT add dark mode, theme toggles, or advanced styling
- ❌ DO NOT make unavailable bikes clickable (or at minimum, style them as clearly non-actionable)

---

## Verification Strategy

> **ZERO HUMAN INTERVENTION** — ALL verification is agent-executed. No exceptions.

### Test Decision
- **Infrastructure exists**: NO
- **Automated tests**: None — demo app
- **Framework**: None
- **QA approach**: Agent-executed build verification + Playwright for UI

### QA Policy
Every task MUST include agent-executed QA scenarios.
Evidence saved to `.sisyphus/evidence/task-{N}-{scenario-slug}.{ext}`.

- **Frontend/UI**: Use Playwright (playwright skill) — Navigate, interact, assert DOM, screenshot
- **Backend**: Use Bash (curl) — Send requests, assert status + response fields
- **Build**: Use Bash — `npm run build`, verify exit code 0

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (Start Immediately — no dependencies between tasks):
├── Task 1: Install shadcn Switch component [quick]
├── Task 2: Backend useCache parameter [quick]
└── Task 3: Type normalization fix in normalizeShop [quick]

Wave 2 (After Wave 1 — frontend UI wiring):
└── Task 4: Cache toggle Switch + multi-select bike type filter UI + hook wiring [unspecified-high]

Wave 3 (After Wave 2 — clickable cards):
└── Task 5: Clickable bike cards with URL extraction [unspecified-high]

Wave FINAL (After ALL tasks):
├── Task F1: Build + lint verification [quick]
├── Task F2: Plan compliance audit [oracle]
└── Task F3: UI QA with Playwright [unspecified-high]

Critical Path: Task 1+2 → Task 4 → Task 5 → F1-F3
Parallel Speedup: ~50% faster than sequential (Wave 1 runs 3 tasks simultaneously)
Max Concurrent: 3 (Wave 1)
```

### Dependency Matrix

| Task | Depends On | Blocks |
|------|-----------|--------|
| 1 (Switch install) | — | 4 |
| 2 (Backend useCache) | — | 4 |
| 3 (Type normalization) | — | 4, 5 |
| 4 (Frontend UI) | 1, 2, 3 | 5 |
| 5 (Clickable cards) | 3, 4 | F1, F2, F3 |
| F1 (Build verify) | 5 | — |
| F2 (Plan audit) | 5 | — |
| F3 (UI QA) | 5 | — |

### Agent Dispatch Summary

- **Wave 1**: **3 tasks** — T1 → `quick`, T2 → `quick`, T3 → `quick`
- **Wave 2**: **1 task** — T4 → `unspecified-high` + `frontend-ui-ux` skill
- **Wave 3**: **1 task** — T5 → `unspecified-high` + `frontend-ui-ux` skill
- **FINAL**: **3 tasks** — F1 → `quick`, F2 → `oracle`, F3 → `unspecified-high` + `playwright` skill

---

## TODOs

> Implementation + Test = ONE Task. Never separate.
> EVERY task MUST have: Recommended Agent Profile + Parallelization info + QA Scenarios.
> **A task WITHOUT QA Scenarios is INCOMPLETE. No exceptions.**

- [ ] 1. Install shadcn Switch component

  **What to do**:
  - Run `npx shadcn@latest add switch` in project root
  - Verify `src/components/ui/switch.tsx` is created with proper exports
  - Run `npm run build` to confirm no dependency conflicts with `radix-ui` v1.4.3
  - If `shadcn add` fails (Tailwind v4 compatibility), manually create a Switch component using Radix UI `@radix-ui/react-switch` primitives following the pattern in existing shadcn components (`button.tsx`, `badge.tsx`)

  **Must NOT do**:
  - Do NOT install any additional UI libraries
  - Do NOT modify existing shadcn components

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: `[]`
    - No special skills needed — simple package installation

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 2, 3)
  - **Blocks**: Task 4
  - **Blocked By**: None

  **References**:
  - `src/components/ui/button.tsx` — Existing shadcn component pattern to follow
  - `src/components/ui/badge.tsx` — Another shadcn component for reference
  - `package.json` line `"radix-ui": "^1.4.3"` — Umbrella package already installed
  - `components.json` — shadcn configuration file

  **Acceptance Criteria**:
  - [ ] File `src/components/ui/switch.tsx` exists
  - [ ] `npm run build` exits 0

  **QA Scenarios:**
  ```
  Scenario: Switch component installed and builds
    Tool: Bash
    Steps:
      1. Run: ls src/components/ui/switch.tsx
      2. Run: npm run build
    Expected Result: switch.tsx exists; build exits with code 0
    Evidence: .sisyphus/evidence/task-1-switch-install.txt
  ```

  **Commit**: YES
  - Message: `chore(ui): install shadcn Switch component`
  - Files: `src/components/ui/switch.tsx`, `package.json` (if deps changed)
  - Pre-commit: `npm run build`

- [ ] 2. Backend: Add `useCache` parameter to search API

  **What to do**:
  - In `src/app/api/search/route.ts`:
    1. Update `SearchBody` type (line 79-81) to add `useCache?: boolean`:
       ```typescript
       type SearchBody = {
         city: string;
         useCache?: boolean;
       };
       ```
    2. After line 269 (`const city = body.city?.toLowerCase()`), add:
       ```typescript
       const useCache = body.useCache ?? false;
       ```
    3. Wrap the cache-read block. Change line 286 from `if (supabase)` to:
       ```typescript
       if (supabase && useCache)
       ```
    4. Wrap the cache-write call inside `siteEnqueue`. Change line 343-345 from:
       ```typescript
       if (supabase && event.shop) {
         cacheResult(supabase, city, url, event.shop).catch(() => {});
       }
       ```
       to:
       ```typescript
       if (supabase && useCache && event.shop) {
         cacheResult(supabase, city, url, event.shop).catch(() => {});
       }
       ```
  - DO NOT change `export const runtime = "nodejs"` — Supabase requires Node.js
  - DO NOT change the SSE ReadableStream architecture
  - DO NOT modify the GOAL_PROMPT (that's Task 5)
  - DO NOT change the Mino API call logic

  **Must NOT do**:
  - Do NOT change the runtime from "nodejs" to "edge" (despite AGENTS.md saying so — it's stale)
  - Do NOT modify the SSE streaming architecture
  - Do NOT add cache-clearing or cache-management endpoints

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: `[]`
    - Simple conditional wrapping in one file

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 3)
  - **Blocks**: Task 4
  - **Blocked By**: None

  **References**:
  - `src/app/api/search/route.ts:79-81` — Current `SearchBody` type (only `city: string`)
  - `src/app/api/search/route.ts:269` — Where `city` is parsed from body
  - `src/app/api/search/route.ts:286-293` — Cache read block to wrap in `useCache` conditional
  - `src/app/api/search/route.ts:343-345` — Cache write call to wrap in `useCache` conditional
  - `src/app/api/search/route.ts:1` — `export const runtime = "nodejs"` — DO NOT CHANGE THIS

  **Acceptance Criteria**:
  - [ ] `SearchBody` type includes `useCache?: boolean`
  - [ ] Cache read block only executes when `useCache` is true
  - [ ] Cache write call only executes when `useCache` is true
  - [ ] `npm run build` exits 0

  **QA Scenarios:**
  ```
  Scenario: useCache=false skips all cache operations
    Tool: Bash (grep)
    Steps:
      1. Read route.ts and verify SearchBody includes useCache?: boolean
      2. Verify `if (supabase && useCache)` guard on cache read block
      3. Verify `if (supabase && useCache && event.shop)` guard on cache write
      4. Run: npm run build
    Expected Result: All conditions present; build passes
    Evidence: .sisyphus/evidence/task-2-backend-cache-toggle.txt

  Scenario: Default value is false when useCache omitted
    Tool: Bash (grep)
    Steps:
      1. Verify `const useCache = body.useCache ?? false;` in route.ts
    Expected Result: Default is false (live mode)
    Evidence: .sisyphus/evidence/task-2-default-false.txt
  ```

  **Commit**: YES
  - Message: `feat(search): add useCache toggle param to search API`
  - Files: `src/app/api/search/route.ts`
  - Pre-commit: `npm run build`

- [ ] 3. Fix type normalization in `normalizeShop()`

  **What to do**:
  - In `src/hooks/use-bike-search.ts`, BEFORE line 67, add a `normalizeType()` function:
    ```typescript
    const normalizeType = (raw: unknown): Bike['type'] => {
      const t = String(raw || '').toLowerCase().trim();
      const typeMap: Record<string, Bike['type']> = {
        scooter: 'scooter',
        automatic: 'scooter',
        auto: 'scooter',
        moped: 'scooter',
        'step-through': 'scooter',
        'semi-auto': 'semi-auto',
        'semi-automatic': 'semi-auto',
        'semi automatic': 'semi-auto',
        underbone: 'semi-auto',
        manual: 'manual',
        standard: 'manual',
        sport: 'manual',
        naked: 'manual',
        adventure: 'adventure',
        enduro: 'adventure',
        'dual-sport': 'adventure',
        'off-road': 'adventure',
        touring: 'adventure',
        trail: 'adventure',
      };
      return typeMap[t] ?? 'scooter';
    };
    ```
  - Replace line 67:
    FROM: `type: (b.type as Bike['type']) || 'scooter',`
    TO:   `type: normalizeType(b.type),`
  - DO NOT move `normalizeShop()` to a separate file
  - DO NOT touch `convertPrice()` or any other field normalization
  - DO NOT change the `Bike` interface type union (keep the 4 types)

  **Must NOT do**:
  - Do NOT refactor normalizeShop into a separate module
  - Do NOT add new bike types beyond the existing 4
  - Do NOT modify convertPrice logic (known bug, out of scope)

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: `[]`
    - Small self-contained function addition

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 2)
  - **Blocks**: Tasks 4, 5
  - **Blocked By**: None

  **References**:
  - `src/hooks/use-bike-search.ts:5-15` — `Bike` interface with `type` field definition
  - `src/hooks/use-bike-search.ts:36-85` — `normalizeShop()` function
  - `src/hooks/use-bike-search.ts:67` — Current broken type cast: `(b.type as Bike['type']) || 'scooter'`
  - `src/components/bike-card.tsx:10-15` — `typeColors` map showing the 4 valid types
  - rentabikevn.com navigation — Real site uses "Manual", "Semi-Auto", "Automatic" (not "Scooter")

  **Acceptance Criteria**:
  - [ ] `normalizeType()` function exists in `use-bike-search.ts`
  - [ ] Line 67 uses `normalizeType(b.type)` instead of cast
  - [ ] `normalizeType('automatic')` returns `'scooter'`
  - [ ] `normalizeType('SEMI-AUTOMATIC')` returns `'semi-auto'`
  - [ ] `normalizeType(undefined)` returns `'scooter'`
  - [ ] `normalizeType('enduro')` returns `'adventure'`
  - [ ] `npm run build` exits 0

  **QA Scenarios:**
  ```
  Scenario: Type normalization handles all common variants
    Tool: Bash (grep + build)
    Steps:
      1. Read use-bike-search.ts and verify normalizeType function exists
      2. Verify typeMap contains: automatic→scooter, semi-automatic→semi-auto, enduro→adventure
      3. Verify line 67 calls normalizeType(b.type)
      4. Run: npm run build
    Expected Result: Function present with correct mappings; build passes
    Evidence: .sisyphus/evidence/task-3-type-normalization.txt
  ```

  **Commit**: YES
  - Message: `fix(normalize): add type normalization mapping for Mino variants`
  - Files: `src/hooks/use-bike-search.ts`
  - Pre-commit: `npm run build`

---

- [ ] 4. Frontend: Cache toggle Switch + multi-select bike type filter UI + hook wiring

  **What to do**:
  This is the main UI task. Three sub-features in one task because they're all in `page.tsx`:

  **A) Cache toggle Switch:**
  - Import `Switch` from `@/components/ui/switch`
  - Add state: `const [useCache, setUseCache] = useState(false)`
  - Render a Switch with label below the city selector row:
    ```tsx
    <div className="flex items-center gap-3">
      <Switch
        id="cache-toggle"
        checked={useCache}
        onCheckedChange={setUseCache}
        disabled={state.isSearching}
      />
      <label htmlFor="cache-toggle" className="text-sm text-zinc-600">
        {useCache ? '⚡ Cached results (faster)' : '🔴 Live scraping (shows TinyFish in action)'}
      </label>
    </div>
    ```
  - Disable the Switch during active search (`state.isSearching`)

  **B) Multi-select bike type filter:**
  - Add state: `const [selectedTypes, setSelectedTypes] = useState<Set<Bike['type']>>(new Set())`
  - Define type config array:
    ```typescript
    const BIKE_TYPES = [
      { name: 'scooter' as const, label: '🛵 Scooter', color: 'bg-blue-500 hover:bg-blue-600 text-white' },
      { name: 'semi-auto' as const, label: '⚙️ Semi-Auto', color: 'bg-green-500 hover:bg-green-600 text-white' },
      { name: 'manual' as const, label: '🏍️ Manual', color: 'bg-orange-500 hover:bg-orange-600 text-white' },
      { name: 'adventure' as const, label: '🏔️ Adventure', color: 'bg-purple-500 hover:bg-purple-600 text-white' },
    ];
    ```
  - Render pill buttons below the cache toggle:
    ```tsx
    <div className="flex flex-wrap gap-2">
      {BIKE_TYPES.map((type) => (
        <Button
          key={type.name}
          variant={selectedTypes.has(type.name) ? 'default' : 'outline'}
          size="sm"
          className={selectedTypes.has(type.name) ? type.color : ''}
          onClick={() => {
            setSelectedTypes(prev => {
              const next = new Set(prev);
              next.has(type.name) ? next.delete(type.name) : next.add(type.name);
              return next;
            });
          }}
        >
          {type.label}
        </Button>
      ))}
    </div>
    ```
  - Multiple types can be selected simultaneously (union filter)
  - No types selected = show all bikes

  **C) Hook wiring + filter logic:**
  - Update `useBikeSearch` hook in `use-bike-search.ts`:
    - Change `search` signature: `(city: string, useCache?: boolean) => void`
    - In the fetch body, change `JSON.stringify({ city })` to `JSON.stringify({ city, useCache: useCache ?? false })`
  - Update `handleCitySelect` in `page.tsx`:
    - Pass useCache: `search(city.name, useCache)`
    - Reset filter: `setSelectedTypes(new Set())`
  - Add filter logic before rendering `ResultsGrid`:
    ```typescript
    const filteredShops = useMemo(() => {
      if (selectedTypes.size === 0) return state.shops;
      return state.shops
        .map(shop => ({
          ...shop,
          bikes: shop.bikes.filter(bike => selectedTypes.has(bike.type)),
        }))
        .filter(shop => shop.bikes.length > 0);
    }, [state.shops, selectedTypes]);
    ```
  - Pass `filteredShops` instead of `state.shops` to `ResultsGrid`
  - When `filteredShops` is empty but `state.shops` is not (filters active, zero matches):
    Show a message: "No bikes match your filter. Try selecting more types."

  **Must NOT do**:
  - Do NOT create new routes or pages
  - Do NOT add sorting, price range, or duration filtering
  - Do NOT add state management libraries — plain useState is fine
  - Do NOT change the ResultsGrid, ShopGroup, or BikeCard components (those are other tasks)
  - Do NOT implement STREAMING_URL live preview

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
  - **Skills**: [`frontend-ui-ux`]
    - `frontend-ui-ux`: UI layout, component composition, responsive design

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 2 (sequential after Wave 1)
  - **Blocks**: Task 5
  - **Blocked By**: Tasks 1, 2, 3

  **References**:
  - `src/app/page.tsx` — Main page, all UI changes go here. Current city selector pattern at lines 38-50.
  - `src/hooks/use-bike-search.ts:87-91` — Current hook return type and `search(city: string)` signature
  - `src/hooks/use-bike-search.ts:115-219` — The `search` useCallback that needs `useCache` param added
  - `src/hooks/use-bike-search.ts:138` — The `JSON.stringify({ city })` that needs `useCache` added
  - `src/components/ui/switch.tsx` — The Switch component installed in Task 1
  - `src/components/bike-card.tsx:10-15` — `typeColors` map to match pill button colors
  - `src/components/results-grid.tsx` — Results container that receives `shops` prop

  **Acceptance Criteria**:
  - [ ] Switch component renders below city selector
  - [ ] Switch defaults to unchecked (live mode)
  - [ ] Switch is disabled when `state.isSearching` is true
  - [ ] 4 pill buttons render with correct emoji labels
  - [ ] Clicking a pill toggles its selection state (multi-select)
  - [ ] Selected pills show matching type color; unselected show outline
  - [ ] With filters active, only matching bike types appear in results
  - [ ] With no filters selected, all bikes appear
  - [ ] Changing city resets selected types to empty
  - [ ] "No bikes match" message appears when filters produce zero results
  - [ ] Search hook sends `useCache` in POST body
  - [ ] `npm run build` exits 0

  **QA Scenarios:**
  ```
  Scenario: Cache toggle renders and defaults to OFF
    Tool: Playwright
    Steps:
      1. Navigate to http://localhost:3000
      2. Find switch with id="cache-toggle"
      3. Assert: switch is unchecked (aria-checked="false")
      4. Assert: label text contains "Live scraping"
    Expected Result: Switch unchecked, label says live mode
    Evidence: .sisyphus/evidence/task-4-cache-toggle.png

  Scenario: Multi-select bike type filter toggles independently
    Tool: Playwright
    Steps:
      1. Navigate to http://localhost:3000
      2. Find buttons containing text "Scooter", "Semi-Auto", "Manual", "Adventure"
      3. Click "Scooter" button → assert it gains colored background
      4. Click "Manual" button → assert both "Scooter" and "Manual" are now selected
      5. Click "Scooter" again → assert only "Manual" is selected
    Expected Result: Multi-select works, toggling is independent
    Evidence: .sisyphus/evidence/task-4-multiselect-filter.png

  Scenario: Filter state resets on city change
    Tool: Playwright
    Steps:
      1. Click "Scooter" and "Manual" pills to select them
      2. Click a city button (e.g., "HCMC")
      3. Assert: no pills are selected (all show outline variant)
    Expected Result: City change clears filter selection
    Evidence: .sisyphus/evidence/task-4-reset-on-city-change.png
  ```

  **Commit**: YES
  - Message: `feat(ui): add cache toggle and multi-select bike type filter`
  - Files: `src/app/page.tsx`, `src/hooks/use-bike-search.ts`
  - Pre-commit: `npm run build`

- [ ] 5. Clickable bike cards with per-bike URL extraction

  **What to do**:
  Three files need changes:

  **A) Update Bike interface + normalization (`use-bike-search.ts`):**
  - Add `url: string | null` to the `Bike` interface (after `available: boolean`)
  - In `normalizeShop()`, add URL extraction after the `available` field:
    ```typescript
    const rawUrl = b.url ? String(b.url).trim() : '';
    // ...
    url: rawUrl || null,
    ```

  **B) Update Mino GOAL_PROMPT (`route.ts`):**
  - In the `GOAL_PROMPT` string, add to the extraction list (after the `available` instruction):
    ```
    - URL to this bike's individual detail page (the href on the bike listing).
      If all bikes are on the same page with no individual links, set to null.
    ```
  - In the JSON example, add: `"url": "https://example.com/bikes/honda-wave-110"`
  - DO NOT rewrite the entire prompt — only add the url field

  **C) Make bike cards clickable (`bike-card.tsx` + `shop-group.tsx`):**
  - In `shop-group.tsx`: update where `<BikeCard>` is rendered to pass `shopWebsite={shop.website}`
  - In `bike-card.tsx`:
    1. Update `BikeCardProps` to add `shopWebsite: string`
    2. Import `ExternalLink` from `lucide-react`
    3. Compute link: `const href = bike.url || shopWebsite || null;`
    4. If `href` is truthy, wrap the Card in an `<a>` tag:
       ```tsx
       const Wrapper = href
         ? ({ children }: { children: React.ReactNode }) => (
             <a href={href} target="_blank" rel="noopener noreferrer" className="block group">
               {children}
             </a>
           )
         : ({ children }: { children: React.ReactNode }) => <div>{children}</div>;

       return (
         <Wrapper>
           <Card className="... cursor-pointer group-hover:shadow-lg group-hover:ring-2 group-hover:ring-zinc-200 ...">
             {/* existing internals */}
             {href && (
               <ExternalLink className="w-3.5 h-3.5 text-zinc-400 group-hover:text-zinc-600" />
             )}
           </Card>
         </Wrapper>
       );
       ```
    5. Add `cursor-pointer` class when clickable, keep default cursor when not
    6. Add a small `ExternalLink` icon in the card header area next to the bike name

  **Must NOT do**:
  - Do NOT make unavailable bikes look clickable (dim the link icon or reduce hover effect)
  - Do NOT add loading states when clicking cards (they're external links)
  - Do NOT rewrite the entire GOAL_PROMPT — only add the URL field
  - Do NOT add `<a>` tags that nest inside other `<a>` tags

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
  - **Skills**: [`frontend-ui-ux`]
    - `frontend-ui-ux`: Component refactoring, interactive elements, hover states

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 3 (sequential after Wave 2)
  - **Blocks**: F1, F2, F3
  - **Blocked By**: Tasks 3, 4

  **References**:
  - `src/components/bike-card.tsx` — Full file (91 lines), the Card to make clickable
  - `src/components/shop-group.tsx` — Where BikeCard is rendered, needs to pass shopWebsite
  - `src/hooks/use-bike-search.ts:5-15` — Bike interface to add `url` field
  - `src/hooks/use-bike-search.ts:56-76` — normalizeShop bike mapping, where to add url extraction
  - `src/app/api/search/route.ts:41-77` — GOAL_PROMPT to add URL extraction instruction
  - `src/app/api/search/route.ts:59-76` — JSON example in prompt to add url field
  - `lucide-react` — Already in dependencies, used in shop-group.tsx (`Zap` icon)

  **Acceptance Criteria**:
  - [ ] `Bike` interface includes `url: string | null`
  - [ ] `normalizeShop()` extracts and normalizes URL (empty string → null)
  - [ ] GOAL_PROMPT includes URL extraction instruction and example
  - [ ] `BikeCardProps` includes `shopWebsite: string`
  - [ ] `shop-group.tsx` passes `shopWebsite` to BikeCard
  - [ ] Bike cards with URLs render as `<a>` tags with `target="_blank"`
  - [ ] Bike cards without URLs fall back to `shopWebsite`
  - [ ] ExternalLink icon visible on clickable cards
  - [ ] `npm run build` exits 0

  **QA Scenarios:**
  ```
  Scenario: Bike cards render as links
    Tool: Playwright
    Steps:
      1. Navigate to http://localhost:3000
      2. Click HCMC city button and wait for results
      3. Find first bike card element
      4. Assert: card is wrapped in <a> tag
      5. Assert: <a> has target="_blank" and rel="noopener noreferrer"
      6. Assert: ExternalLink icon (svg) is visible in card header
    Expected Result: Cards are clickable links opening in new tab
    Evidence: .sisyphus/evidence/task-5-clickable-cards.png

  Scenario: Cards without bike URL fall back to shop website
    Tool: Bash (grep)
    Steps:
      1. Read bike-card.tsx
      2. Verify fallback: const href = bike.url || shopWebsite || null
      3. Verify non-clickable case: if !href, render as <div> not <a>
    Expected Result: Fallback chain is correct
    Evidence: .sisyphus/evidence/task-5-fallback-logic.txt
  ```

  **Commit**: YES
  - Message: `feat(cards): make bike cards clickable with per-bike URLs`
  - Files: `src/components/bike-card.tsx`, `src/components/shop-group.tsx`, `src/hooks/use-bike-search.ts`, `src/app/api/search/route.ts`
  - Pre-commit: `npm run build`

---

## Final Verification Wave (MANDATORY — after ALL implementation tasks)

> 3 review agents run in PARALLEL. ALL must APPROVE. Rejection → fix → re-run.

- [ ] F1. **Build + Lint Verification** — `quick`
  Run `npm run build` and `npm run lint`. Verify zero errors. Check TypeScript compilation succeeds with all new types (Bike.url, SearchBody.useCache, BikeCardProps.shopWebsite).
  Output: `Build [PASS/FAIL] | Lint [PASS/FAIL] | VERDICT: APPROVE/REJECT`

- [ ] F2. **Plan Compliance Audit** — `oracle`
  Read the plan end-to-end. For each "Must Have": verify implementation exists (read file, check code). For each "Must NOT Have": search codebase for forbidden patterns — reject with file:line if found. Check evidence files exist.
  Output: `Must Have [N/N] | Must NOT Have [N/N] | VERDICT: APPROVE/REJECT`

- [ ] F3. **UI QA with Playwright** — `unspecified-high` + `playwright` skill
  Start dev server. Use Playwright to: (a) verify Switch toggle renders and is clickable, (b) verify 4 bike type pill buttons render and are independently toggleable, (c) verify bike cards have `<a>` tags with `target="_blank"`, (d) screenshot the filter area. Save screenshots to `.sisyphus/evidence/final-qa/`.
  Output: `Scenarios [N/N pass] | Screenshots [N captured] | VERDICT: APPROVE/REJECT`

---

## Commit Strategy

| Wave | Commit Message | Files |
|------|---------------|-------|
| 1 | `feat(search): add useCache toggle param to search API` | `route.ts` |
| 1 | `fix(normalize): add type normalization mapping for Mino variants` | `use-bike-search.ts` |
| 1 | `chore(ui): install shadcn Switch component` | `src/components/ui/switch.tsx` |
| 2 | `feat(ui): add cache toggle and multi-select bike type filter` | `page.tsx`, `use-bike-search.ts` |
| 3 | `feat(cards): make bike cards clickable with per-bike URLs` | `bike-card.tsx`, `shop-group.tsx`, `use-bike-search.ts`, `route.ts` |

---

## Success Criteria

### Verification Commands
```bash
npm run build   # Expected: exits 0, no TypeScript errors
npm run lint     # Expected: exits 0 or pre-existing warnings only
```

### Final Checklist
- [ ] Cache toggle Switch renders, defaults OFF, disabled during search
- [ ] When cache OFF: zero `[CACHE]` messages in server logs
- [ ] When cache ON: cached results stream instantly with ⚡ badge
- [ ] 4 bike type pill buttons render with correct colors
- [ ] Multiple types selectable simultaneously (union filter)
- [ ] No types selected = all bikes shown
- [ ] Filter resets when city changes
- [ ] Empty filter results show "No bikes match your filter" message
- [ ] Bike cards are clickable `<a>` tags with `target="_blank"`
- [ ] Cards link to bike.url when available, fall back to shop.website
- [ ] Type normalization maps "automatic" → "scooter", "semi-automatic" → "semi-auto"
- [ ] `npm run build` passes with zero errors
