# Task 1: Issues & Blockers

## BLOCKER: Missing API Key

**Status**: BLOCKED  
**Date**: 2026-02-24  
**Severity**: CRITICAL

### Issue
Task 1 requires the environment variable `TINYFISH_API_KEY` to be set, but it is not available in the bash environment.

### Root Cause
- Task instructions state: "Use env var `TINYFISH_API_KEY` — it is already set in the environment"
- Actual state: Environment variable not found
- .env file does not exist in the project directory
- No alternative mechanism to access the API key

### Discrepancies Found
1. **Task instructions** say: `TINYFISH_API_KEY`
2. **use-case-brief.md** code uses: `MINO_API_KEY`
3. **Error feedback** mentioned: `TINY_FISH_API_KEY`

### Impact
- Cannot test Mino API against real Vietnam motorbike rental websites
- Cannot validate that the API returns usable bike pricing data
- Blocks Task 1 and downstream tasks (Task 3, 7, 8 depend on Task 1)

### What Was Prepared
- ✓ Evidence directory created
- ✓ Mino goal prompt prepared (5-step extraction)
- ✓ Test sites identified (1 per city)
- ✓ API endpoint verified
- ✓ Curl command template ready
- ✓ Report files created

### Resolution
1. Provide the `TINYFISH_API_KEY` environment variable with actual API key
2. Clarify the correct environment variable name
3. Re-run Task 1 validation

### Files Created
- `.sisyphus/evidence/task-1-report.md` - Detailed blocker report
- `.sisyphus/evidence/task-1-api-validation.json` - Structured validation data
- `.sisyphus/evidence/task-1-blocker-summary.txt` - Quick reference

---

## Env Var Name Confusion

The codebase has inconsistent naming for the API key:
- AGENTS.md plan: `TINYFISH_API_KEY`
- use-case-brief.md: `MINO_API_KEY`
- Error feedback: `TINY_FISH_API_KEY`

**Recommendation**: Standardize on one name and update all references.
