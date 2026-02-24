# Task 1: Mino API Validation Report

**Date**: 2026-02-24  
**Status**: BLOCKED - API Key Not Available

## Summary

Attempted to validate the TinyFish Mino API against 4 Vietnam motorbike rental websites. Validation could not proceed due to missing API key in the environment.

## Blocker

**Issue**: The environment variable `TINYFISH_API_KEY` (or `TINY_FISH_API_KEY`) is not set in the bash environment.

**Expected**: According to task instructions, the API key should be "already set in the environment" and available as `$TINYFISH_API_KEY`.

**Actual**: 
- Checked `env | grep -i "api\|key"` — no results
- Checked `printenv | grep -i "api\|key"` — no results
- Attempted to load from `.env` file — file does not exist
- Attempted to source `.env` — file not found

## Test Sites Prepared

The following sites were prepared for testing (one per city):

| City | Site | URL |
|------|------|-----|
| HCMC | Wheelie Saigon | https://wheelie-saigon.com/scooter-motorcycle-rental-hcmc-daily-weekly-or-monthly/ |
| Hanoi | Motorbike Rental in Hanoi | https://motorbikerentalinhanoi.com/ |
| Da Nang | Hoi An Bike Rental | https://hoianbikerental.com/pricing/ |
| Nha Trang | Motorbike Mui Ne | https://motorbikemuine.com/ |

## Mino Goal Prompt

The extraction prompt was prepared and saved to `/tmp/mino_goal.txt`. It includes:
- Navigation to pricing pages
- Popup handling
- Bike listing extraction (name, engine size, type, prices, deposit, availability)
- Load More button handling
- JSON output structure with required fields

## API Endpoint Details

- **Endpoint**: `POST https://agent.tinyfish.ai/v1/automation/run-sse`
- **Auth Header**: `X-API-Key: <TINYFISH_API_KEY>`
- **Content-Type**: `application/json`
- **Body**: `{"url": "...", "goal": "..."}`
- **Response**: SSE stream with events (STARTED, STREAMING_URL, PROGRESS, HEARTBEAT, COMPLETED)

## Next Steps

To proceed with validation:

1. **Provide API Key**: Set the `TINYFISH_API_KEY` environment variable with the actual TinyFish API key
2. **Alternative**: If the key is stored elsewhere (e.g., in a secrets manager, CI/CD environment), provide instructions on how to access it
3. **Verify Env Var Name**: Confirm whether the correct env var name is:
   - `TINYFISH_API_KEY` (as per task instructions)
   - `TINY_FISH_API_KEY` (as per error feedback)
   - `MINO_API_KEY` (as per use-case-brief.md)

## Curl Command Template

Once the API key is available, the validation can proceed with:

```bash
export TINYFISH_API_KEY="<your-api-key>"
GOAL=$(cat /tmp/mino_goal.txt)

curl -N -X POST "https://agent.tinyfish.ai/v1/automation/run-sse" \
  -H "X-API-Key: $TINYFISH_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"url\": \"https://wheelie-saigon.com/scooter-motorcycle-rental-hcmc-daily-weekly-or-monthly/\", \"goal\": $(echo "$GOAL" | jq -Rs .)}" \
  --max-time 120
```

## Evidence Files

- `task-1-report.md` — This report
- `task-1-api-validation.json` — Will contain raw API responses once key is available

---

**Recommendation**: Unblock by providing the API key, then re-run validation against all 4 sites.
