# Vietnam Bike Price Scout — Talk Outline

**Event:** AI Tinkerers Ho Chi Minh City
**Theme:** "From Prompt to Agent"
**Duration:** 5 minutes
**Format:** In-person, projector, live demo + code walkthrough

---

## BEFORE YOU START

- Open **Tab 1**: `viet-bike-scout.vercel.app` — start a live search (HCMC, all bike types, cache OFF)
- Open **Tab 2**: Same app — cache ON, same city, already loaded with instant results
- Open **Tab 3**: VS Code with `src/app/api/search/route.ts` ready
- Start the Tab 1 search **1-2 minutes before your slot**

---

## [0:00 – 0:45] THE HOOK — Show the Problem

> **SHOW:** Tab 1 — the app with live browser agent iframes running

- "If you've ever rented a motorbike in Vietnam, you know the pain"
- "There's no Kayak for bikes. No aggregator. You visit 5-10 different shops, each with different layouts, different currencies, some in Vietnamese"
- "I built an app that sends AI browser agents to all of them at once"

> **SHOW:** Point at the iframe grid — live TinyFish agents navigating real websites

- "What you're seeing right now are real browser sessions — each one is an AI agent navigating a different rental shop, extracting pricing, handling popups, converting VND to USD"
- "This is not a mockup. These are live agents running in parallel right now"

---

## [0:45 – 1:45] THE API — "It's just a fetch call"

> **SHOW:** Tab 3 — VS Code, `route.ts` lines 185-197

```typescript
const response = await fetch(MINO_SSE_URL, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Accept: "text/event-stream",
    "X-API-Key": apiKey,
  },
  body: JSON.stringify({
    url,        // the shop's website
    goal: GOAL_PROMPT,  // natural language instruction
  }),
  signal: controller.signal,
});
```

- "This is the entire TinyFish integration. 11 lines. A fetch call."
- "You give it a URL and a goal in plain English. It gives you back structured JSON."
- "No SDK, no complex setup. Just `url` and `goal`."

> **SHOW:** Scroll to the GOAL_PROMPT (lines 41-80)

- "The goal is just natural language — navigate to pricing page, handle popups, extract all bikes with prices, handle pagination"
- "TinyFish figures out the rest — cookie banners, dynamic loading, VND to USD conversion"

---

## [1:45 – 2:45] THE PARALLELISM — "Promise.allSettled, that's it"

> **SHOW:** `route.ts` lines 341-361

```typescript
const tasks = uncachedSites.map((url) =>
  (async () => {
    return runTinyFishSseForSite(url, apiKey, siteEnqueue);
  })(),
);

const settled = await Promise.allSettled(tasks);
```

- "Parallelism is just `Promise.allSettled`. Fire all sites at once. No staggering, no rate limiting."
- "TinyFish has no concurrency caps — it's designed for this. I send 5-6 requests simultaneously, each one launches its own browser agent"

> **SHOW:** The CITY_SITES config (lines 12-39) — briefly

- "Each city has 5-6 target shops. All niche local sites with no public API."
- "18 shops across 4 cities. All scraped in parallel."

---

## [2:45 – 3:45] THE STREAMING — "Results arrive as they finish"

> **SHOW:** `route.ts` lines 207-240 — the SSE reader

```typescript
const reader = response.body.getReader();
const decoder = new TextDecoder();
let buffer = "";

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  buffer += decoder.decode(value, { stream: true });
  const lines = buffer.split("\n");
  buffer = lines.pop() ?? "";

  for (const line of lines) {
    if (!line.startsWith("data: ")) continue;
    const event = JSON.parse(line.slice(6));

    if (event.streamingUrl) {
      // → forward the live iframe URL to the client
    }
    if (event.status === "COMPLETED") {
      // → parse the JSON, stream to client, cache the result
    }
  }
}
```

- "TinyFish streams back Server-Sent Events. Two event types matter:"
  - "`streamingUrl` — that's the live browser iframe you saw earlier"
  - "`COMPLETED` — that's the structured JSON with all the bike data"
- "The `getReader()` + buffer pattern — SSE events can arrive split across network chunks. The buffer holds incomplete lines until the next chunk completes them"
- "Results stream to the frontend as each shop finishes — you don't wait for the slowest one"

---

## [3:45 – 4:15] THE OUTPUT — "Structured data from unstructured websites"

> **SHOW:** Tab 1 or Tab 2 — the results grid with bike cards

- "Every card you see came from a completely different website with a completely different layout"
- "TinyFish returns JSON. But AI output is messy — so we normalize it"

> **SHOW:** `use-bike-search.ts` lines 44-68 — type normalization

```typescript
const typeMap: Record<string, Bike['type']> = {
  scooter: 'scooter',
  automatic: 'scooter',    // TinyFish might say "automatic"
  'semi-automatic': 'semi-auto', // or "semi-automatic"
  enduro: 'adventure',     // or "enduro"
  // ... 20+ mappings → 4 canonical types
};
```

- "TinyFish might return 'automatic', 'semi-automatic', 'enduro' — we map all of them to 4 clean types"
- "Same for prices — if a price is over 1,000, it's probably VND, so we divide by 25,000"

---

## [4:15 – 4:40] ERROR HANDLING — "What if an agent fails?"

> **SHOW:** `route.ts` lines 254-259 — the catch block

```typescript
} catch (error) {
  console.error(`[MINO] Failed: ${url}`, error);
  return false;  // this site failed, but others continue
} finally {
  clearTimeout(timeoutId);
}
```

- "Honestly? I haven't seen an agent fully fail — worst case it returns empty results"
- "But the code handles it: each site runs independently. If one fails, the other 4 still complete"
- "`Promise.allSettled` — not `Promise.all` — so one failure doesn't kill the whole search"
- "Supabase cache is also optional — if it's down, the app just skips caching and scrapes live"

---

## [4:40 – 5:00] WRAP UP

> **SHOW:** Tab 1 — results should be fully loaded by now. Scroll through them.

- "4 cities, 18 shops, all scraped in parallel, results streaming in real-time"
- "The entire backend is one API route — 389 lines, and half of that is the cache layer"
- "Built this in about 2 weeks using OpenCode — an open-source AI coding agent — with Nia as the context layer for indexing the TinyFish docs"
- "The takeaway: going from prompt to agent is simpler than you think. The hard part isn't the infrastructure — it's picking the right problem"

---

## BACKUP PLAN

If the live search is still running when you reach the results section:
→ Switch to **Tab 2** (cached results) and say: "Let me show you the cached version — same data, pulled from our 6-hour cache"

If someone asks about rate limits:
→ "TinyFish has no rate limits or concurrency caps. That's by design — it's built for parallel automation at scale."

If someone asks about cost:
→ "TinyFish API credits are unlimited for this demo. In production, you'd want to cache aggressively — which we do. 6-hour TTL in Supabase."

If someone asks about accuracy:
→ "It's surprisingly good. The natural language prompt handles most edge cases. The normalization layer on our side cleans up the rest — type mapping, VND conversion, filtering out bikes with no name."

---

## KEY NUMBERS TO REMEMBER

| Metric | Value |
|--------|-------|
| TinyFish API integration | 11 lines of code |
| Parallel sites per city | 5-6 simultaneous agents |
| Total shops covered | 18 across 4 cities |
| Typical search time | 15-30 seconds |
| Cached result time | < 1 second |
| Backend file count | 1 API route (389 lines) |
| Cache TTL | 6 hours |
| Max parallel iframes | 5 per search |
