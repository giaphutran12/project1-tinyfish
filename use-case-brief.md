# Vietnam Motorbike Rental Aggregator

## Use Case Name
**Vietnam Bike Price Scout**

## The "Why"
Tourists and expats in Vietnam waste hours checking 10-20 individual rental shop websites to compare motorbike prices — there's no aggregator and no API for any of them.

## Description
Users select a city (HCMC, Hanoi, Da Nang, Hoi An, Nha Trang) and bike type (scooter, semi-auto, manual, adventure) → Mino scrapes 15-20+ rental shop websites **in parallel** → returns a unified price comparison dashboard showing daily/weekly/monthly rates, bike models, deposit requirements, and booking links — all in one view.

## Why Mino Is the Only Solution
- **200+ independent rental shops** across Vietnam, each with their own website (WordPress, Wix, custom builds)
- **Zero API exists** — no shop exposes pricing data programmatically
- **Zero aggregator exists** — no Vietnamese "Kayak for motorbikes"
- Sites require **multi-step navigation**: bike model pages → pricing tables → availability calendars → booking forms
- Mino's **parallel processing** is the killer feature: checking 20 shops simultaneously vs. opening 20 browser tabs manually

## Target Persona
- **Primary**: Tourists and backpackers planning Vietnam trips (millions annually)
- **Secondary**: Expats and digital nomads needing monthly rentals
- **Tertiary**: Vietnamese locals comparing prices for weekend trips

## Target Websites (Verified, Real, With Pricing)

### Ho Chi Minh City
| Site | URL | Pricing Visible | Complexity |
|------|-----|----------------|------------|
| Tigit Motorbikes | https://www.tigitmotorbikes.com/prices | Yes — full price table, currency switcher | Multi-step booking (dates, pickup/drop-off city) |
| The Extra Mile | https://theextramile.co/city-rental-prices/ | Yes — monthly/daily rates by model | Multi-step (city, date, return city) |
| Wheelie Saigon | https://wheelie-saigon.com/scooter-motorcycle-rental-hcmc-daily-weekly-or-monthly/ | Yes — daily/weekly/monthly | Simple listing |
| Saigon Motorcycles | https://saigonmotorcycles.com/rentals/ | Yes — rates by engine size (50cc–750cc) | Simple + form |
| Style Motorbikes | https://stylemotorbikes.com/ | Yes — rental guide with pricing | Simple listing |

### Hanoi
| Site | URL | Pricing Visible | Complexity |
|------|-----|----------------|------------|
| Motorbike Rental Hanoi | https://motorbikerentalinhanoi.com/ | Yes — USD/day per model | Simple listing |
| Offroad Vietnam | https://offroadvietnam.com/ | Yes — scooter + adventure rates | Multi-step (tour vs rental) |
| Rent Bike Hanoi | https://rentbikehanoi.com/ | Yes — daily rates | Simple listing |
| Book2Wheel | https://book2wheel.com/activity/motorcycle-rental-hanoi | Yes — per-bike USD pricing | Multi-step (date picker, login) |
| MotoVina | https://motorvina.com/en/motorbike-rental/ | Yes — city-based, USD/VND toggle | Multi-step (city, dates, model) |

### Da Nang / Hoi An
| Site | URL | Pricing Visible | Complexity |
|------|-----|----------------|------------|
| Motorbike Rental Da Nang | https://motorbikerentaldanang.com/ | Yes — VND + USD per model | Simple + booking |
| Da Nang Bikes Rental | https://danangmotorbikesrental.com/ | Yes — pricing in posts | Simple |
| DaNangBike | https://danangbike.com/ | Yes — pricing guide | Simple |
| Motorbike Rental Hoi An | https://motorbikerentalhoian.com/ | Yes — VND daily per model | Simple listing |
| Hoi An Bike Rental | https://hoianbikerental.com/pricing/ | Yes — dedicated pricing page, multi-currency | Simple |
| Tuan Motorbike | https://tuanmotorbike.com/ | Yes — one-way pricing (Da Nang ↔ Hoi An) | Simple |

### Nha Trang / Mui Ne
| Site | URL | Pricing Visible | Complexity |
|------|-----|----------------|------------|
| Moto4Free | https://moto4free.com/ | Yes — fleet with booking | Simple + booking |
| Motorbike Mui Ne | https://motorbikemuine.com/ | Yes — VND + USD per model | Simple listing |

**Total: 18 verified sites across 5 cities, all with visible pricing, all English-language.**

---

## Architecture Overview

```
┌─────────────────────────────────────────────────┐
│  User Interface (Next.js + Tailwind)            │
│  - City selector (HCMC, Hanoi, Da Nang, etc.)   │
│  - Bike type filter (scooter, semi-auto, etc.)  │
│  - Duration toggle (daily/weekly/monthly)        │
└────────────────────┬────────────────────────────┘
                     │ User clicks "Search"
                     ▼
┌─────────────────────────────────────────────────┐
│  Next.js API Route (/api/search)                │
│  - Builds Mino requests for selected city       │
│  - Fires 10-18 parallel Mino calls              │
│  - Uses Promise.allSettled() for fault tolerance │
└────────────────────┬────────────────────────────┘
                     │ 10-18 parallel requests
                     ▼
┌─────────────────────────────────────────────────┐
│  TinyFish Mino API (SSE Streaming)              │
│                                                  │
│  Agent 1  → tigitmotorbikes.com        ─┐       │
│  Agent 2  → theextramile.co            ─┤       │
│  Agent 3  → wheelie-saigon.com         ─┤       │
│  Agent 4  → saigonmotorcycles.com      ─┤ SSE   │
│  Agent 5  → stylemotorbikes.com        ─┤ streams│
│  ...                                   ─┤       │
│  Agent N  → motorbikemuine.com         ─┘       │
│                                                  │
└────────────────────┬────────────────────────────┘
                     │ Streamed results (real-time)
                     ▼
┌─────────────────────────────────────────────────┐
│  Results Dashboard                               │
│  - Cards populate as each agent completes        │
│  - Sorted by price (cheapest first)              │
│  - Filterable by bike type, duration, deposit    │
│  - "Book Now" links to original rental site      │
└─────────────────────────────────────────────────┘
```

**API calls per search**: 10-18 Mino SSE calls (one per rental site in the selected city), fired in parallel via `Promise.allSettled()`.

**Why SSE streaming**: Results appear in real-time as each agent finishes — the user watches bikes populate the dashboard live. This is the best UX for demos and showcases Mino's parallel power visually.

---

## Mino Goal (Exact Prompt)

```
You are extracting motorbike/scooter rental pricing from this rental shop website.

STEP 1 - NAVIGATE TO PRICING:
Look for links or pages containing: "Pricing", "Rates", "Rental", "Fleet", "Our Bikes", "Price List"
Click to navigate to the pricing/rental page.
If the homepage already shows bikes with prices, stay on this page.

STEP 2 - HANDLE POPUPS:
If a cookie banner, newsletter popup, or chat widget appears, close it by clicking "Accept", "Close", "X", or "Got it".

STEP 3 - EXTRACT BIKE LISTINGS:
For each motorbike or scooter listed on the page, extract:
- Bike name/model (e.g. "Honda Wave 110", "Yamaha NVX 155")
- Engine size if shown (e.g. "110cc", "155cc")  
- Bike type: classify as "scooter", "semi-auto", "manual", or "adventure"
- Daily rental price (if shown)
- Weekly rental price (if shown)
- Monthly rental price (if shown)
- Currency (VND or USD)
- Deposit amount (if mentioned)
- Whether the bike is currently available

STEP 4 - CHECK FOR MORE BIKES:
If there is a "Load More", "View All", "Next Page", or "See More Bikes" button, click it once to load additional listings. Then extract those too.

STEP 5 - RETURN RESULTS:
Return a JSON object with this exact structure:
{
  "shop_name": "Name of the rental shop",
  "city": "City where the shop is located",
  "website": "The URL you are on",
  "bikes": [
    {
      "name": "Honda Wave 110",
      "engine_cc": 110,
      "type": "semi-auto",
      "price_daily_usd": 5,
      "price_weekly_usd": 28,
      "price_monthly_usd": 80,
      "currency": "USD",
      "deposit_usd": 50,
      "available": true
    }
  ],
  "notes": "Any important rental terms (helmet included, delivery available, etc.)"
}

If prices are in VND, convert to approximate USD using 1 USD = 25,000 VND.
If a price tier is not shown (e.g. no weekly rate), set it to null.
Extract up to 20 bikes maximum.
```

---

## Sample JSON Output

```json
{
  "type": "COMPLETE",
  "status": "COMPLETED",
  "resultJson": {
    "shop_name": "Tigit Motorbikes",
    "city": "Ho Chi Minh City",
    "website": "https://www.tigitmotorbikes.com/prices",
    "bikes": [
      {
        "name": "Honda Blade 110",
        "engine_cc": 110,
        "type": "semi-auto",
        "price_daily_usd": 8,
        "price_weekly_usd": 45,
        "price_monthly_usd": 120,
        "currency": "USD",
        "deposit_usd": 100,
        "available": true
      },
      {
        "name": "Yamaha NVX 155",
        "engine_cc": 155,
        "type": "scooter",
        "price_daily_usd": 12,
        "price_weekly_usd": 70,
        "price_monthly_usd": 180,
        "currency": "USD",
        "deposit_usd": 150,
        "available": true
      },
      {
        "name": "Honda XR 150",
        "engine_cc": 150,
        "type": "manual",
        "price_daily_usd": 15,
        "price_weekly_usd": 85,
        "price_monthly_usd": 250,
        "currency": "USD",
        "deposit_usd": 200,
        "available": true
      },
      {
        "name": "Honda CB500X",
        "engine_cc": 500,
        "type": "adventure",
        "price_daily_usd": 35,
        "price_weekly_usd": 200,
        "price_monthly_usd": 550,
        "currency": "USD",
        "deposit_usd": 500,
        "available": false
      }
    ],
    "notes": "Helmet and phone holder included. Free delivery in District 1. One-way rental to Da Nang available (+$30)."
  }
}
```

---

## Code Snippet (TypeScript — Next.js API Route)

```typescript
// /api/search/route.ts
import { NextRequest, NextResponse } from "next/server";

const MINO_API_URL = "https://mino.ai/v1/automation/run-sse";

const CITY_SITES: Record<string, string[]> = {
  "hcmc": [
    "https://www.tigitmotorbikes.com/prices",
    "https://theextramile.co/city-rental-prices/",
    "https://wheelie-saigon.com/scooter-motorcycle-rental-hcmc-daily-weekly-or-monthly/",
    "https://saigonmotorcycles.com/rentals/",
    "https://stylemotorbikes.com/",
  ],
  "hanoi": [
    "https://motorbikerentalinhanoi.com/",
    "https://offroadvietnam.com/",
    "https://rentbikehanoi.com/",
    "https://book2wheel.com/activity/motorcycle-rental-hanoi",
    "https://motorvina.com/en/motorbike-rental/",
  ],
  "danang": [
    "https://motorbikerentaldanang.com/",
    "https://danangmotorbikesrental.com/",
    "https://danangbike.com/",
    "https://motorbikerentalhoian.com/",
    "https://hoianbikerental.com/pricing/",
  ],
  "nhatrang": [
    "https://moto4free.com/",
    "https://motorbikemuine.com/",
  ],
};

const GOAL = `You are extracting motorbike/scooter rental pricing from this rental shop website.
// ... (full prompt from above)
`;

async function scrapeShop(url: string): Promise<any> {
  const response = await fetch(MINO_API_URL, {
    method: "POST",
    headers: {
      "X-API-Key": process.env.MINO_API_KEY!,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ url, goal: GOAL }),
  });

  if (!response.ok) {
    if (response.status === 429) throw new Error("RATE_LIMITED");
    throw new Error(`Mino API error: ${response.status}`);
  }

  // Parse SSE stream
  const text = await response.text();
  const lines = text.split("\n").filter((l) => l.startsWith("data: "));
  const lastEvent = JSON.parse(lines[lines.length - 1].replace("data: ", ""));

  if (lastEvent.streamingUrl) {
    console.log(`[MINO] Live browser: ${lastEvent.streamingUrl}`);
  }

  if (lastEvent.status === "COMPLETED" && lastEvent.resultJson) {
    return lastEvent.resultJson;
  }
  return null;
}

async function scrapeWithRetry(url: string, retries = 2): Promise<any> {
  for (let i = 0; i <= retries; i++) {
    try {
      return await scrapeShop(url);
    } catch (e: any) {
      if (e.message === "RATE_LIMITED" && i < retries) {
        console.log(`[MINO] Rate limited on ${url}, retrying in ${2 ** i}s...`);
        await new Promise((r) => setTimeout(r, 2 ** i * 1000));
        continue;
      }
      console.error(`[MINO] Failed ${url}:`, e.message);
      return null;
    }
  }
}

export async function POST(request: NextRequest) {
  const { city } = await request.json();
  const sites = CITY_SITES[city];

  if (!sites) {
    return NextResponse.json({ error: "Invalid city" }, { status: 400 });
  }

  console.log(`[MINO] Searching ${sites.length} shops in ${city}...`);
  const startTime = Date.now();

  // Fire ALL requests in parallel
  const results = await Promise.allSettled(
    sites.map((url) => scrapeWithRetry(url))
  );

  const shops = results
    .filter((r) => r.status === "fulfilled" && r.value !== null)
    .map((r) => (r as PromiseFulfilledResult<any>).value);

  const elapsed = ((Date.now() - startTime) / 1000).toFixed(1);
  console.log(`[MINO] Done. ${shops.length}/${sites.length} shops returned data in ${elapsed}s`);

  return NextResponse.json({ shops, city, elapsed, total_sites: sites.length });
}
```

---

## What Makes This Use Case Stand Out

1. **Parallel Scale**: 10-18 sites scraped simultaneously — the core Mino advantage, visually demonstrated as results stream in real-time
2. **Zero API Territory**: Not a single rental shop has an API — this is impossible without a web agent
3. **Vietnam-Specific**: Leverages local market knowledge that no other applicant has. Vietnam = uncovered category in the Use Case Library
4. **Real Utility**: Millions of tourists visit Vietnam annually. Every single one rents a motorbike. This solves a daily pain point
5. **Complex Navigation**: Multi-step booking forms, currency switchers, pagination — showcases Mino's AI navigation vs. simple scrapers
6. **Visual Demo**: Cards populating in real-time as agents complete = compelling demo video
7. **No Anti-Bot Risk**: These are small WordPress/Wix sites with zero bot protection — most reliable demo possible

---

## Tech Stack
- **Frontend**: Next.js 15 + Tailwind CSS + shadcn/ui
- **API**: TinyFish Mino SSE streaming endpoint
- **Hosting**: Vercel (free tier)
- **Build Tool**: Claude Code

## Estimated Build Time
- Scaffold + UI: ~30 min (Next.js + Tailwind + shadcn/ui)
- Mino integration + prompt tuning: 1-2 hours
- Frontend polish + real-time streaming UX: 1-2 hours
- Testing across all cities + edge cases: 1 hour
- Demo video recording: 1 hour
- **Total: ~4-6.5 hours**
