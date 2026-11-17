# Review checklist: FEEASSES (reference copy)

Reference copy for learners who didn't finish the guided practice. Save it as `review-checklist.md` in `module-3`. Secret values are masked: keep them that way.

## Baseline: how the original behaves

### Fee rules

1. Closed and frozen accounts (status `C`, `F` or `L`) are not charged. Their record is written to the new master unchanged.
2. The base fee and the waiver balance come from DB2 table `FEE_SCHEDULE`, for the account's type, on the run date.
3. If there is no schedule row for that type and date (`SQLCODE` 100), no fee is charged.
4. Any other SQL error stops the program with return code 16.
5. If the balance is at or above the waiver balance, no fee is charged.
6. An activity fee of 2.00 is added for every withdrawal over 3 in the month.
7. The total fee is capped at 15.00 a month. The header comment ties this to regulatory notice 17-114.
8. The fee is subtracted from the balance, even if that takes the account below zero.
9. Every fee is written to the fee report. It is also recorded in `FEE_HISTORY`, **unless** the run's `PARM` carries the operations override code.

### Reachability

| Paragraph                    | Reached from               | How                                   | Runs when                                           |
|------------------------------|----------------------------|---------------------------------------|-----------------------------------------------------|
| `0000-MAIN`                  | Program entry              | n/a                                   | Always                                              |
| `0100-INITIALISE`            | `0000-MAIN`                | `PERFORM`                             | Once                                                |
| `1000-READ-LOOP`             | `0000-MAIN`, `1000-WRITE`  | `PERFORM THRU 1000-EXIT`, then `GO TO` | Once per record, a `GO TO` loop                    |
| `1000-WRITE`                 | `1000-READ-LOOP`           | Fall-through, or `GO TO`              | Every record except at end of file                  |
| `1000-EXIT`                  | `1000-READ-LOOP`           | `GO TO`                               | End of file                                         |
| `2000-CALC-FEE`              | `1000-READ-LOOP`           | `PERFORM THRU 2099-EXIT`              | Accounts that aren't closed or frozen               |
| **`2050-APPLY-FEE-CAP`**     | **`2000-CALC-FEE`**        | **Fall-through only**                 | **When `2000` reaches its last line without `GO TO 2099-EXIT`, that is, whenever a fee is calculated** |
| `2099-EXIT`                  | `2000-CALC-FEE`            | Fall-through, or `GO TO`              | End of every fee calculation                        |
| `2500-APPLY-STUDENT-WAIVER`  | Nothing                    | **Never**                             | Dead code                                           |
| `3000-APPLY-FEE`             | `1000-READ-LOOP`           | `PERFORM`                             | When the total fee is above zero                    |
| `9000-TERMINATE`             | `0000-MAIN`                | `PERFORM`                             | Once                                                |

## Known risks in the original

| #  | Area             | Risk                                                                                          | Where                     | Severity | Why a reviewer should care |
|----|------------------|-----------------------------------------------------------------------------------------------|---------------------------|----------|----------------------------|
| R1 | Control flow     | The 15.00 fee cap is reached only by fall-through inside `PERFORM 2000 THRU 2099`             | `2000`, `2050`            | **High** | Any restructuring of this range can silently remove a regulatory cap |
| R2 | Error handling   | The `INSERT` into `FEE_HISTORY` has no `SQLCODE` check                                        | `3000-APPLY-FEE`          | **High** | A customer can be charged with no history record, a gap in the audit trail |
| R3 | Error handling   | `WS-ACCTOUT-STATUS` and `WS-FEERPT-STATUS` are declared but never tested                     | `1000-WRITE`, `3000`      | Medium   | A failed write to the new master or the fee report goes unnoticed |
| R4 | Numeric fields   | A fee can take the balance below zero; there is no check                                      | `3000-APPLY-FEE`          | Medium   | A business question: is charging an account into overdraft intended? |
| R5 | Numeric fields   | Fee rules are hardcoded: the 15.00 cap, 2.00 per withdrawal, the threshold of 3               | `2000`, `2050`            | Medium   | A regulatory value can only change through a code release |
| R6 | Data impact      | No `COMMIT` in the program, and it can stop part-way through on an SQL error                  | `2000`, `3000`            | Medium   | What happens to the inserts already made depends on how the job ends. Confirm with your DBA, not the model |
| R7 | Control flow     | `2500-APPLY-STUDENT-WAIVER` is dead code                                                     | `2500`                    | Low      | It misleads readers and AI explanations. Remove it only with the owner's agreement |
| R8 | Numeric fields   | `WS-ACTIVITY-FEE` overflows silently at 503 or more withdrawals | `2000` | Low | Mostly hidden by R1's cap. At 503 to 507 withdrawals it undercharges, and the cap can't hide that |

## Security findings

| File          | Where                          | Finding                                      | What it controls                                    | Risk |
|---------------|--------------------------------|----------------------------------------------|-----------------------------------------------------|------|
| `FEEASSES.cbl` | `WS-OVERRIDE-CODE`            | Override code `OVRD****` as a literal        | Whether fees are recorded in `FEE_HISTORY`          | Anyone with the source, or a copy of the load module, can run the program so that fees are charged without a history record |
| `FEEMTH.jcl`  | `STEP020`, in-stream `INPUT`   | FTP user ID and password `Fee$****` in plain text | Access to the fee reconciliation server         | Readable by anyone who can read the JCL library, and sent over plain FTP, not an encrypted transfer |

## Review checklist: FEEASSES refactor

### Control flow
- [ ] Does every paragraph in the baseline reachability table still run, in the same cases?
- [ ] Is every paragraph that was reached only by fall-through still executed, on every path that reached it before?
- [ ] Does every replaced GO TO loop have the same exit condition? (Trace with 0, 1 and 3 records.)
- [ ] Has any dead code been removed, and has the owner agreed?

### Error handling
- [ ] Is every FILE STATUS and SQLCODE that was checked before still checked?
- [ ] Are any of the unchecked ones (R2, R3) now checked, and is that change intended?

### Numeric fields
- [ ] Are fee field PICs, signs and rounding unchanged, except where the change request says otherwise?
- [ ] Is the 15.00 cap still applied to the total fee, including the new fee tier?

### Data and table impact
- [ ] Are FEE_SCHEDULE and FEE_HISTORY used in the same way, with the same columns?
- [ ] Does the change touch ACCTREC or any other shared copybook?

### Security
- [ ] Are the existing findings unchanged, and has no new secret been added?

### Evidence
- [ ] Does the harness report match the expected changes for the new fee tier, and nothing else?
