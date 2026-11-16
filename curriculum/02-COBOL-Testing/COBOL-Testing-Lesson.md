![General Assembly](/GA_Horiz_Small_RedBlack_RGB@2x.png)

# Module 2: COBOL Testing
 
| Topic                                                                  | Minutes |
|------------------------------------------------------------------------|---------|
| Scenarios before data: the reviewable scenario table                   | 5       |
| From scenario to records that match the copybook                       | 5       |
| Guided practice: scenario table and boundary records for the rate change | 20    |
 
**Total lesson run time:** 30 min
 
## Table of Contents
 
- [Learning Objectives](#learning-objectives)
- [What You'll Need](#what-youll-need)
- [Introduction: prove nothing else moved](#introduction-prove-nothing-else-moved)
- [Scenarios before data: the reviewable scenario table](#scenarios-before-data-the-reviewable-scenario-table)
- [From scenario to records that match the copybook](#from-scenario-to-records-that-match-the-copybook)
- [Guided practice: scenario table and boundary records for the rate change](#guided-practice-scenario-table-and-boundary-records-for-the-rate-change)
    - [Step 1, pin down the change and what should change (4 minutes)](#step-1-pin-down-the-change-and-what-should-change-4-minutes)
    - [Step 2, build the scenario table (8 minutes)](#step-2-build-the-scenario-table-8-minutes)
    - [Step 3, generate the boundary records (6 minutes)](#step-3-generate-the-boundary-records-6-minutes)
    - [Step 4, review coverage (2 minutes)](#step-4-review-coverage-2-minutes)
- [Check for Understanding](#check-for-understanding)
- [Summary](#summary)
- [Vocabulary](#vocabulary)
## Learning Objectives
 
- Turn a change request into an explicit list of which accounts *should* change, before any test runs.
- Prompt Copilot Chat for a reviewable scenario table (input, expected result, why it matters) built from the program's real decision points.
- Find the boundary values in a COBOL calculation: branch conditions, tier floors, `PIC` limits, truncation and rounding points, date logic and sign.
- Generate synthetic boundary records that match a copybook layout, including `COMP-3` fields, and verify their offsets and packed encoding.
- Check AI-calculated expected results by hand, and catch where the model rounded when the code truncates.
- Review scenario coverage against the Module 1 spec, and decide which gaps are worth closing.
## What You'll Need
 
Everything runs in VS Code. You'll prompt and review in this lesson; you'll run the harness in the Lab.
 
- VS Code with the course COBOL extension, and GitHub Copilot Chat signed in
- The `module-2/` folder from the course pack, open as your workspace:
  - `INTCALC.cbl` and `RATETIER.cpy`, the current production versions from Module 1
  - `INTCALC-v2.cbl` and `RATETIER-v2.cpy`, the modified versions for this change
  - `ACCTREC.cpy`, unchanged
  - `rates-current.csv` and `rates-new.csv`, the old and new rate tier files
  - `harness/`, the pre-built tools you'll use in the Lab, including `pack-records`, which turns a CSV of `ACCTREC` field values into the 100-byte fixed-width file
- Your `INTCALC-spec.md` from Module 1. You'll reuse it as context.
- Two new, empty files: `scenarios.md` and `boundary-records.csv`
> [!IMPORTANT]
> Test data in this course is generated from the copybook layout, never copied from production. Do the same on your own work. Copying real account records into a test file, or into a prompt, takes customer data somewhere it should not go, even when the intention is only to "get realistic values".
 
## Introduction: prove nothing else moved
 
The business has asked for more precise savings rates. Today, tier rates in `INTCALC` carry two decimal places, so a product priced at 2.275% has been running at 2.27%. The change request is simple: carry rates to three decimal places.
 
A developer has made the change. It touches a copybook and a handful of field definitions. It looks small, and that is exactly what makes it risky. `RATETIER` is a shared copybook, every account on the master file passes through this program every night, and interest is customer money.
 
Your job is to prove two things before this goes anywhere near production. The accounts that *should* change do change, by the right amount. And every other account in the 5,000-account test file produces **identical** output.
 
Today this takes days. Someone works out the cases by hand, builds packed-decimal test records field by field in a hex editor, runs the job, and eyeballs the output. The work is slow, so the cases get thin, and the thin cases are the ones that miss the bug.
 
This session changes the order of that work. You decide *what* to test, with AI helping you find the cases, and get that agreed in a table a reviewer can read. Only then do you generate the data. And at every step, you check the numbers the model gives you, because models are fluent at COBOL and unreliable at arithmetic.
 
## Scenarios before data: the reviewable scenario table
**Time:** 5 minutes
 
The most common mistake in AI-assisted testing is asking for test data first: "Generate 20 test records for INTCALC." You get 20 records. You have no idea which cases they cover, what the right answer for each one is, or what is missing. Nobody can review a file of packed decimal.
 
Ask for scenarios first, in a table a person can read and challenge.
 
**Figure 01,** the scenario table
 
| Column                   | What goes in it                                          | Why it's there                                                         |
|--------------------------|----------------------------------------------------------|------------------------------------------------------------------------|
| **ID**                   | `S01`, `S02` …                                           | Ties the scenario to its test record and to the harness report         |
| **Input**                | The fields that matter, and the run (`PARM`)              | A reviewer can see the case without decoding a record                  |
| **Expected result**      | The value, for both the original and the modified program | This is the answer key. Without it, any output looks plausible         |
| **Should change?**       | Yes or No                                                 | Separates *expected* differences from *unexplained* ones in the Lab    |
| **Why it matters**       | The rule or risk the case protects                        | Stops the table filling up with cases that test nothing                |
 
The **Should change?** column is the one people leave out, and it's the one that makes regression testing work. When the Lab report says "M changed", you can only call that good news if you wrote down beforehand which accounts were meant to change. Otherwise, a change that silently did nothing produces zero differences and looks like a perfect pass.
 
**Where the boundaries come from.** Good scenarios aren't invented; they're read out of the code. Every decision point, size limit and rounding step in the program is a boundary, and each boundary gets a value on it and a value just beside it. The Module 1 spec already lists most of them.
 
**Figure 02,** where the boundaries are in `INTCALC`
 
| Source of the boundary         | Where it is in `INTCALC`                                          | Values worth testing                                       |
|--------------------------------|--------------------------------------------------------------------|------------------------------------------------------------|
| Branch conditions              | `EVALUATE` in `1000-PROCESS-ACCOUNT`: closed, frozen, balance ≤ 0 | Each status, a balance of 0.00 and of -0.01                |
| Tier floors                    | 1,000.00, 10,000.00 and 50,000.00 in the rate file                 | Exactly on each floor, and one cent below                  |
| `PIC` limits                   | `ACCT-BALANCE PIC S9(7)V99`, `ACCT-ACCRUED-INT PIC S9(5)V9(4)`     | The maximum value, and a value that overflows after adding |
| Truncation and rounding        | Daily interest truncated to 4 places; month-end `ROUNDED` to 2     | A half cent at capitalisation, and just below it           |
| Date logic                     | Days in year from the run year divided by 4                        | A leap-year run and a normal run                           |
| Storage format                 | `COMP-3` sign half-byte                                            | A negative and a positive packed value                     |
| The change itself              | Rates move from 2 to 3 decimal places                              | Tiers whose rate gains a third decimal, and tiers that don't |
 
**The prompt pattern.** Give the model the program, the copybooks, the change, and your Module 1 spec. Ask for boundaries by decision point, not "some test cases". Demand the table format. Then ask it to show its working for every expected value, because you're going to check them.
 
> **Discuss:**
>
> Think about the last change you tested. If someone had asked you to write the **Should change?** column before you ran anything, how confident would you have been in it?
 
## From scenario to records that match the copybook
**Time:** 5 minutes
 
Once the table is agreed, each scenario becomes a record. This is where the hours go today, and it's where AI saves the most time, as long as you check the layout.
 
**The layout is the copybook.** Every record in the account file is 100 bytes, laid out exactly as `ACCTREC.cpy` says. Ask the model to derive the offsets from the copybook, then check that they add up to the record length.
 
**Figure 03,** the `ACCTREC` layout, derived from the copybook
 
| Field                  | `PIC`             | Format   | Offset | Length |
|------------------------|-------------------|----------|--------|--------|
| `ACCT-NUMBER`          | `X(12)`           | Text     | 1      | 12     |
| `ACCT-TYPE`            | `X(02)`           | Text     | 13     | 2      |
| `ACCT-STATUS`          | `X(01)`           | Text     | 15     | 1      |
| `ACCT-BALANCE`         | `S9(7)V99`        | `COMP-3` | 16     | 5      |
| `ACCT-ACCRUED-INT`     | `S9(5)V9(4)`      | `COMP-3` | 21     | 5      |
| `ACCT-LAST-INT-DATE`   | `9(08)`           | Zoned    | 26     | 8      |
| `ACCT-BONUS-FLAG`      | `X(01)`           | Text     | 34     | 1      |
| `ACCT-WITHDRAWALS-MTD` | `9(03)`           | Zoned    | 35     | 3      |
| `FILLER`               | `X(63)`           | Text     | 38     | 63     |
|                        |                   |          | **Total** | **100** |
 
**Packed decimal in one minute.** A `COMP-3` field stores two digits per byte, with the sign in the last half-byte: `C` for positive, `D` for negative. Its length is the number of digits plus one, divided by two, rounded up. `S9(7)V99` has nine digits, so it takes five bytes. The decimal point is implied; it is never stored.
 
So a balance of -0.01 is stored as `X'000000001D'`, and 1,234.56 as `X'000123456C'`.
 
**How the course harness handles it.** You don't hand-build bytes. You write one CSV row per record, one column per `ACCTREC` field, in readable decimal. `pack-records` in the harness folder packs the `COMP-3` fields and writes the fixed-width file. Your job is to get the values right and to check a few packed values yourself, because that's the skill you'll need on your own team's files. Your instructor will show it now: in the `module-2` terminal, `python3 harness/pack-records.py --hex demo.csv` packs four demo records and prints each packed field in hex, including the two values above.
 
**Figure 04,** layout mistakes AI makes with test records
 
| Mistake                                  | What it looks like                                     | How to catch it                                                     |
|------------------------------------------|--------------------------------------------------------|---------------------------------------------------------------------|
| Wrong `COMP-3` length                    | Treats `S9(7)V99` as 9 or 10 bytes                     | Check the offsets add up to the record length                       |
| Decimal point in the data                | `0001234.56` written into a numeric field              | The `V` is implied; numeric fields never contain a `.`              |
| Unsigned sign half-byte                  | Positive values packed with `F` instead of `C`         | Ask for the hex of a positive and a negative value, and read the last half-byte |
| Text fields not padded                   | An 8-character account number in a 12-byte field      | Text fields are padded with spaces to their full length             |
| Values that break the `PIC`              | A balance of 12,000,000.00 in `S9(7)V99`               | Compare every value with the field's maximum before packing         |
 
> **Discuss:**
>
> How does your team build packed-decimal test data today? Which of the mistakes in Figure 04 has cost you time before?
 
## Guided practice: scenario table and boundary records for the rate change
**Time:** 20 minutes
 
Work through four steps with your instructor. Run each prompt in your own Copilot Chat, compare your answer with the room's, then check it before moving on. By the end you'll have `scenarios.md` and `boundary-records.csv`, ready for the Lab. To paste an answer into your file without Copilot's working log, end each prompt with *"Put your whole answer in a single Markdown code block"* and use the code block's **Copy** icon.
 
Your wording won't match the expected results exactly. Your **numbers** should, and where they don't, the source code is the referee.
 
### Step 1, pin down the change and what should change (4 minutes)
 
Start with the change itself. You can't define "should change" until you know exactly what the developer touched.
 
```text
#file:INTCALC.cbl #file:INTCALC-v2.cbl #file:RATETIER.cpy #file:RATETIER-v2.cpy
#file:rates-current.csv #file:rates-new.csv #file:INTCALC-spec.md
 
Change request: tier interest rates move from 2 to 3 decimal places.
 
1. List every line that differs between the original and the v2 versions,
   with the field, its old PIC and its new PIC.
2. Compare the two rate files and say which tier rates change in value.
3. Using the business rules in INTCALC-spec.md, state exactly which
   accounts should produce different output after this change, and which
   should not. Express it as a rule on ACCT-STATUS, ACCT-TYPE and
   ACCT-BALANCE.
4. List anything outside INTCALC that this change affects.
Cite line numbers. If something is not in the code, write "not in code".
```
 
**Expected result**
 
The developer changed four definitions:
 
| File              | Field              | Old `PIC`    | New `PIC`     |
|-------------------|--------------------|--------------|---------------|
| `RATETIER.cpy`    | `RT-TIER-RATE`     | `9(02)V99`   | `9(02)V999`   |
| `RATETIER.cpy`    | `RT-BONUS-RATE`    | `9(02)V99`   | `9(02)V999`   |
| `INTCALC.cbl`     | `RI-TIER-RATE-PCT` | `9(02)V99`   | `9(02)V999`, with `FILLER` reduced from 71 to 70 bytes |
| `INTCALC.cbl`     | `WS-APPLIED-RATE`  | `9(02)V99`   | `9(02)V999`   |
 
Only tiers 3 and 4 change in value: 2.27% becomes 2.275%, and 3.12% becomes 3.125%. Tiers 1 and 2 and the bonus rate are unchanged.
 
**The expected-change rule:** in a daily run, an account should produce different output when it is not closed, frozen or dormant, and its balance is 10,000.00 or more. Everything else should be identical.
 
Month-end runs need one extra line in the rule, and the model will rarely volunteer it. At capitalisation the balance is rounded to the cent, which can swallow a difference smaller than half a penny. The S05 account in Step 2, for example, capitalises to 10,000.62 in both versions. So at month-end, "should change" means "may change", and a rounded-away difference is expected, not a failure. It can go either way: S16 has the same balance but starts with a little accrued interest, and ends one cent apart.
 
Outside `INTCALC`, there are two things to raise. Every program that copies `RATETIER` must be recompiled, because the table layout has changed. Your Module 1 spec suggests `STMTGEN`, which prints the rate on statements, may be one of them. And the rate file's layout has changed, so the upstream job that writes `RATETIER.CURRENT` has to be released at the same time as `INTCALC`, or tonight's run reads the new file with the old layout.
 
**Check it against the source.** The fourth line in that table is the one that matters most. If the developer had changed the copybook but missed `WS-APPLIED-RATE`, the third decimal would be truncated away inside `INTCALC`. The output would be **identical** to today's, and a regression test without a **Should change?** column would call it a pass.
 
Write the expected-change rule at the top of `scenarios.md`.
 
### Step 2, build the scenario table (8 minutes)
 
Now ask for the scenarios, using the boundaries from Figure 02 and the change from Step 1.
 
```text
#file:INTCALC.cbl #file:INTCALC-v2.cbl #file:ACCTREC.cpy #file:RATETIER-v2.cpy
#file:rates-current.csv #file:rates-new.csv #file:INTCALC-spec.md
 
Act as a test analyst. Build a boundary scenario table for this change.
Cover: zero and negative balance, the exact tier 2, 3 and 4 floors and
one cent below them, a bonus saver, a dormant account, the maximum
ACCT-BALANCE, a leap-year run, half-cent rounding at month-end
capitalisation, a month-end account where the rate change survives
rounding to the cent, and the COMP-3 sign.
 
Use three runs: D = PARM '20261030,N', M = PARM '20261031,Y',
L = PARM '20280229,N'.
Number the scenarios S01, S02 and so on, in table order.
 
Return a table with these columns:
ID | Input (status, type, balance, accrued, run) | Expected: original |
Expected: modified | Should change? | Why it matters
 
For every expected value, show the calculation, and remember that
INTCALC truncates daily interest to 4 decimal places. Do not round.
```
 
**Expected result.** This is the answer key. Expected values are the daily interest added to `ACCT-ACCRUED-INT`, unless the row says otherwise. All accounts are type `SV` with status `A` unless noted.
 
**Figure 05,** boundary scenarios for the rate change
 
| ID  | Input                                             | Expected: original | Expected: modified | Should change? | Why it matters |
|-----|---------------------------------------------------|--------------------|--------------------|----------------|----------------|
| S01 | Balance 0.00, run D                               | No accrual         | No accrual         | No  | Zero balance is excluded by the `EVALUATE` |
| S02 | Balance -0.01, run D                              | No accrual         | No accrual         | No  | Overdrawn accounts earn nothing; tests the `D` sign half-byte |
| S03 | Balance 1,000.00, run D                           | 0.0342             | 0.0342             | No  | Exactly on the tier 2 floor, so 1.25% applies |
| S04 | Balance 999.99, run D                             | 0.0136             | 0.0136             | No  | One cent below tier 2, so 0.50% applies |
| S05 | Balance 10,000.00, run D                          | 0.6219             | 0.6232             | **Yes** | Exactly on the tier 3 floor, where the rate gains its third decimal |
| S06 | Balance 9,999.99, run D                           | 0.3424             | 0.3424             | No  | One cent below tier 3 must stay on tier 2 |
| S07 | Balance 50,000.00, run D                          | 4.2739             | 4.2808             | **Yes** | Exactly on the tier 4 floor |
| S08 | Type `BS`, bonus flag `Y`, 0 withdrawals, balance 20,000.00, run D | 1.5178 | 1.5205    | **Yes** | Tier 3 plus the 0.500 bonus, both at 3 decimals |
| S09 | Status `D` (dormant), balance 20,000.00, run D    | 0.2739             | 0.2739             | No  | Dormant accounts stay on tier 1, whatever the balance |
| S10 | The S03 record, run L                             | 0.0341             | 0.0341             | No  | Leap year divides by 366; compare with S03's 0.0342 |
| S11 | Balance 100.00, accrued 0.0037, run M             | Balance 100.01, accrued 0 | Same        | No  | Accrued reaches 0.0050, a half cent, which rounds up |
| S12 | Balance 100.00, accrued 0.0036, run M             | Balance 100.00, accrued 0 | Same        | No  | Accrued reaches 0.0049, which rounds down |
| S13 | Balance 9,999,999.99, run M                       | Overflow exception; balance unchanged, accrued reset to 0 | Same | No | Maximum `S9(7)V99`. The interest is lost in both versions: Module 1, rule 12 |
| S14 | Balance 1,234.56, run D                           | 0.0422             | 0.0422             | No  | Positive packed value, tests the `C` sign half-byte |
| S15 | Balance 49,999.99, run D                          | 3.1095             | 3.1164             | **Yes** | One cent below tier 4 stays on tier 3, whose rate also changes |
| S16 | Balance 10,000.00, accrued 0.0030, run M          | Balance 10,000.62, accrued 0 | Balance 10,000.63, accrued 0 | **Yes** | The difference survives month-end rounding: 10,000.6249 rounds down, 10,000.6262 rounds up |
 
**Check it against the source.** Pick two rows and work them out yourself before you trust the table. S05 is a good one:
 
```text
Original:  10,000.00 × 2.27  ÷ 100 ÷ 365 = 0.621917…  → truncated to 0.6219
Modified:  10,000.00 × 2.275 ÷ 100 ÷ 365 = 0.623287…  → truncated to 0.6232
```
 
Then look for these, which come up in almost every room:
 
- **Rounding instead of truncating.** If your model says 0.6233 for S05 modified, it rounded. The code doesn't.
- **S13 marked "Should change? Yes".** In a daily run it would change, because tier 4 earns more. In the month-end run it overflows, and the accrued interest is reset to zero in both versions, so the output is identical. Identical output here is **not** the same as correct output. Parity proves the change didn't break anything new; it says nothing about the defect that was already there.
- **S10 marked "Should change? Yes".** The leap-year divisor differs between runs, not between versions. That row exists to prove the day count, not the rate change.
- **S12 forgotten.** One half-cent case proves very little. The case just below it is what shows the rounding boundary is in the right place.
- **A case quietly dropped.** Models often leave items out of a long list. Count your rows against the **Cover** list in the prompt before you move on.
Your IDs may not match Figure 05's, and that's fine. Compare scenarios by their inputs, not their numbers.
 
Paste your checked table into `scenarios.md`.
 
### Step 3, generate the boundary records (6 minutes)
 
Now, and only now, generate the data.
 
```text
#file:ACCTREC.cpy #file:scenarios.md
 
1. From ACCTREC, list every field with its PIC, format, offset and length,
   and confirm the total is 100 bytes.
2. Write one CSV row for every scenario in scenarios.md, in table order.
   ACCT-NUMBER is BND plus the scenario number, padded to 9 digits:
   BND000000001 for S01, BND000000002 for S02, and so on.
   If a scenario only reruns another scenario's record with a different
   PARM, write no row for it and skip its account number.
   Use the ACCTREC field names as the header, and readable decimal values.
   Do not include FILLER. Use ACCT-LAST-INT-DATE 20260930,
   ACCT-BONUS-FLAG N and ACCT-WITHDRAWALS-MTD 0 unless the scenario says
   otherwise.
3. List any scenario you wrote no row for, and why. Then confirm that
   every other scenario has exactly one row.
4. Show the COMP-3 hex for ACCT-BALANCE for the negative-balance, the
   maximum-balance and one positive scenario.
```
 
**Expected result**
 
The layout should match Figure 03, with a total of 100 bytes. The CSV should have **15** rows:
 
```csv
ACCT-NUMBER,ACCT-TYPE,ACCT-STATUS,ACCT-BALANCE,ACCT-ACCRUED-INT,ACCT-LAST-INT-DATE,ACCT-BONUS-FLAG,ACCT-WITHDRAWALS-MTD
BND000000001,SV,A,0.00,0.0000,20260930,N,0
BND000000002,SV,A,-0.01,0.0000,20260930,N,0
BND000000003,SV,A,1000.00,0.0000,20260930,N,0
BND000000004,SV,A,999.99,0.0000,20260930,N,0
BND000000005,SV,A,10000.00,0.0000,20260930,N,0
BND000000006,SV,A,9999.99,0.0000,20260930,N,0
BND000000007,SV,A,50000.00,0.0000,20260930,N,0
BND000000008,BS,A,20000.00,0.0000,20260930,Y,0
BND000000009,SV,D,20000.00,0.0000,20260930,N,0
BND000000011,SV,A,100.00,0.0037,20260930,N,0
BND000000012,SV,A,100.00,0.0036,20260930,N,0
BND000000013,SV,A,9999999.99,0.0000,20260930,N,0
BND000000014,SV,A,1234.56,0.0000,20260930,N,0
BND000000015,SV,A,49999.99,0.0000,20260930,N,0
BND000000016,SV,A,10000.00,0.0030,20260930,N,0
```
 
And the packed values:
 
| Scenario | Balance       | `COMP-3` hex      | Last half-byte |
|----------|---------------|-------------------|----------------|
| S02      | -0.01         | `X'000000001D'`   | `D`, negative  |
| S13      | 9,999,999.99  | `X'999999999C'`   | `C`, positive  |
| S14      | 1,234.56      | `X'000123456C'`   | `C`, positive  |
 
**Check it against the source.** Confirm the offsets total 100, and that `ACCT-BALANCE` and `ACCT-ACCRUED-INT` are five bytes each. Read the last half-byte of every hex value. If the model used `F` for positive values, or gave a 10-byte field, correct it now: those are exactly the errors that make a test file fail to load, or worse, load with every field shifted.
 
**Count the rows.** The model should have listed which scenarios got no row (here, only S10). If any other scenario is missing from the CSV, it was dropped: add it before you save.
 
Note the gap at `BND000000010`. There's no S10 record, because S10 is the S03 record in a different run. Leaving the gap keeps account numbers and scenario IDs in step, which makes the Lab report easier to read.
 
Save the CSV as `boundary-records.csv`.
 
### Step 4, review coverage (2 minutes)
 
Last, use your Module 1 spec to find what the table doesn't cover.
 
```text
#file:scenarios.md #file:INTCALC-spec.md
 
Compare the scenarios with the business rules in the spec. List every
rule that no scenario exercises. For each, say what could go wrong if
this change affected it, and whether it is worth adding a scenario.
```
 
**Expected result.** The model should find gaps along these lines:
 
- **Closed accounts**: no scenario proves a closed account is still dropped from the master file.
- **Legal hold**, status `L`: no scenario proves it still behaves as frozen.
- **Bonus lost after a withdrawal**: no `BS` account with withdrawals above zero.
- **Accrued-interest overflow**: only the balance overflow is tested.
- **A fifth rate record**: if someone adds a tier to the new rate file, it is silently ignored.
- **An unexpected status value**: any status that isn't `C`, `F`, `L` or `D` falls through to `WHEN OTHER` and earns interest.
You don't need to add them all. Decide as a room which two matter most for *this* change and add them to `scenarios.md` and `boundary-records.csv`. A good argument: the bonus-after-withdrawal case, because it uses the bonus rate the developer just changed, and the closed-account case, because a dropped record is the hardest kind of difference to spot in a report.
 
> **Discuss:**
>
> S13 produces identical output from both versions and still loses a customer's interest. What should a tester do with a finding like that, when it isn't caused by the change under test?
 
## Check for Understanding
 
Before moving into the Lab, work through these, out loud or on paper.
 
- The harness reports zero differences between the original and modified `INTCALC`. Using your expected-change rule, explain why that would be a **failure**, not a pass.
- A colleague's AI-generated test file has a balance field that is 10 bytes long. What else in the record is now wrong, and how would you have caught it before running anything?
- Why does the scenario table include a value one cent below each tier floor, and not just the floor itself?
- Copilot gives you 0.6233 as the modified expected value for S05. What did it do wrong, and what is the correct value?
- Name two things outside `INTCALC` that must be released together with this change, and what happens if one of them isn't.

## Summary
 
- Decide what should change before you test anything. The **Should change?** column is what turns "no differences" into a result you can trust.
- Ask AI for scenarios in a table a reviewer can read, with an expected result and a reason for every row. Data comes second.
- Boundaries come from the code: branch conditions, tier floors, `PIC` limits, truncation and rounding points, date logic and sign. Test on each boundary and just beside it.
- Records come from the copybook. Check that offsets add up to the record length, that `COMP-3` lengths are right, and that sign half-bytes are `C` or `D`.
- Models are fluent at COBOL and unreliable at arithmetic. Ask for the working, and check rows yourself, especially where the code truncates.
- Parity is not correctness. Identical output proves a change didn't introduce a difference; it doesn't prove the original was right.
In the Lab, you add `boundary-records.csv` to the 5,000-account synthetic file, run the harness on the original and modified `INTCALC`, and read its report: unchanged, changed as expected, and unexplained. Then you use AI to explain the differences no scenario predicted.
 
## Vocabulary
 
- **Regression test**, a test that confirms a change hasn't altered behaviour it wasn't meant to alter, usually by comparing output before and after the change.
- **Scenario table**, a readable list of test cases with their inputs, expected results and the reason each one exists. Agreed before any test data is built.
- **Expected-change rule**, a precise statement of which records should produce different output after a change. Everything outside it should be identical.
- **Boundary value**, a value at the edge of a decision or limit in the code, such as a tier floor, a field's maximum size or a rounding point. Defects cluster at boundaries.
- **Equivalence partition**, a range of inputs the program treats the same way. One value from inside each partition, plus the boundaries between them, gives good coverage with few cases.
- **Synthetic test data**, records generated from the copybook layout to cover chosen scenarios, rather than copied from production.
- **`COMP-3` (packed decimal)**, a numeric format storing two digits per byte, with the sign in the last half-byte: `C` positive, `D` negative.
- **Sign half-byte (nibble)**, the last four bits of a packed field, which hold its sign.
- **Offset**, the starting position of a field within a fixed-width record. Offsets must add up to the record length.
- **Fixed-width record**, a record where every field sits at a fixed position and length, as the copybook defines. A one-byte error shifts every field after it.
- **Parity**, identical output from two versions of a program for the same input. Proves nothing changed; doesn't prove anything is correct.
- **Regression harness**, the course's pre-built tooling that runs the original and modified program on the same file and reports which records are unchanged, changed as expected or unexplained.
- **Truncation**, dropping digits that don't fit the receiving field. `INTCALC` truncates daily interest to four decimal places.
 
