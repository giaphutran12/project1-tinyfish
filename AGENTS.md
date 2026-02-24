# AGENTS.md — TinyFish Product Growth Intern Take-Home

**Generated:** 2026-02-24
**Mission:** Idea approved → working demo → PR merged → hired.

**Universal rules apply** — see `C:\Users\Administrator\.claude\AGENTS.md` for orchestrator behavior, background tasks, research priorities, and end-of-session protocol.

## THE MISSION

You are helping a developer land a Product Growth Intern role at TinyFish ($47M Series A, SF-based AI infra). The interviewer is **Sky Zhang**. The assignment:

1. **Phase 1 (24h)**: Submit a use case idea with full documentation → get approved by Sky
2. **Phase 2 (72h after approval)**: Build a working MVP demo, submit PR to GitHub, post on socials

**Everything you do must serve this timeline.** No yak-shaking. No over-engineering. Ship or die.

## WHAT TINYFISH / MINO IS

TinyFish built **Mino** — a web agent API. You give it:
- A URL
- A natural language goal

It returns structured JSON with exactly what you asked for.

**Mino's superpowers:**
- Parallel browser automation (10-50+ sites simultaneously)
- AI-powered navigation through complex sites (multi-step forms, nested menus, dynamic loading)
- Works on niche sites that have NO API

**Mino API docs are indexed in Nia** (source: `123d2659-98bb-4843-8a26-f6c6f4c09a30`).

## NIA INDEXED SOURCES (ALREADY AVAILABLE)

| Source | Nia ID | What's In It |
|--------|--------|-------------|
| TinyFish API Docs | `123d2659-98bb-4843-8a26-f6c6f4c09a30` | Full Mino API reference, auth, examples, prompting guide |
| Job Posting | `8b21a4c4-f34c-49e0-8afb-66d69d5d8086` | Role description, expectations, red flags |
| Product Growth SOP | `fb92db29-bc94-4ad0-b3d1-ab8c37177610` | Onboarding, strategy rules, deliverable format, payment |
| Use Case Library | `702d4811-c513-4ec9-9803-9c971500b886` | 60+ existing use cases — what's been built already |
| Sky's Example (Pediatric Urgent Care) | `cf5dbb89-62fb-40d7-91b7-5635a7001738` | The gold standard — how Sky documented his own use case |

**To search across all**: `search.sh universal "your query"`
**To read a specific doc page**: `sources.sh read <nia_id> "index.md"`
**To search Mino API specifics**: `sources.sh grep "123d2659-98bb-4843-8a26-f6c6f4c09a30" "pattern"`

## WHAT "QUALIFIED" MEANS (The Quality Bar)

Your submission MUST include ALL of these or it gets rejected:

| Deliverable | Details |
|---|---|
| **Use case name** | 3-5 words (e.g. "Bulk Freelance Gig Applier") |
| **The "Why"** | One-sentence problem statement |
| **Technical Assets** | Code repository link |
| **Live Demo or Video** | Working URL or screen recording |
| **App Screenshot** | For their website showcase |
| **Duplicable PRD** | Architecture overview, code snippet, exact Mino prompt, sample JSON output |

PRD must be generated using this prompt to your vibe-coding tool:
> "Please generate a standard developer documentation example for the Mino API use case I've developed." (Full prompt in SOP Nia source)

## STRATEGY: WHAT TO BUILD (AND WHAT NOT TO)

### The Rules

- **Uniqueness Rule**: Check the Use Case Library FIRST. If it exists, pick a different industry/problem.
- **Scale Rule**: Prioritize tasks requiring parallel processing (checking many niche sites simultaneously). This is where Mino wins.
- **Complexity Rule**: Target complex single-query tasks competitors (Atlas, Comet) struggle with — 5-step forms, nested menus, dynamic "Load More" logic.
- **Iteration Exception**: You may revisit an existing use case ONLY if significantly improving reliability, speed, or adding an action step (from "finding" to "auto-filling/booking").

### DO NOT BUILD

- Anything on major platforms with anti-bot/2FA: X, Reddit, LinkedIn, Amazon, Instagram
- Anything where a clean public API already exists (Stripe, Google Maps)
- Duplicates of these heavily-covered categories: retail (8+), medical (4), finance (3), legal (5+), dev tools (7+), personal/lifestyle (8+), enterprise (5+), education (4)

### HIGH-VALUE GAPS (Undercovered or Uncovered)

- Vietnam-specific use cases (you're IN Vietnam — local knowledge = unfair advantage)
- Agriculture / farming
- Immigration / visa
- Travel (beyond hotels)
- Automotive (beyond used cars)
- HR / recruiting (beyond job boards)
- Events / ticketing
- Manufacturing
- Telecom
- Insurance (beyond Singapore medical)
- Sports (beyond betting odds)
- Non-profit / NGO

### BOUNTY USE CASES (Pre-approved, bonus pay)

Check the SOP for "Homer's Use Cases" bounty list:
- **$200**: Competitive Pricing Intelligence, Aggregate Product Feedback, New Customer Discovery
- **$150**: Monitor New Product Releases, Track Competitor Features, Quantitative Benchmarks
- **$100**: Social Media Sentiment Tracking, Organic Mentions Monitoring

## THE BUILD WORKFLOW

### Phase 1: Ideation + Documentation (Target: 24h)

```
1. Brainstorm 3-5 ideas → cross-reference Use Case Library for uniqueness
2. Pick the one that best combines: Scale (parallel sites) + Real utility + Novelty
3. Write the full product brief per "Qualified" checklist above
4. Submit to Sky for approval
```

### Phase 2: Build (72h after approval)

```
1. Scaffold the app (Next.js recommended — strong frontend dev)
2. Integrate Mino API (see docs: auth, sync/async endpoints, SSE streaming)
3. Build frontend that showcases the Mino automation visually
4. Get a live demo URL (Vercel, Lovable, Railway — anything works)
5. Record a demo video
6. Submit PR to github.com/tinyfish-io/tinyfish-cookbook (folder: vietnam-bike-price-scout/)
7. Write social post using their tool: https://tinyfish-social.vercel.app/
```

### Tech Stack (Recommended)

- **Frontend**: Next.js + Tailwind (you're strong here)
- **Mino API**: REST — sync endpoint for simple, SSE streaming for live UX
- **Hosting**: Vercel (free, instant deploys, most existing demos use it)
- **Build tool**: Claude Code (this tool — your primary)

## MINO API QUICK REFERENCE

**Base URL**: `https://agent.tinyfish.ai/v1/automation/run-sse` (check docs for exact endpoints)

**Three endpoint modes:**
1. **Sync** (`/run/sync`) — Wait for result. Simple. Good for <30s tasks.
2. **Async** (`/run/async`) — Start task, poll for result. Good for long tasks.
3. **SSE Streaming** (`/run/stream`) — Real-time updates. Best UX for demos.

**Core request shape:**
```json
{
  "url": "https://example.com",
  "goal": "Find all available appointment slots for next week",
  "schema": { ... }  // Optional: structured output schema
}
```

**For exact API details**: `sources.sh read "123d2659-98bb-4843-8a26-f6c6f4c09a30" "quick-start.md"`

## PROJECT-SPECIFIC GUARDRAILS

- **Never read `.env` files** — reference `TINYFISH_API_KEY` by name only
- **MUST use Edge Runtime** on API routes: `export const runtime = 'edge'`
- **MUST use `getReader()` + buffer pattern for SSE** — NEVER `await response.text()`
- **No Supabase, no database, no auth, no persistence** — stateless MVP only
- **PR target**: `tinyfish-io/tinyfish-cookbook`, folder: `vietnam-bike-price-scout/`
- **Env var name**: `TINYFISH_API_KEY`

## ANTI-PATTERNS (WILL GET YOU REJECTED)

- Submitting without ALL deliverables (name, why, repo, demo, screenshot, PRD)
- Building something already in the Use Case Library without significant improvement
- Targeting anti-bot platforms (X, Reddit, LinkedIn, Amazon)
- Over-engineering — MVP is explicitly fine, they said "doesn't need to be production-ready"
- 4-page formal resume energy — they want builders, not corporate
- No evidence of shipping — show half-broken demos over polished pitch decks

## WHAT IMPRESSES THEM

- Speed of delivery (72h window is the bar — faster = better)
- Creative "wait, you can DO that?" factor
- Showing Mino's parallel processing advantage (many sites at once)
- Clean demo video showing the automation in action
- Social post that genuinely helps people discover what's possible
- Honest bug reports when Mino fails — they value feedback for their eng team

## KEY CONTACTS

- **Interviewer**: Sky Zhang (built the pediatric urgent care example)
- **Application email**: keith@tinyfish.ai
- **GitHub repo for PRs**: github.com/tinyfish-io/tinyfish-cookbook
- **Post generator**: https://tinyfish-social.vercel.app/
- **Mino playground**: https://mino.ai/

## NOTES

- Mino API credits are unlimited for interns — don't worry about cost
- They'll provide Claude Code / Lovable / Cursor access after you pass the 48h challenge
- Payment: 50% after PR merged, 50% after post is live
- Performance bonus: 5k views = $20, up to 1M views = $1,000
- The product is early-stage — document bugs as you find them, they're valuable
- Best practices doc by top contributor Pranav: https://mino-intern-docs.vercel.app/best-practices
