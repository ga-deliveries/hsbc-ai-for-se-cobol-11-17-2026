# INTCALC – Business Logic Spec (reference copy)

Program: INTCALC | Job/step: EODINT / STEP030 | Status: verified against source
Reference copy for learners who didn't finish Module 1. Save it as `INTCALC-spec.md` in `module-2`.

Line numbers refer to `module-1/INTCALC.cbl`.

## Purpose
Accrues one day of interest on every savings account each night, and on month-end runs adds the accrued interest to the balance. It writes the next generation of the account master.

## Inputs and outputs
| COBOL file | DD name | Dataset | Open mode | FILE STATUS field | Record layout |
|---|---|---|---|---|---|
| ACCT-IN-FILE | ACCTIN | PROD.SAV.ACCTMAST.SORTED (sorted in STEP020) | INPUT | WS-ACCTIN-STATUS | ACCT-REC (ACCTREC.cpy) |
| RATE-FILE | RATEIN | PROD.SAV.RATETIER.CURRENT | INPUT | WS-RATE-STATUS | RATE-IN-REC |
| ACCT-OUT-FILE | ACCTOUT | PROD.SAV.ACCTMAST(+1), a new generation | OUTPUT | WS-ACCTOUT-STATUS (declared, never tested) | ACCT-OUT-REC, written from ACCT-REC |
| EXCEPTION-FILE | EXCPTN | SYSOUT | OUTPUT | not in code | EXCPTN-REC |

PARM `'20261030,N'` → LK-RUN-DATE (run date) and LK-MONTH-END-SW (month-end switch).

## Business rules
| # | Rule | Source (paragraph, lines) | Status |
|---|---|---|---|
| 1 | WHEN status is C (closed) THEN no interest is accrued and the record is not written to the new master: closed accounts are removed | 1000-PROCESS-ACCOUNT, 158–159, 174–177 | Verified |
| 2 | WHEN status is F (frozen) or L (legal hold) THEN no accrual and exception "FROZEN - NO ACCRUAL". The record is still written. "Legal hold" comes from the ACCTREC change-log comment (line 12), not code | ACCTREC 12, 23; 1000, 160–162 | Verified |
| 3 | WHEN balance is zero or negative THEN no accrual, no exception | 1000, 163–164 | Verified |
| 4 | WHEN status is D (dormant) THEN tier 1 rate, whatever the balance. Only for a positive balance: a dormant account with a zero or negative balance matches rule 3 first and gets no rate. That tier 1 is the lowest rate is not in code | 2100-DETERMINE-TIER, 205–206; 1000, 163 | Verified |
| 5 | Rate = highest tier whose floor ≤ balance, applied to the whole balance; a balance on a floor gets the higher tier | 2100, 207–216 | Verified |
| 6 | WHEN type BS AND bonus flag Y AND withdrawals this month = 0 THEN + 0.50 (bonus rate lives in RATETIER.cpy). Includes dormant Bonus Savers | 2000-ACCRUE-INTEREST, 185–189 | Verified |
| 7 | Daily interest = balance × rate ÷ 100 ÷ days in year, **truncated** to 4 dp (no ROUNDED). The header comment (line 12) saying "rounded" is wrong | 2000, 191–192 | Corrected |
| 8 | Days in year = 366 when run year divides by 4, else 365 (wrong for 2100) | 0100-INITIALISE, 128–134 | Verified |
| 9 | WHEN the daily add overflows ACCT-ACCRUED-INT THEN the day is not added, exception written; still counted as accrued | 2000, 194–201 | Verified |
| 10 | WHEN PARM month-end switch = Y THEN accrued added to balance, ROUNDED to 2 dp, for every non-closed account (incl. frozen, dormant, overdrawn) | 1000, 169–172; 3000-CAPITALISE-INTEREST | Verified |
| 11 | After capitalisation: accrued reset to 0, last-interest date = run date, withdrawals = 0 | 3000, 232–234 | Verified |
| 12 | WHEN capitalisation overflows the balance THEN balance unchanged but accrued still reset: interest lost | 3000, 222–232 | Verified |
| 13 | Only the first 4 rate records are loaded; extra tiers ignored silently | 0100, 136–138; 0200-LOAD-RATE-TABLE | Verified |
| 14 | WHEN no tier floor is at or below the balance THEN WS-APPLIED-RATE is not set, so the previous account's rate (possibly including the 0.50 bonus) is used. No VALUE clause, so the first account's rate is undefined. Not reachable with the current rate file (tier 1 floor 0) | 2100, 208–216; WS-APPLIED-RATE 84 | Verified |
| 15 | WHEN status is anything other than C, F, L or D (including spaces) THEN the account is accrued like an active account: status values are never validated | 1000, 165–166 | Verified |
| 16 | Only the ACCTIN and RATEIN opens are checked (RC 16). WS-ACCTOUT-STATUS is never tested, EXCPTN has no FILE STATUS, and the PARM (length, date, Y/N) is never validated | 0100, 119–124; 32, 33, 60, 97 | Verified |

## Calculations
- Daily: `COMPUTE WS-DAILY-INT = ACCT-BALANCE * WS-APPLIED-RATE / 100 / WS-DAYS-IN-YEAR`. WS-DAILY-INT is S9(5)V9(4): truncated to 4 dp.
- Month-end: `COMPUTE ACCT-BALANCE ROUNDED = ACCT-BALANCE + ACCT-ACCRUED-INT`. ACCT-BALANCE is S9(7)V99: rounded to 2 dp, halves away from zero.

## Open questions for the business
- Is capitalising accrued interest on frozen and legal-hold accounts intended? (Rule 10)
- Is the loss of accrued interest on balance overflow known and accepted? (Rule 12)
- Closed accounts are dropped from the new master with no exception. Does another process keep them? (Rule 1)
- Can a balance fall below every tier floor, and what rate should apply then? (Rule 14)
- Is it intended that a dormant account with a zero or negative balance gets no accrual and no rate? (Rule 4)
- What does status F mean, and should legal hold (L) have its own exception message? (Rule 2)

## Shared copybooks and impact
ACCTREC, RATETIER: users per header comment are STMTGEN, FEEASSES and ACCTUPD. Not yet confirmed by a repository search; only INTCALC and EODINT are in the Module 1 workspace.
