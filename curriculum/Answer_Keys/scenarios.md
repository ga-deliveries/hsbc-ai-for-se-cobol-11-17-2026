# scenarios.md: reference copy

Reference copy for learners who didn't finish the guided practice. Save it as `scenarios.md` in `module-2`, with `boundary-records.csv` beside it.

## Expected-change rule

In a daily run, an account should produce different output when it is not closed, frozen (including legal hold) or dormant, and its balance is 10,000.00 or more. Everything else should be identical.

At month-end, rounding to the cent can absorb the difference, so "should change" means "may change". S05 capitalises to 10,000.62 in both versions.

## Boundary scenarios for the rate change

Runs: D = `20261030,N`, M = `20261031,Y`, L = `20280229,N`. Values are the daily interest added to `ACCT-ACCRUED-INT`, unless the row says otherwise. All accounts are type `SV`, status `A`, unless noted.

| ID | Input | Expected: original | Expected: modified | Should change? | Why it matters |
|---|---|---|---|---|---|
| S01 | Balance 0.00, run D | No accrual | No accrual | No | Zero balance is excluded by the `EVALUATE` |
| S02 | Balance -0.01, run D | No accrual | No accrual | No | Overdrawn accounts earn nothing; tests the `D` sign half-byte |
| S03 | Balance 1,000.00, run D | 0.0342 | 0.0342 | No | Exactly on the tier 2 floor, so 1.25% applies |
| S04 | Balance 999.99, run D | 0.0136 | 0.0136 | No | One cent below tier 2, so 0.50% applies |
| S05 | Balance 10,000.00, run D | 0.6219 | 0.6232 | **Yes** | Exactly on the tier 3 floor |
| S06 | Balance 9,999.99, run D | 0.3424 | 0.3424 | No | One cent below tier 3 must stay on tier 2 |
| S07 | Balance 50,000.00, run D | 4.2739 | 4.2808 | **Yes** | Exactly on the tier 4 floor |
| S08 | Type `BS`, bonus `Y`, 0 withdrawals, 20,000.00, run D | 1.5178 | 1.5205 | **Yes** | Tier 3 plus the 0.500 bonus |
| S09 | Status `D`, balance 20,000.00, run D | 0.2739 | 0.2739 | No | Dormant stays on tier 1 |
| S10 | The S03 record, run L | 0.0341 | 0.0341 | No | Leap year divides by 366 |
| S11 | Balance 100.00, accrued 0.0037, run M | Balance 100.01, accrued 0 | Same | No | Accrued reaches 0.0050 and rounds up |
| S12 | Balance 100.00, accrued 0.0036, run M | Balance 100.00, accrued 0 | Same | No | Accrued reaches 0.0049 and rounds down |
| S13 | Balance 9,999,999.99, run M | Overflow; balance unchanged, accrued reset to 0 | Same | No | Interest lost in both versions (Module 1, rule 12) |
| S14 | Balance 1,234.56, run D | 0.0422 | 0.0422 | No | Positive packed value, `C` sign half-byte |
| S15 | Balance 49,999.99, run D | 3.1095 | 3.1164 | **Yes** | One cent below tier 4 stays on tier 3, whose rate changes |
| S16 | Balance 10,000.00, accrued 0.0030, run M | Balance 10,000.62, accrued 0 | Balance 10,000.63, accrued 0 | **Yes** | The daily difference survives month-end rounding: 10,000.6249 rounds to .62, 10,000.6262 to .63 |

**Packed values:**
- S02 = `X'000000001D'`
- S13 = `X'999999999C'`
- S14 = `X'000123456C'`

Run `python3 harness/pack-records.py --hex boundary-records.csv` to show them.

**S05 and S16 together:** at month-end, S05 (no accrued interest) rounds to 10,000.62 in both versions, but S16 starts with 0.0030 accrued and ends one cent apart. Same balance, same rate change: whether month-end shows the difference depends on what is already accrued. That's why the rule says "may change" at month-end.
