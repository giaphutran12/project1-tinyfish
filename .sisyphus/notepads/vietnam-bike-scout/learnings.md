# Task 1: Learnings & Patterns

## Mino API Details Confirmed

### Endpoint
- **URL**: `POST https://agent.tinyfish.ai/v1/automation/run-sse`
- **Auth**: `X-API-Key: <TINYFISH_API_KEY>` header
- **Content-Type**: `application/json`

### Request Body
```json
{
  "url": "https://example.com",
  "goal": "Extract motorbike rental pricing..."
}
```

### Response Format
- **Type**: Server-Sent Events (SSE) stream
- **Event Types**: STARTED, STREAMING_URL, PROGRESS, HEARTBEAT, COMPLETED
- **Parsing**: Read lines starting with `data: `, JSON-parse each
- **Final Event**: Contains `status: "COMPLETED"` and `resultJson` with extracted data

### Key Implementation Notes
1. **SSE Parsing**: Must use `getReader()` + buffer pattern, NOT `await response.text()`
2. **Timeout**: Use `--max-time 120` for curl (30-90s typical, up to 120s max)
3. **Rate Limiting**: Stagger requests 500ms-2s apart to avoid queuing
4. **Streaming URL**: Log the `streamingUrl` field when received (useful for debugging)

## Vietnam Motorbike Rental Sites

### Test Sites Identified (1 per city)
| City | Site | URL | Complexity |
|------|------|-----|-----------|
| HCMC | Wheelie Saigon | https://wheelie-saigon.com/scooter-motorcycle-rental-hcmc-daily-weekly-or-monthly/ | Simple listing |
| Hanoi | Motorbike Rental Hanoi | https://motorbikerentalinhanoi.com/ | Simple listing |
| Da Nang | Hoi An Bike Rental | https://hoianbikerental.com/pricing/ | Simple + dedicated pricing page |
| Nha Trang | Motorbike Mui Ne | https://motorbikemuine.com/ | Simple listing |

### Extraction Schema
The Mino goal prompt extracts:
- Bike name/model
- Engine size (cc)
- Bike type (scooter, semi-auto, manual, adventure)
- Daily/weekly/monthly prices (USD)
- Currency (VND or USD)
- Deposit amount
- Availability status

### Price Conversion
- VND to USD: 1 USD ≈ 25,000 VND
- If price > 1000, assume VND and convert

## Environment Variable Naming

**Issue**: Inconsistent naming across codebase
- Plan (AGENTS.md): `TINYFISH_API_KEY`
- Brief (use-case-brief.md): `MINO_API_KEY`
- Error feedback: `TINY_FISH_API_KEY`

**Recommendation**: Standardize on `TINYFISH_API_KEY` (matches plan)

## Evidence Files Structure

Created in `.sisyphus/evidence/`:
1. `task-1-report.md` - Detailed blocker report with next steps
2. `task-1-api-validation.json` - Structured validation data (ready for API responses)
3. `task-1-blocker-summary.txt` - Quick reference for unblocking

## Curl Command Template

Ready to use once API key is available:
```bash
export TINYFISH_API_KEY="<your-api-key>"
GOAL=$(cat /tmp/mino_goal.txt)

curl -N -X POST "https://agent.tinyfish.ai/v1/automation/run-sse" \
  -H "X-API-Key: $TINYFISH_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"url\": \"$SITE\", \"goal\": $(echo "$GOAL" | jq -Rs .)}" \
  --max-time 120
```

## Next Steps

1. **Unblock**: Provide `TINYFISH_API_KEY` environment variable
2. **Validate**: Run 4 sequential curl requests (one per site)
3. **Parse**: Extract bike data from SSE responses
4. **Report**: Generate final validation report with results

---

**Status**: Ready to proceed once API key is available
**Estimated Time to Complete**: 5-10 minutes (4 sites × 30-90s each + parsing)

## Task 2: SSE Proxy Route Implementation

- Added Edge runtime API route at `src/app/api/search/route.ts` with `export const runtime = "edge"`.
- Implemented city-based fan-out using hardcoded `CITY_SITES` and `POST { city }` input validation.
- Mino calls now use `POST https://agent.tinyfish.ai/v1/automation/run-sse` with `X-API-Key: process.env.TINYFISH_API_KEY`.
- SSE parsing uses `response.body.getReader()` with line-buffer handling (`data: ` parsing), no `await response.text()`.
- Per-site request guardrails: 90s `AbortController` timeout + 500ms stagger per request to reduce queue contention.
- Route streams custom SSE events:
  - per completed site: `{ type: "SHOP_RESULT", shop: ... }`
  - final summary: `{ type: "SEARCH_COMPLETE", total, succeeded, elapsed }`
- Failure isolation is handled via `Promise.allSettled()` so one bad site cannot kill the batch.
- Added operational logs:
  - `[MINO] Starting: <url>`
  - `[MINO] streamingUrl <url>` when emitted by Mino
  - `[MINO] Complete: <url> (<elapsed>s)`

## use-bike-search Hook Implementation

### Key Patterns
- **SSE Stream Parsing**: Used `getReader()` + `TextDecoder` + buffer pattern (same as server)
- **AbortController**: Manages both fetch abort and reader cancellation for cleanup
- **State Management**: Incremental shop appending on `SHOP_RESULT`, final state on `SEARCH_COMPLETE`
- **Normalization**: `normalizeShop()` coerces types, filters empty bikes, ensures arrays

### Type Safety
- Exported `Bike`, `BikeShop`, `SearchState` interfaces for consumer components
- Hook returns exact interface: `{ state, search, abort }`
- All price fields nullable (not all sites provide all pricing tiers)

### Error Handling
- Distinguishes AbortError (user-initiated) from network/parse errors
- Sets error message on failure, clears on new search
- Reader cleanup in finally block

### Build Status
- ✓ TypeScript compilation clean
- ✓ No new dependencies added
- ✓ Ready for consumer component integration

## T1 Da Nang Result
- Site: hoianbikerental.com
- Status: SUCCESS
- Bikes found: 16
- Sample: Honda Air Blade 110 (110cc scooter, $5.95/day)

### Key Findings
- Mino SSE streaming works reliably for Vietnam sites
- API extracted structured data with correct schema (engine_cc as number, prices as USD)
- Site has no weekly/monthly pricing (all null) - daily only
- No deposit required (all 0)
- All bikes marked available: true
- Rich notes field captured: helmet types, delivery, 24/7 support
- Execution time: ~2 minutes (55 seconds from start to completion)

### Technical Notes
- SSE streaming shows real-time progress updates (PROGRESS events)
- HEARTBEAT event at 51s mark (normal)
- Final COMPLETE event contains full resultJson
- No errors or failures during extraction


## UI Implementation
- Replaced `src/app/page.tsx` with the Vietnam Bike Price Scout UI.
- Used `useBikeSearch` hook for state management.
- Implemented city selection with `Button` components.
- Added progress bar and results placeholder.
- Verified build passes with `npm run build`.
- Note: The progress bar total is only known at the end of the search, so it might jump from 0% to 100%. Added a small visual indicator (5% width) during search to show activity.
## T1 HCMC Result
- Site: wheelie-saigon.com
- Status: SUCCESS
- Bikes found: 15
- Sample: Yamaha WR155 Super Motard (155cc, adventure, $40/day, $20 deposit)

### Key Findings
- Mino successfully extracted all 15 bikes from the rental page
- Prices are in VND on the site, Mino converted to USD using 1 USD = 25,000 VND rate
- Daily prices range: $6-$60 USD
- All bikes have $20 USD deposit requirement
- Weekly/monthly fixed prices not listed on site (20-30% discounts mentioned)
- Bike types: scooter (8), manual (4), adventure (1), semi-auto (1)
- Engine sizes: 110cc-400cc range
- Shop includes free delivery/pickup, helmet, fuel, insurance

### API Performance
- Total execution time: ~2 minutes 16 seconds
- Multiple navigation steps required (visited individual bike pages)
- SSE streaming worked smoothly
- Result JSON properly formatted and complete
## T1 Hanoi Result
- Site: motorbikerentalinhanoi.com
- Status: SUCCESS
- Bikes found: 29
- Sample: Tiger 900 Rally (adventure, 900cc, $115/day)
- Shop: Gia Hưng Motorbike Rental
- API Runtime: ~9 minutes (15:32:45 - 15:41:23)
- Key Finding: Mino successfully extracted structured data including engine_cc, type, daily/weekly/monthly pricing, and deposit amounts across all bike categories (scooter, semi-auto, manual, adventure)
